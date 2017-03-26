# 引用

Java中一共有四种类型的引用。StrongReference、 SoftReference、 WeakReference 以及 PhantomReference。

- StrongReference 是 Java 的默认引用实现, 它会尽可能长时间的存活于 JVM 内，当没有任何对象指向它时将会被GC回收
- WeakReference，顾名思义, 是一个弱引用, 当所引用的对象在 JVM 内不再有强引用时, 将被GC回收
- 虽然 WeakReference 与 SoftReference 都有利于提高 GC 和 内存的效率，但是 WeakReference ，一旦失去最后一个强引用，就会被 GC 回收，而 SoftReference 会尽可能长的保留引用直到 JVM 内存不足时才会被回收(虚拟机保证), 这一特性使得 SoftReference 非常适合缓存应用



# 线程

生命周期

1. NEW  新建
2. RUNNABLE 就绪，可调度
3. RUNNING 运行
4. DEAD 结束
5. BLOCKED 被阻塞

```
NEW -> RUNNABLE -> RUNNING -> DEAD
                \            \
                 \-> BLOBKED  \
```

thread.setDaemon(true)必须在thread.start()之前设置，否则会跑出一个IllegalThreadStateException异常

在Daemon线程中产生的新线程也是Daemon

#  ThreadPool

## 作用

- 资源：复用线程资源，通过减少新建线程的次数减少系统开销。
- 效率：并发处理多个工作任务，也可将任务切分为若干个子任务并行执行，提高工作效率。
- 调度：可以对工作任务队列进行调度和控制。
- 管理：可以对线程资源进行管理、分配和控制。

## 流程

1. 定义一个可执行任务，通常是一个方法（`Runnable`）。
2. 将任务提交给`Executor`。
3. `Executor`根据工作情况决定执行的时机，这由`Executor`本身的实现确定，最简单的可以立即执行任务。通常的是将任务加入到队列中，如同排队一样，线程不断地从队列拿出任务执行，当任务处于队首并有线程空闲时便满足执行条件。
4. 当满足执行的条件时，`Executor`从线程池中拿取或复用一个空闲的线程运行该任务。
5. 线程运行任务。与此同时，`Executor`可以继续安排或调整其他提交任务的执行时机。
6. 任务结束后，线程资源回收，如果所有任务都已经处理完毕，线程则处于空闲状态。

## 影响效用的因素

1. 线程池的数量，包括

   - 最小线程数（`corePoolSize`）。一般来说不会显著影响性能，适当的设置能够避免创建过多的线程。

   - 最大线程数（`maximumPoolSize`）。若设置过小，则会失去多核处理器的并行优势。若设置过大，则频繁的线程切换开销或者线程间通信的增加反而导致效率降低。**线程数并非越大越好，一定要与可利用资源相匹配，并将任务执行的性能瓶颈点加以考虑**。

   - 扩充线程的策略。一般情况下，当新任务到来时，是：

     1. 若**没有空闲线程**且未满足最小线程数，则新建线程，也可一开始就准备好最小线程数的线程。

     2. 若**任务队列已满**且未满足最大线程数，则新建线程。

        <u>？ 顺序 ？ 空闲线程 - 新建满足最小线程数 - 任务队列 - 最大线程数 - 饱和策略 ？</u> 

2. 线程的空闲时间（`keepAliveTime`）。线程池的工作线程空闲后，保持存活的时间。根据任务提交的频率和执行时间调整空闲时间能够提高线程的复用率。

3. 任务队列（``runnableTaskQueue``），包括

   - 队列大小（`runnableTaskQueue.size`）。过大的队列会使任务响应时间过长。
   - 队列特性。依据队列的实现决定。可以按照FIFO方式处理任务（`ArrayBlockingQueue`\ `LinkedBlockQueue`），也可根据优先级（`PriorityBlockingQueue`）确定任务执行顺序，等等。

4. 饱和策略（`RejectedExecutionHandler`）。当队列和线程池都满时，线程池处于饱和状态下对提交的新任务的处理策略。如`AbortPolicy`、`DiscardOldestPolicy`、`DiscardPolicy`等等。




## ConcurrentHashMap

size(), 使用的算法和LongAdder相同

## LongAdder

- Add：采用逐步策略的手段更新值，先对base进行CAS add此时性能和**AtomicLong**相近，失败时向Cell分段更新，根据Thread的hashcode映射到Cell数组中的某个元素，向该元素更新值。
- Cell：对Cell进行Padding（增加无用字段），减少cache contention，避免多个core对同一个cache块的争用。
- Sum：会统计base和Cell的值。






# 动态代理

## InvocationHandler

实现代理具体逻辑的接口，在调用被代理的接口方法时会传递到invok方法，其中实现代理逻辑，代码签名

```java
public interface InvocationHandler {
  // proxy: 被实例化的代理类
  // method：传递的被代理的方法
  // args：method执行的实参
  // return： method返回类型的值
  public Object invoke(Object proxy, Method method, Object[] args) throw Exception;
}
```

限制：

1. 被代理类必须实现接口
2. 接口一定需对Classloader可见
3. 接口为public接口或与被代理对象的类与接口在相同命名空间内
4. 抛出的异常和接口签名兼容，或RuntimeException

## Proxy 

基本使用。

