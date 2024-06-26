# Execution of an SQL  statement

本节介绍一条SQL语句的执行过程，需要先掌握MySQL的基础架构后再来看本节。

以查询操作举例，客户端想要执行一条SQL语句，首先需要通过连接器的权限校验，然后查询缓存（8.0之前如果开启该功能的话），依次经过分析器、优化器，到执行器后，执行器会调用存储提供的接口得到数据，如果查询时有使用到不止一个索引，那么会根据优化器得出的结论根据其中一个索引调接口，在执行器也就是Server层对返回数据再做筛选，如果是联合索引并且使用恰当的情况下，5.6之后会做索引下推，这是查询的基本过程。

更新的情况更加复杂，前面都和查询过程一样，执行器也要先把数据查出来，修改后再调接口写入这条新数据。

如果数据页不在内存中，那么会先将更新指令存到change buffer中，等查询要用到该数据页时再去做更新。

存储引擎收到执行器的调用，将这条新数据更新到内存中，同时将这个更新操作记录到redo log buffer里面，此时redo log处于prepare状态。然后告知执行器执行完成了，随时可以提交事务。

执行器生成这个操作的binlog，并把binlog写入磁盘（具体是先存到binlog cache，写入磁盘时机要看刷盘策略）。

执行器调用引擎的提交事务接口，引擎把刚刚写入的redo log改成提交（commit）状态，更新完成。

为什么要分两阶段提交？为了保证redo log和bin log之间的数据一致性，无论是redo log还是bin log先更新，当一个更新完另一个没更新时MySQL挂了，数据就会不一致，而采用两阶段提交，在bin log更新完之前挂了，则事务回滚，如果bin log更新完了，则redo log一定是prepare状态，此时MySQL挂了，重启时会检查bin log是否完整，如果完整则commit redo log，否则回滚事务，保证数据一致性。

OK，本节主要介绍了查询和更新SQL的具体执行过程，希望可以帮助你对前面所讲建立一个系统的认识。

