# Node vs Netty



## Buffer

- 实现了[`Uint8Array`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array) API接口

- 固定大小的堆外内存，通过被Buffer对象的引用关联（Buffer对象本身在v8控制内）实现内存自动回收。






## Module

- 模块类型

  - 核心模块
  - 相对路径文件模块
  - 绝对文件模块
  - Node模块，按搜索路径

- 模块作用域隔离，exports、require、module通过形参传入

  ```javascript
  (function (exports, require, module, __filename, __dirname) {
  // Your module code actually lives in here
  });
  ```

  因此不能用export  = ...

  

## IO

### 文件IO

- 使用线程池实现（模拟）异步模型，读写时读写线程阻塞



## 网络IO

- 使用epoll





## Eventloop

- 一系列trick，一个trick为一个循环

- IO阻塞的异步方法被封装成Handler，包括调用方法引用、方法参数、处理状态，以及完成时回调的方法引用。交由线程池处理任务（或？），处理完毕后，将处理状态更新为完成，并将此作为事件提交到队列。Eventloop的线程需要在一个loop中处理事件队列（是Eventloop线程在一个loop中需要处理的事情之一），从而在处理事件时即等于调用回调函数。

- 定时任务，handler放置在一个优先队列，由Evenloop线程进行处理（是Eventloop线程在一个loop中需要处理的事情之二），当接近或到达时限时，取出定时时间对应的handler，调用定时的注册函数。

- 处理任务顺序 （https://github.com/nodejs/node/blob/feedca787940fe4bf036b6df767335eda866630f/doc/topics/the-event-loop-timers-and-nexttick.md Node4.7）

  - timers 定时任务 ， 通过setTimeout 和 setImmediate添加的任务，任务保存在按时间优先的队列中。
  - I/O回调任务，当IO完成时，回调任务塞入此队列，此阶段处理这些加入到队列的任务。
  - idle， prepare任务 
  - IO任务 （epoll )  (阻塞？非阻塞 ？ 时间？)，当有就绪的读写事件时执行这些事件。
    - 执行一次selectNow，若有就绪事件，执行。
    - 若没有，则如果有check任务，则结束IO任务阶段。
    - 若没有check任务，则适当阻塞select一段时间，时间不应超过定时任务中的最早的deadline
  - check任务，通过setImmediate注册的任务 ，FIFO队列
  - close任务，执行socket的销毁操作。
  - ?? 通过nexttick添加的任务，直到任务清空（包括递归添加的）为止，**没有任务大小限制**，执行后结束本次tick

- 以下输出顺序不确定(Node.js v6.9.1)

  ```javascript
  // timeout_vs_immediate.js
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
  ```

  按照时间处理逻辑，应该是immediate先于timeOut执行

  Github issue 

  [non-deterministic order of execution of setTimeout vs setImmediate #392](https://github.com/nodejs/help/issues/392)

  [observed setImmediate and setTimeout execution order contradicts documentation #7145](https://github.com/nodejs/node/issues/7145)



## Event

事件四要素

- 事件（event），一个标识符表示这个事件，通常是名字
- 注册（on），将监听函数关联到事件上，表示当事件发生时，函数被调用。
- 分发（emit），当满足一定条件，事件需要发起时（这是用户自定义的），则调动注册的函数。
- 上下文，调用函数时，将事件发生时的一些信息传入函数中。- 



listener调度

仅仅是简单的同步遍历：

```javascript
for (var i = 0; i < len; ++i)
      listeners[i].apply(self, args);
```

