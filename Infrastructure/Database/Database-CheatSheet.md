[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# 数据库技术清单盘点

![](https://cdn-images-1.medium.com/max/1600/1*YRTutyyz56U7IaT3EzpQJw.png)

The field of databases has a long history. Many different kinds of databases have been developed since the first relational DB was invented in 1970. For the longest time, relational databases, such as MySQL and Oracle, were state-of-the-art for any type of application that was developed. At that time, monoliths were the standard architecture for application development.

At some point, NoSQL databases entered the game. MongoDB, Redis and Neo4j, to name a few, fall into that category. The goal of NoSQL was to either provide a simpler interface to working with data or cater to specific use cases and choose different optimization strategies.

Nowadays, databases are becoming available as globally distributed systems such as Google Spanner or CockroachDB. These kinds of databases are horizontally scalable and come with automatic replication features as well as guarantees for consistency, availability and scalability.

![](https://cdn-images-1.medium.com/max/1600/1*el8WJKzpZYR2r9k0TvaJag.png)

Similar to the historic development of database technologies, the ways how databases are hosted also has evolved quite a bit over the past decades.

In the early days of the web everybody had to run databases on their own physical servers. EC2 and Digital Ocean made this easier, but a deep technical understanding is still required to manually operate database.

Managed services such as Heroku’s Postgres Service, AWS RDS and Mongo Atlas abstracted away a lot of the complex details. Database management has become simpler, but the underlying model is still the same, requiring developers to provision compute capacity upfront.

The most recent development, serverless databases, frees developers from worrying about infrastructure, as their database simply scales up and down to match the current load while paying based on actual usage. Aurora Serverless and CosmosDB are prominent examples.

# Database

2017 年的 Oracle Open World 大会上，Oracle 总裁拉里·埃里森公布了新杀器，Oracle 自治数据库云。这款全球首款“自动驾驶”的数据库，集成了人工智能和自适应的机器学习技术，实现全面自动化。Oracle 自治数据库云，消除了复杂性、人为错误和人工管理，能够以更低的成本提供更高的可靠性、安全性和运营效率。

# NoSQL

NoSQL 家族主要分为键值(Key-Value)存储数据库、列存储数据库、文档型数据库和图数据库四大类，其产生就是为了解决大规模数据集合多重数据种类带来的挑战，故场景化也格外明显。

键值存储数据库适用于内容缓存，主要用于处理大量数据的高访问负载，也用于一些日志系统等。

列存储数据库适用于分布式的文件系统；文档型数据库适用于 Web 应用(与 Key-Value 类似，Value 是结构化的，不同的是数据库能够了解 Value 的内容)。

图数据库适用于社交网络、推荐系统等，专注于构建关系图谱，如果与 AI 结合起来，我们可以设想一下他们美好的未来。

# New SQL

一些过去不支持 SQL 的非关系型分布式数据库也开始拥抱传统数据库的特性，例如标准 SQL 支持、OLTP 特性与 ACID 支持等。

新型分布式数据库的管理和应用方向则集中在两个领域。第一，大数据分析相关，针对海量数据的挖掘、复杂的分析计算；第二，在线数据操作，包括分布式交易型操作、和海量数据的实时访问与高并发查询操作。一般来说，用户会根据业务场景以及对数据处理结果的期望选择不同的分布式数据库产品。

Multi-Model 多模是指在单个数据库平台中支持非结构化结构化数据在内的多种数据类型。一直以来，传统关系型数据库仅支持表单类型的结构化数据存储和访问能力，而对于层次型对象、图片影像等半结构化与非结构化数据管理无能为力。如今，随着应用类型的多样化和存储成本的降低，单一数据类型已经无法满足许多综合性业务平台的需求。数据库层面的 Multi-Model 和非结构化数据管理，将能实现结构化、半结构化和非结构化数据的统一管理，实现非结构化数据的实时访问，大大降低了运维和应用的成本。同时，非关系型数据库在访问模式上也渐渐将 SQL、K/V、文档、宽表、图等分支互相融合，支持除了 SQL 查询语言之外的其他访问模式，大大丰富了过去 NoSQL 数据库单一的设计用途。

2017 年，Amazon 在 SIGMOD 上发表了论文《Amazon Aurora: Design Considerations for High Throughput Cloud Native Relational Databases》，描述了 Amazon 的云数据库 Aurora 的架构。基于 MySQL 的 Aurora 对于单点写多点读的主从架构做了进一步的发展，使得事务和存储引擎分离，为数据库架构的发展提供了具有实战意义的已实践用例。其主要特点如下：

* 实践了“日志即数据库”(参考《High Performance Transactions in Deuteronomy》) 的理念。
* 事务引擎和存储引擎分离。数据缓冲区提前预热。
  * REDO 日志从事务引擎中剥离，归并到存储引擎中。
  * 储存层可以有 6 个副本，多个副本之间通过 Gossip 协议可以保障数据的“自愈”能力。
  * 主备服务的备机可达 15 份，提供强大的读服务能力。
* 持续可靠的云数据库服务能力。
  * 数据存储跨多个区：提供了多级别容灾能力。
  * 数据容灾能力：数据冗余、备份、实时恢复等多种能力集成到云服务，提高数据保障能力。

Aurora 对国内的计算与存储分离的产品研发影响深远，阿里的 PolarDB、X-DB，华为的 FusionInsight 系列等都在向 Aurora 对齐。相传，腾讯、京东等都跃跃欲试准备做类 Aurora 的产品。可见 Aurora 对国内的影响深远。

2012 年的《Spanner: Google’s Globally-Distributed Database》论文描述了基于 KV 系统(《Bigtable: A Distributed Storage System for Structured Data》)实现的一个半数据库式的“分布式系统”(《Spanner: Becoming a SQL System》：Spanner is built on ideas from both the systems and database communities.) ，这个系统具备了大规模的扩展性，具有如下几个方面的特色：可扩展性、自动分片、容错性、一致性复制、外部一致性，和数据广域分布。这些特色是通过提供了多行事务、外部一致性、跨数据中心的透明故障转移等功能实现的。Spanner 开创了 NoSQL 分布式数据库的新时代，主要解决了如下问题：

    1. 数据分布

    2. 多副本高可用：failover

    3. 分布式事务处理：外部一致

    4. 计算分布(通过F1支持SQL，松耦合结构)

    5. KV存储模型

2017 年，Google 发表了一篇题为《Spanner: Becoming a SQL System》的论文。这篇论文描述了查询执行的切分、瞬态故障情况下查询重新执行、驱动查询做路由和索引查找的范围查询，以及改进的基于块的列存等分布式查询优化技术。

# 索引

有多种类型的索引，它们各有各的性能特点，常见的索引包括：

B-Tree 索引
Hash 索引
空间R-Tree索引
全文索引

- [](https://flacro.me/content/images/2016/07/btree-index.png)


B-Tree索引可以很好地用于单行、范围或者前缀扫描，他们只有在查找使用了索引的最左前缀(Leftmost Prefix)的时候才有用。

P.s.

暂时不讨论非前导索引(non-leading index parts)
当然完全前缀匹配更有效率(complete prefix)
对于以下类型的索引，可以使用B-Tree索引：

匹配全部索引列
匹配最左前缀
匹配列前缀
匹配范围值
精确匹配一部分(前一部分)，并匹配某个范围内的另一部分
覆盖索引，B-Tree索引通常能支持只访问索引的查询
此外，由于树的节点是排好序的，可以用于查找，那么也可以用同样的方式排序，所以也可以用于ORDER BY。

B-Tree索引存在一些限制：

如果查找不从索引列的最左边开始，索引就无法使用；同样，不能查找字符串结尾
不能跳过索引中的列
不能使用任何在第一个范围条件右边的列作为条件
因此B-Tree的列顺序非常重要，上述使用规则都和列顺序有关。对于实际的应用，一般要根据具体的需求，创建不同列和不同列顺序的索引。

举例如下： 假设有索引index (A, B, C)

使用索引：

A>5 AND A<10 - 最左前缀匹配
A=5 AND B>6 - 最左前缀匹配
A=5 AND B=6 AND C=7 - 全列匹配
A=5 AND B IN (2,3) AND C>5 - 最左前缀匹配，填坑
不能使用索引：

B>5 - 没有包含最左前缀
B=6 AND C=7 - 没有包含最左前缀
使用部分索引：

A>5 AND B=2 - 使用索引A列
A=5 AND B>6 AND C=2 - 使用索引的A和B列
使用索引对结果进行排序，需要索引的顺序和ORDER BY子句中的顺序一致，并且所有列的升降序一致(ASC/DESC)。如果查询连接了多个表，只有在ORDER BY的列引用的是第一个表才可以(需要按序JOIN)。

使用索引排序：

ORDER BY A - 最左前缀匹配
WHERE A=5 ORDER BY B,C - 最左前缀匹配
WHERE A=5 ORDER BY B DESC - 最左前缀匹配
WHERE A>5 ORDER BY A,B - 最左前缀匹配
不能使用索引排序：

WHERE A=5 ORDER BY B DESC,C ASC - 升降序不一致
WHERE A=5 ORDER BY B,D - D不在索引中
WHERE A=5 ORDER BY C - 没有包含最左前缀
WHERE A>5 ORDER BY B,C - 第一列是范围条件，无法使用BC排序
WHERE A=5 AND B IN(1, 2) ORDER BY C - B也是范围条件，无法用C排序


## 
