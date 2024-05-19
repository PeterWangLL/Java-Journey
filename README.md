# Why does InnoDB use B+Tree?

通过本文的学习，希望你可以理解索引的底层数据结构，在此基础上，很多索引设计的技巧和优化方式就水到渠成了；

MySQL默认的存储引擎是InnoDB，而InnoDB就是使用了B+树作为索引的实现，这里首先介绍B+树这种数据结构，为什么在众多数据结构中会选择它？

以下分别说明其他常见数据结构（请自行了解）的缺点：

哈希表：不支持范围查询；

二叉查找树：极端情况下会退化成链表；

平衡二叉树：插入节点时维护成本高；

红黑树。。。

所有的二叉树在面对B树时都是完败的，因为InnoDB操作数据时，首先需要将磁盘上的数据加载到内存中，然后在内存中对数据进行加工，而一次磁盘IO可以理解为从某个节点到子节点到过程，那么树的高度/深度越小，磁盘IO越少，加载数据也就越快。

从名字上也能看出，B+树是在B树的基础上进化而来的，B+树在非叶子节点上只存储索引，在叶子节点上存储索引和数据，并且叶子节点有一条引用链指向与它相临的叶子节点（以表的一行记录为例，索引就是指索引字段，数据就是行记录）

上面说到InnoDB需要将数据加载到内存，它每次会以页（16KB）为单位加载数据，并且单个索引占的空间相比存的数据而言是非常小的，也就是说一次磁盘IO相比B树能够加载的索引树指数级地增加了，那么单个节点的子节点就大大增加了，即树的高度能够显著减小，上面说了树的高度越小，磁盘IO越少。

同时，所有的叶子节点通过指针相连，在范围查询时只要找到第一个满足条件的节点，后面通过指针顺序遍历即可，非常便捷高效。

说了这么多，其实宗旨就是较少磁盘IO。
