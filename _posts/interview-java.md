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
+-------------------+-------------------------|
|  +----------------------------------------+ |
|  | l/d | i/f | s/ch | byte/bool |  ref    | |
|  +----------------------------------------+ |
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