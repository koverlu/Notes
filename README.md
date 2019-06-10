# Notes
https://www.cnblogs.com/kevingrace/p/8343060.html
https://blog.csdn.net/weixin_41004350/article/details/80479051
9.1.2 插入操作和数据库恢复
在不考虑性能情况下，为了处理上述的系统错误，PG支持WAL机制。下面会介绍一些关键词，关键观点，以及写WAL数据和数据库恢复。
PG将所有的修改作为历史数据写到持久化存储介质，来防止错误发生。这些历史数据被称作XLOG records或WAL data。
当增删改或提交操作发生时，XLOG记录会被写到内存介质WAL buffer中。当一个事务提交或abort，他们会被立即写入到存储介质上的一个WAL segment文件中(实际上，日志的写操作会在其他时候发生，后文详解)。XLOG record的LSN(Log Segment Number)是XLOG record的唯一id，表示这条record写入到segment文件中的实际位置。
考虑到数据库系统恢复，还有个问题，PG要从哪里开始恢复？答案就是REDO point， 最近一个checkpoint的写入点。因此数据库恢复进程和checkpoint进程是紧密相关，不可分离的。
关键词和观点就介绍到这里，下面来介绍插入tuple时的WAL。
9.2 插入操作的WAL
Checkpointer是一个后台进程，周期性地完成checkpoint工作。当Checkpointer开始时，他会写一条checkpoint record到现在的WAL segment。这条record包含了最近的redo point的位置(LSN)。
2. 执行第一条插入操作时，PG从shared buffer pool载入TABLE_A的page，插入一条tuple到这个page，在LSN_1的位置创建并写一条XLOG record到WAL buffer，然后将TABLE_A的LSN从LSN_0更新到LSN_1。
3. 这个事务提交(commit)时，PG会创建并写一条提交操作的XLOG record到WAL buffer，然后将WAL buffer中的从LSN_1起的所有XLOG records刷入(flush)到WAL segment文件中。
4. 执行第二条插入时，PG插入一条新tuple到page，在LSN_2的位置创建并写一条XLOG record到WAL buffer，然后将TABLE_A的LSN从LSN_1更新到LSN_2。
5. 这个事务提交时，同步骤3
6. 在这个点操作系统发生错误挂机。即使shared buffer pool中所有的数据丢失，page的所有改动都已经被写到了WAL segment 文件中。
1. PG从合适的WAL segment文件中读取第一条插入XLog记录，将TABLE_A的页从数据库读取到shared buffer pool。
2. 在回放XLOG记录之前，PG会比较XLOG的LSN和page的LSN。如果XLOG的LSN比page的LSN大，则根据XLOG的内容修改page，并将page的LSN更新到XLOG的LSN；否则就什么都不做，继续读下一个XLOG数据。
3. PG不断重复上述步骤


PG通过按时间顺序回放WAL segment文件中的XLOG记录来恢复。因此PG的XLOG就是redo log。PG不支持undo log。
9.1.3 Full-Page Writes
假设在background-write进程正在写脏页的时候，操作系统挂了，那么存储中TABLE_A的页数据会损坏掉。XLOG记录无法再损坏的页面上回放。
PG支持Full-Page Writes来解决这个问题。当这个特性被打开时，PG会在每个checkpoint之后第一次修改页面时，写一个包括头数据和整个页面的XLOG记录。这个特性默认是开的，在PG中，这种包含整个页面的XLOG记录叫做backup block(或full-page image)
