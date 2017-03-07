---
layout: post
title: leveldb - Put 操作解读
---

## Objectives

- 了解leveldb并发写的处理手段，以及提高写吞吐量的Tricks。

## DB::Put

这是最高层的API入口，DB Interface 提供了Put和Write方法，前者针对单个KV数据，后者支持批量数据写，接口提供了Put的默认实现，直接复用Write方法，将数据封装成批量对象（其实只有一条）调用Write方法。

了解过Memtable、InternalKey和底层的skiplist就可知道删除（Delete）、更新（Put）也复用了Write方法。

```c++
// db_impl.cc - line 1413
// Default implementations of convenience methods that subclasses of DB
// can call if they wish
Status DB::Put(const WriteOptions& opt, const Slice& key, const Slice& value) {
  WriteBatch batch;
  batch.Put(key, value);
  return Write(opt, &batch);  // Invoke Write implemention method.
}
```

## DBImpt::Write - 1

首先封装成一个任务Writer，包含数据、设置选项（只有sync一项，见options.h - line 169），互斥锁，以及完成标志。

原理上，同一时间应该是只有一个Writer执行写操作的（skipList底层实现的Insert不是lock-free算法，同时操作会丢失节点），但却能够有其他的方法优化写吞吐。思路是，在一个线程获得锁进行写操作时，可以顺带地把其他等待的写的线程的写任务执行完，这样其他线程发现写操作已完成（Writer.done）时，立即释放锁返回，从而提高并发性能。

Leveldb 的Java port版本（https://github.com/dain/leveldb）就不是一个好的实现，写操作是全程锁的 (https://github.com/dain/leveldb/blob/master/leveldb/src/main/java/org/iq80/leveldb/impl/DbImpl.java#L682)，而且底层使用的是支持并发读写的ConcurrentSkipList，应该有更好的实现。（如何实现需要进一步思考）。

DB维护了一个写任务队列writers，在队列上设置互斥锁，获得锁后加入到writer队列，先忽略Writer.done，当写任务的当前线程在队首时被signal，否则一直wait，注意同步互斥的四要素（检查变量、mutex、cv.wait()、和while）。

```c++
// dp_impl.cc - line 1163 
MutexLock l(&mutex_);                // lock with mutex
writers_.push_back(&w);              // add write task to the queue 
while (!w.done && &w != writers_.front()) {   
  w.cv.Wait();  // wait until the front write tasks have been done.
}
```

待线程被唤起时，写任务必在队首，限制只有队首的线程才可进行写操作。1173 行之后开始核心的写流程。MakeRoomForWrite用于检查是否到做Compaction、Dump Memtable的时机，本篇忽略。

## DBImpt::BuildBatchGroup

注意到一句 BuildBatchGroup(&last_writer); 该函数执行后last_writer的值会变化，它做的是适当合并队列中等待中的剩余的写请求，设置一个大小上限，将这些排在当前线程之后的写任务打包到tmp_batch_，并将last_writer设置为最后一个被合并的写任务。这样这些正等待写的线程就不需再执行写任务了，在唤醒前一定会被之前的这个线程帮忙做了。

另外之前的合并操作中，注意不能把sync的写合并到非sync的写任务，否则不符合意图。

```c++
// db_impl.cc - line 1243
if (w->sync && !first->sync) {
  // Do not include a sync write into a batch handled by a non-sync write.
  break;
}
```

## DBImpt::Write - 2

然后写操作使用的是log-ahead策略，即现确保写到日志上，再写到数据库中，这样确保写入了日志的数据必定能够保存或恢复，当写数据时发生故障时。日志使用了mmap，进程挂了也不受影响，但断电会受影响，Sync设置其实是指日志是否立即flush到磁盘，设置为true时写性能会降低，一般策略是周期性count的flush，也即如果sync为false时（默认值），其实是有可能丢失数据的，仅当机器断电时。日志是append操作保证速度。

而在log和insert到memtable这期间，writer可以释放所持互斥锁，因为只有队首的线程才可写，故可以保证即使释放锁也没有问题，释放锁可以让其他更多的写线程加入到队列中，结合合并写的策略增大写吞吐量。

```c++
// Add to log and apply to memtable.  We can release the lock
// during this phase since &w is currently responsible for logging
// and protects against concurrent loggers and concurrent writes
// into mem_.
{
  mutex_.Unlock();
  status = log_->AddRecord(WriteBatchInternal::Contents(updates));
  if (status.ok() && options.sync) {
    status = logfile_->Sync();
  }
  if (status.ok()) {
    status = WriteBatchInternal::InsertInto(updates, mem_);
  }
  mutex_.Lock();
}
```

最后，清空打包的tmp_batch_和更新Seq号，这期间当然需要重新获取互斥锁。之后再通知已经帮忙处理过写任务的任务完成标志，（设置为done = true和signal线程），并清出写队列，最后，应该还有未被合并处理的写任务线程，通过检查队列是否为空进行通知，signal队首线程。

方法结束时，MutexLock被析构，自动释放互斥锁（RAII）。

```c++
while (true) {
  Writer* ready = writers_.front();
  writers_.pop_front();
  if (ready != &w) {
    ready->status = status;
    ready->done = true;    // Tell that the task has been done by current thread.
    ready->cv.Signal();    
  }
  if (ready == last_writer) break;
}

// Notify new head of write queue
if (!writers_.empty()) {
  writers_.front()->cv.Signal();
}
```

## Tricks

- 细粒度的锁，而不是整个write过程上锁，这样数据库就没有意义了。
- 线程在执行任务时可以帮着完成其他线程的任务，增大吞吐量。
- 向文件Append-Write而不是随机写。
- 内存核心的数据结构skiplist。

