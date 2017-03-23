# CC

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