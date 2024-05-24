# Redo Log\&Bin Log

MySQL主要有三大日志，分别为redo log、bin log、undo log。本节介绍redo log和bin log。

首先我们要明确日志的作用，undo log用于事务回滚，我们在事务部分就重点讲，redo log用于持久化，bin log用于主从同步。

MySQL架构分为Server层和存储引擎两部分，存储引擎是插件式的，现在MySQL默认InnoDB为存储引擎，这之前的是MyISAM，redo log只有在InnoDB中实现，而bin log是在Server层实现的，所以最初的MySQL就是通过bin log记录日志的，后来随着InnoDB的引入才有了redo log。

redo log是物理日志，记录了“在某个数据页上做了哪些修改”，大小固定，InnoDB特有。当某条数据需要更新时，InnoDB会将其写入redo log buffer，并更新数据页，然后在适当的时机将其更新到磁盘（取决于刷盘策略），别忘了，还会记录到change buffer中哦！

redo log大小是固定的，这里介绍两个指针，write pos表示暂存到redo log中数据的最新位置，check point表示已更新到磁盘的位置，它们之间的数据就是尚未更新到磁盘的数据，redo log是顺序写，随着指针后移到末尾，再从起点重新开始，check point永远要比write pos靠后，当write pos与check pos重合时，有两种情况，第一种redo log已经全部更新到磁盘中了，第二种redo log记满了，这时必须停下来刷盘，推进一下check pos，这就和操场跑步套圈类似。

为什么InnoDB要引入redo log呢，加入没有redo log，每执行一条事务，就将数据页刷盘，要知道数据页大小16KB，有时更新的只有几Byte，并且数据页刷盘是随机写，肯定是划不来的，引入redo log后，它记录的是在某个数据页上做的修改，一行记录很小，加上是顺序写，配合刷盘策略攒多一点一起刷，效率高多了。

再提一下change buffer，它主要节省的是随机读磁盘的IO消耗，而redo log节省的是随机写磁盘的IO消耗。

接下来是bin log，它是逻辑日志，记录的就是SQL语句的原始逻辑，采用追加写，Server层实现，主要用于数据备份。

事务执行过程中，先将日志写到bin log cache，等事务提交后再根据刷盘策略同步。

在之后的执行过程章节中，我会将redo log，bin log，change buffer等串联起来，先挖个坑。
