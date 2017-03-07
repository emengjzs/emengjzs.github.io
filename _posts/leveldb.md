# SkipList



# Memory Barrier

- x86 内存模型
- std::memory_order 和std::atomic
- asm volatile("" : : : "memory");
- Java 内存模型

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

