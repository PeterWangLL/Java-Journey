# Data Storage on Disk

上节我们提到更新数据时，先将日志记录到redo log buffer和bin log cache中，之后根据刷盘策略将其持久化到磁盘，本节会介绍刷盘策略。

对于redo来说，首先会有后台线程每秒把redo log buffer中的日志数据写入page cache，然后调用fsync刷盘，另外当redo log buffer占用的空间即将达到 innodb\_log\_buffer\_size 一半的时候，后台线程也会主动刷盘。刷盘策略由innodb\_flush\_log\_at\_trx\_commit字段控制，0，表示每次事务提交时不进行刷盘操作，会有1秒数据丢失的风险；1，表示每次事务提交时都将进行刷盘操作；2，表示每次事务提交时都只把 log buffer 里的 redo log 内容写入 page cache，MySQL挂了数据不会丢失，服务器宕机会丢失数据；

对于bin log来说，由sync\_binlog字段控制，0，表示每次提交事务都只write，由操作系统自行判断什么时候执行fsync；1，表示每次提交事务都会执行fsync；折中方案，可以设置为N(N>1)，表示每次提交事务write，等累积N个事务后才fsync；

可以看出，三种策略关注的是性能和安全的平衡点，是要极致的性能，还是数据的安全，或者折中的方案，这要我们根据业务需求谨慎选择。
