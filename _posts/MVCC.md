# MVCC

目的：解决ACI。

## 并发控制协议

### 基于时间戳的协议 （Timestamp Ordering）

https://en.wikipedia.org/wiki/Timestamp-based_concurrency_control

https://www.tutorialspoint.com/dbms/dbms_concurrency_control.htm

#### Thomas write rule

https://en.wikipedia.org/wiki/Thomas_write_rule



### 基于乐观锁的协议 （Optimistic Concurrency Control）





### 二阶段协议 （Two-Phase Locking）



## 版本存储实现

### 追加型

### 穿越型

### Diff型 



## 副本回收算法

### 元组级

###  事务级





## 并发控制与二级索引





## MVCC 实现

### MySQL with Innodb

快照读：

当前读：对读取的行数据进行加锁，以下的操作会使用当前读：

1. select lock in share mode;
2. select for update
3. insert into
4. update table set
5. delete from table

1使用共享锁，2-5使用互斥锁

对于update或者delete的数据集，其加锁过程是逐条加锁的，先读取，后加锁返回，再对数据行进行更新。

## SQLLite

https://sqlite.org/src4/doc/trunk/www/lsmusr.wiki#database_transactions_and_mvcc

## TiDB （Optional） 

https://pingcap.github.io/blog/2016/11/17/mvcc-in-tikv/





索引

聚簇索引

索引键选取：主键 -> unique非NULL键 -> 隐藏主键

指叶节点包含了行的所有数据，节点数据组成页，innodb 默认使用主键做聚簇索引，其他键的索引叫二级索引。

缺点：

更新索引列的代价高

插入、移动数据行可能导致页分裂

二级索引叶节点包含的是主键值，故索引命中时需要两次索引查询。