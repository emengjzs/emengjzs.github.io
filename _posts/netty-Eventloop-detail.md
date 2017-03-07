



## 0 概要

- 了解Netty中Eventloop的运行机制，包括epoll、网络IO任务、文件IO任务、定时任务、非阻塞任务调度，只关注tcp和epoll（Nio）。
- Netty版本为4.1.6.Final



## 1 初始化



### 1-1 NioEventLoop类继承关系图

TODO

## 2 启动Eventloop

reactor本质上是完成调度和完成任务的Executor，继承ExecutorService接口。

任务执行逻辑：

- 判断提交任务的线程是否为本线程，若是直接添加任务。
- 否则，启动线程，state 是线程的状态（启动、停止等），改变状态为ST_STARTED，将reactor总调度过程（即Eventloop）作为一个任务提交给另外一个executor，executor的作用是为reactor分配一个独立线程，线程的运行即Eventloop的运行，executor的实现类是ThreadPerTaskExecutor，即一个Eventloop分配一个线程，非常合理。

```java
// SingleThreadEventExecutor.java - line 751
public void execute(Runnable task) {
  if (task == null) {
    throw new NullPointerException("task");
  }

  boolean inEventLoop = inEventLoop();
  if (inEventLoop) {
    addTask(task);
  } else {
    startThread();
    addTask(task);
    if (isShutdown() && removeTask(task)) {
      reject();
    }
  }

  if (!addTaskWakesUp && wakesUpForTask(task)) {
    wakeup(inEventLoop);
  }
}
```

###  2-1 ThreadPerTaskExecutor 

TODO



###  3 执行Eventloop

- 任务队列中有任务，进行一次非阻塞select （selectNowSupplier）；否则，设定为SelectStrategy.SELECT，进行可能阻塞的select任务。
- ​