# SkipList



# Memory Barrier

- x86 内存模型
- std::memory_order 和std::atomic
- asm volatile("" : : : "memory");
- Java 内存模型



## MemoryBarrier

指令的乱序包含两方面，一方面是编译器上的乱序，另一方面是CPU上的乱序。

在编译器上的乱序使用MemoryBarrier指令可以确保在代码中该指令位置之前的代码对应的指令必定在之后的代码对应的指令之前。在GCC x86上可以使用`__asm__ __volatile__("" : : : "memory");`, 而在C++11中可以使用语句`atomic_signal_fence(memory_order_acq_rel);`

在CPU上的乱序中，不同计算机体系结构其内存模型以及可能出现的乱序都有所不同。x86-64是强内存模型，只存在SL乱序，要确保SL顺序，则第一需要确保将L之后的指令在L后面（Acquire)，第二，确保S之前的指令在S前面(Release)

偏序关系语义的确定是避免影响其他线程对变量的理解

## atomic





# Memory Map



# Memtable Write





# Log Ahead



# LSM Tree

- 怎么保证各个level上的key范围不会重叠？

- 只在同一个LEVEL下的文件相互不会重叠。

- LEVEL0 - 1, 选定一个LEVEL1文件，合并LEVEL0中和此文件key范围有交集的文件。

- LEVELN - N+1，轮流选定一个文件A，选择L+1层中和文件A在key range上有重叠的所有文件来和文件A进行合并。

  ​

# Cache

