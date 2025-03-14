> [详解 MySql InnoDB 的 MVCC 实现机制_mysql innodb mvcc-CSDN博客](https://blog.csdn.net/mrluo735/article/details/135260663)
> 
> [面试官：说说MVCC？ 事务隔离级别实现原理是什么？_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Hr421p7EK/?spm_id_from=333.337.search-card.all.click&vd_source=2d885cb62bb9393fa8a5379c72eabd82)
> 
> [MVCC多版本并发控制原理总结（最终版） - 郭慕荣 - 博客园 (cnblogs.com)](https://www.cnblogs.com/jelly12345/p/14889331.html)

MVCC，全称多版本并发控制，**是关于事务隔离级别的一种无锁的实现方式**，用于提高事务的执行效率

MySQL用的就是MVCC这种无锁的设计，实现 **读已提交 和 可重复读** 的并发控制

MVCC机制具有以下优点：

- 提高并发性能：读操作不会阻塞写操作，写操作也不会阻塞读操作，有效地提高数据库的并发性能。
- 降低死锁风险：由于无需使用显式锁来进行并发控制，MVCC可以降低死锁的风险。

MVCC有以下三个关键点
### 隐藏列
数据库表中除了已有的数据列，还有三个隐藏列
1. **当前这条数据是哪个事务维护的**（DB_TRX_ID）
2. **指向上一个undoLog回滚事务的指针**（DB_ROLL_PTR）
3. **隐含的自增ID**（DB_ROW_ID，不重要，只在没有主键时才有）
## undoLog
Undo日志的作用主要有两个方面：

- **事务回滚**：当事务需要回滚时，MySQL可以通过Undo日志中的旧值将数据还原到事务开始之前的状态，保证了事务回滚的一致性。
- **MVCC实现**：MVCC 是InnoDB存储引擎的核心特性之一。通过使用Undo日志，MySQL可以为每个事务提供独立的事务视图，使得事务读取数据时能看到一致且符合隔离级别要求的数据版本。

不同事务或者相同事务对同一记录行的修改，会使该记录行的 undo log 成为一条链表，链首就是最新的记录，链尾就是最早的旧记录，通过查找undoLog就可以找到历史的**已提交**的数据

在MVCC中，对于每次更新操作，旧值会被保存到一条undo日志中，即使它是该记录的旧版本。随着更新次数的增加，所有的版本都会通过roll_pointer属性连接成一个链表，称之为**版本链**。

版本链的头节点代表当前记录的最新值。此外，每个版本还包含生成该版本的事务ID。
## readView
是事务进行快照读操作时候生成的读视图，通过其中的四个条件就可以快速地定位需要查找的已提交的数据

| 字段             | 描述名称       | 含义                        |
| -------------- | ---------- | ------------------------- |
| ids            | 未提交事务ID列表  | 当前所有未提交的，活跃的事务ID列表        |
| creator_trx_id | 当前事务ID     | 当前事务，创建该 Read View 的事务 ID |
| min_trx_id     | 最小未提交事务ID  | ids 中最小的事务 ID             |
| max_trx_id     | 未提交事务ID右边界 | 未开始的下一个事务ID               |
通过上面的字段信息，限制可见的undoLog版本号

分别是 **已提交的事务、还未提交的事务 和 还未开始的事务**

如果某个版本的数据对当前事务是不可见的，那就顺着版本链找到下一个版本数据，继续执行上面的步骤来判断记录的可见性。

依次类推，直到版本中的最后一个版本。如果记录的最后一个版本也不可见，意味着该条记录对当前事务完全不可见，查询结果就不包含该记录。

如果**找到了对当前事务可见的记录，将它从undolog中取出来**

## 隔离级别与MVCC的具体关联
- **读未提交**：无需锁、无需MVCC，直接改数据源，可能出现脏读
- **读已提交**：每次查询都会创建readView读取数据
- **可重复读**：同样的查询只会在第一次查询创建readView，所以对事务中之后的修改不可见
- **序列化**：表锁，颗粒度非常大

## 小结

原始的表锁和并发行锁颗粒度太大，数据库操作不够精细于是就有了MVCC

MVCC，**是关于事务隔离级别的一种无锁的实现方式**，用于提高事务的执行效率，实现 **读已提交 和 可重复读** 的并发控制。

主要依赖Undolog和ReadView实现

Undolog：每个记录都有三个隐藏列，分别代表事务ID、下一条记录的指针、隐藏主键

ReadView：根据事务ID-版本号，**找到对当前事务可见的记录**，达到符合隔离级别要求的能力

**读已提交**：每次查询都会创建readView读取数据

**可重复读**：同样的查询只会在第一次查询创建readView