
## Innodb 是如何实现事务的
Innodb 通过Buffer Pool，LogBuffer， Redo Log , undo log 来实现事务的，以update语句为例
1. Innodb 在收到一个update语句后，会先根据条件找到数据所在的页，并将该页缓存在Buffer Pool中（？changebuffer的存在不需要找到）
2. 执行update语句，修改Buffer Pool中的数据，也就是已存在的数据
3. 针对update语句生成一个redo log对象，并存入logbuffer中
4. 针对update语句生成一个undo log日志，用于事务回滚，
5. 如果事务提交，则将redo log对象进行持久化，后续还有其他机制将Buffer Pool中所修改的数据页持久化到磁盘中
6. 如果事务回滚，则利用undo log进行回滚。



## ACID是靠什么保证的？

A 原子性由undo log日志保证，它记录了需要回滚的日志信息，事务回滚时撤销已经执行成功的sql
C 一致性由其他三大特性保证，程序代码要保证业务上的一致性

I 隔离性由MVCC来保证

D 持久性由内存+redo log来保证，mysql修改数据同时在内存和redo log记录这次操作，宕机的时候可以从redo log恢复。

 InnoDB rodo log写盘，InnoDB 事务进入prepare状态。
 如果前面prepare成功，binlog写盘，再继续将事务日志持久化到binlog，如果持久化成功，那么InnoDB 事务则进入commit状态 （在redo log里面写一个commit记录）


 redo log的刷盘会在系统空闲时进行。