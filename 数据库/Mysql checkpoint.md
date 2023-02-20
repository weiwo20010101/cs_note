# Mysql checkpoint

## redo log

什么是缓冲池：缓存磁盘上的数据页，减少磁盘io

什么是是change buffer：先将写操作存在内存中，等到下次需要读取这些操作涉及到的数据页时，就把数据页加载到缓冲池中，然后在缓冲池中更新。

在将写操作写入 redo log 的过程中并不是直接就进行磁盘IO来完成的，而是分为三个步骤。

![img](https://img2020.cnblogs.com/blog/2012006/202101/2012006-20210105101705110-871341100.png)

1、写入 redo log buffer 中，这部分是属于MySQL 的内存中，是**全局公用**的。

2、在事务编写完成后，就可以执行 write 操作，写到文件系统的 page cache （页缓存）中，属于操作系统的内存，如果 MySQL 崩溃不会影响，但如果机器断电则会丢失数据。

3、执行 fsync（持久化）操作，将 page cache 中的数据正式写入磁盘上的 redo log 中，也就是图中的 hard disk。

### redo log的持久化

**1、**持久化策略通过参数 innodb_flush_log_at_trx_commit 控制。

设置为 0 的时候，表示每次事务提交时都只是把 redo log 留在 redo log buffer 中 ; MySQL 崩溃就会丢失。
设置为 1 的时候，表示每次事务提交时都将 redo log 直接持久化到磁盘（**将 redo log buffer 中的操作全部进行持久化，可能会包含其他事务还未提交的记录**）；断电也不会丢失。
设置为 2 的时候，表示每次事务提交时都只是把 redo log 写到 page cache。MySQL 崩溃不会丢失，断电会丢失。

**2、**InnoDB 后台还有一个线程会每隔一秒钟将 redo log buffer 中记录的操作执行 write 写到 page cache，然后再 fsync 到磁盘上。 



未提交的事务操作也可能会持久化，未提交事务操作的持久化触发场景如下：

1、redo log buffer 被占用的空间达到 innodb_log_buffer_size（redo log buffer 大小参数）的一半时，后台会主动写盘，无论是否是已完成的事务操作都会执行。

2、innodb_flush_log_at_trx_commit 设为 1 时，在每次事务提交时，都会将 redo log buffer 中的所有操作（包括未提交事务的操作）都进行持久化。

3、后台有线程每秒清空 redo log buffer 进行落盘。