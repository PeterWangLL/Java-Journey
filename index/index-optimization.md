# Index Optimization

本节讨论使用索引的一些注意事项，以及背后的原因。

限制每张表的索引数量，建议控制在5个以内，因为索引不止需要额外的空间，更新索引的成本会随着索引数量的增加而变大，每添加一条数据，对应的维护成本也就越大，并且还会增加MySQL优化器生成执行计划的时间。

被频繁更新的字段应该慎重建立索引，因为增加了维护B+树的成本。

尽可能考虑建立联合索引，结合业务实际需要。

字符串类型的字段使用前缀索引代替普通索引，因为B+树的key域越小，同样三层树高可以存的记录就越多。

避免索引失效，下面是几种常见的索引失效场景：

未遵循最左匹配原则；

查询的范围过大，如like%，%在左不走索引，%在右走索引，本质就是查询范围过大导致的；在比如in，当结果集大于30%时索引会失效。

有更优的选择，这里的优是由优化器判断得出的，它认为全表扫描的速度比回表快时，就不会走索引，比如order by。

更改字段，发生在where子句使用函数或计算时，因为索引保存的是字段的原始值，经过函数计算后自然就不能走索引了。

隐式转换，当操作符号左右两边的数据类型不一致时，会发生隐式转换，一般是两边都转换成浮点数进行比较。

我们当然希望索引不要失效，当不可避免地会发生索引失效的情况，这时我们可以使用EXPLAIN命令分析SQL语句的执行计划，排查索引是否失效，常见的几个字段，ALL代表全表扫描，range代表对索引列进行了范围查询，ref代表使用普通索引作为查询条件，通常我们看到range或ref就说明索引被正确地使用了。

好的，这节内容就这么多，本文仅列举了常见的关于索引优化的tips，很多点都联系到了索引底层的B+树，可见其重要性。