```java
SuppressWarnings("unchecked")
public static <T> T proxying(T target, Class<T> iface) {
    return (T) Proxy.newProxyInstance(        // 0 - Proxy.newProxyInstance
        iface.getClassLoader(),               // 1 - classloader
        new Class<?>[] { iface },             // 2 - interface
        new MyInvocationHandler(target));     // 3 - handler(beProxyedObject)
}
```

[Ref] https://opencredo.com/dynamic-proxies-java-part-2/



# Memory Area

## 区域

- 不受GC管理的内存都是native memory；受GC管理的内存叫做GC heap或者managed heap



## 对象内存布局

HotSpot VM:

- 继承深度越浅的类所声明的字段越靠前，继承深度越深的类所声明的字段越靠后
- 每个字段按照其宽度来对齐
- 字段不参与多态。派生类如果声明了跟基类同名的字段，则两个字段在最终的实例中都会存在
- 可以使用jol工具查看内存布局

```
+-------------------+-------------------------+
|                   |                         | 
|     _mark         |      _kclass            | 
|                   |                         | 
+-------------------+-------------------------+
|  +-----+-----+------+-----------+---------+ |
|  | l/d | i/f | s/ch | byte/bool |  ref    | |
|  +-----+-----+------+-----------+---------+ |
|  |      ... (extend_field)                | |
|  +----------------------------------------+ |
+---------------------------------------------+
```

[ Ref ]

- https://www.zhihu.com/question/50258991/answer/120450561



## 类数据内存布局

- 对象、类的元数据（InstanceKlass）、类的Java镜像，三者之间的关系：

```
Java object      InstanceKlass       Java mirror
 [ _mark  ]                          (java.lang.Class instance)
 [ _klass ] --> [ ...          ] <-\              
 [ fields ]     [ _java_mirror ] --+> [ _mark  ]
                [ ...          ]   |  [ _klass ]
                                   |  [ fields ]
                                    \ [ klass  ]
```

- 在JDK 7或之前的HotSpot VM里，InstanceKlass是被包装在由GC管理的klassOopDesc对象中，存放在GC堆中的所谓Permanent Generation（简称PermGen）中
- 从JDK 8开始的HotSpot VM则完全移除了PermGen，改为在native memory里存放这些元数据。新的用于存放元数据的内存空间叫做Metaspace，InstanceKlass对象就存在这里。

[ Ref ]

- https://www.zhihu.com/question/50258991/answer/120450561



## 静态变量

- 静态变量存储在方法区中的数据区里。
- OpenJDK6-, HotSpot VM，静态字段依附在InstanceKlass对象的末尾
- OpenJDK7+, HotSpot VM, 静态变量存储在java.lang.Class对象末尾的隐藏字段里，而java.lang.Class对象存储在普通的Java heap里

[ Ref ]

- https://www.zhihu.com/question/50258991/answer/120450561


## 常量池

1.Class文件中的常量池

这里面主要存放两大类常量：字面量(Literal)：文本字符串等；符号引用(Symbolic References)：属于编译原理方面的概念，包含三类常量：类和接口的全限定名(Full Qualified Name)字段的名称和描述符(Descriptor)方法的名称和描述符




# Concurrent

## CountDownLatch

- 某个线程需要等待一个或多个线程操作结束（或达到某种状态）才开始执行



## CyclicBarrier

- CyclicBarrier是让多个线程互相等待某一事件的发生，然后同时被唤醒




# GC

Stop-the-world 会显著影响应用性能，JVM调优主要围绕这减少停顿时间为目的。

一般垃圾回收算法会对堆会分代。新生代和老年代。为什么会分代？新生代用于分配快用快销的对象的内存，减少扫描区域从而减少暂停时间。

当迁移到老年区的对象大小大于老年区可用内存时，将有可能引发Full GC，最为影响程序性能的操作。

并发GC在减少响应时间的同时，可能会增加CPU运算开销。



-XX:+UseSerialGC 使用单线程并行GC ，部分GC和全GC都是全暂停。

-XX:+UseParallelOldGC 使用并行GC，部分GC和全GC都是全暂停，使用多个线程进行回收。

-XX:+UseParNewGC 使用CMS GC，部分GC全暂停，全GC尽量减少暂停时间，并发标记，代价是CPU开销以及内存碎片，需要时切换为SerialGC整理内存。

-XX:+UseG1GC 使用G1算法，针对大堆内存应用，实现并发显著减少老年区回收时的暂停时间。

尽量不使用System.gc()，会引发Full GC，可以通过-XX:+DisableExplicitGC设置禁止调用

需要Full GC的场景：

- GC 算法本身触发条件
- 做内存测试
- 做Heap Dump
- RMI 每小时调用http://blog.csdn.net/chenleixing/article/details/46706039






- ConcurrentHashMap size() 实现？一致性？

- 为什么编译器、CPU会对指令进行乱序？

- synchronize 锁怎样实现？ 偏向锁？

- ThreadLocal 会不会内存泄露，内部map实现的WeakReference？

- 二级索引命中会有几次索引查询 ？？

- Mysql MVCC怎么解决幻读问题？2PL？版本存储策略？间隙锁？ 和聚簇索引

  的关系？

- 数据库和缓存的一致性如何保证？

- 结果100万数据做分页查询，查询第90万条怎样优化？

  ​




>>>>>>> note-mysql
