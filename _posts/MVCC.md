# CC

目的：解决ACI。

<<<<<<< HEAD
为了提高数据库处理能力，在允许的情况下尽可能让多个事务并发执行，只要这些事务的执行结果和在严格串行下执行结果相同即可。我们希望并发的事务调度和串行的事务调度等价，即**冲突可串行化**。

冲突可串行化要求：

1. 各自事务中的指令在调度中的顺序**保持一致**，不得调换一个事务中的两个指令的顺序。
2. 不同事务对同一数据的操作仅当**都是读操作**时才可调换指令顺序，其余情况下不能调换顺序。
3. 不同事务对**不同数据**的操作可以调换指令顺序。

如果根据先后要求得出调度的事务集合满足偏序性质，则为冲突可串行化，此时可以通过拓扑排序转换为一个或多个等价的串行调度。
=======


## 事务

- ACID
- 四个隔离级别
- 可恢复的的调度计划：任何两个的事务A和事务B，如果A依赖于B，则A的提交点必须不早于B的提交点，否则当B回滚时，A不能恢复。
- 连锁回滚：在回滚事务A时，任何依赖于事务A的所有事务都必须回滚。
- 无连锁的调度计划：任意两个事务，事务A对事务B的依赖发生在事务B提交之后。
>>>>>>> 428df9709361edf25026671bbf97c45ea5995ae0



## 并发控制协议

### 基于锁的协议

使用S锁和X锁实现事务隔离。不允许冲突的锁同时出现在同一个数据上。避免饥饿的方法是一个事务不能被之后的事务阻塞对数据锁的获取。



### 基于时间戳的协议 （Timestamp Ordering）

https://en.wikipedia.org/wiki/Timestamp-based_concurrency_control

https://www.tutorialspoint.com/dbms/dbms_concurrency_control.htm

#### Thomas write rule

https://en.wikipedia.org/wiki/Thomas_write_rule



### 基于乐观锁的协议 （Optimistic Concurrency Control）





### 二阶段协议 （Two-Phase Locking）

1. 一旦事务获取了锁，事务即进入上升期，若需要维持在上升期，可以继续获取锁但不能够释放锁。
2. 如果在上升期释放了锁，则进入下降期。
3. 在下降期期间，可以继续释放锁但不能够获取锁。



严格的二阶段协议还要求线程必须在提交事务之时才可以释放X锁。为了？

严密的二阶段协议则要求在提交事务之时才可以释放所有持有的锁。

在遵守二阶段协议的同时为了减少互斥锁的持有时间，读锁可以在需要时提升为写锁，锁的提升只能在上升期内完成。



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

每个数据行会有三个隐藏字段：DB_TRX_ID 记录最近的插入或更新（删除将视作更新，通过标志位区别）的事务ID，`DB_ROLL_PTR` 是指向undo日志的指针，undo日志包含了数据的历史更新版本的存储，用来实现**事务回滚**以及**保证读一致性**，日志包含插入日志和更新日志。DB_ROW_ID 记录行的自增ID。

行数据的版本存储使用Diff的形式存储不同历史版本数据之间的差异，需要时计算出历史版本的数据。更新日志只有当没有可能读取旧版本快照的事务时才可删除。

对于事务中删除的数据，只有当事务提交后才根据undo日志真正删除物理行



对版本数据即undo日志的回收过程称为[purge](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_purge)，

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