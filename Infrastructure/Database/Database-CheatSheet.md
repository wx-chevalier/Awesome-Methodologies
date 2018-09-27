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

- 实践了“日志即数据库”(参考《High Performance Transactions in Deuteronomy》) 的理念。
- 事务引擎和存储引擎分离。数据缓冲区提前预热。
  - REDO 日志从事务引擎中剥离，归并到存储引擎中。
  - 储存层可以有 6 个副本，多个副本之间通过 Gossip 协议可以保障数据的“自愈”能力。
  - 主备服务的备机可达 15 份，提供强大的读服务能力。
- 持续可靠的云数据库服务能力。
  - 数据存储跨多个区：提供了多级别容灾能力。
  - 数据容灾能力：数据冗余、备份、实时恢复等多种能力集成到云服务，提高数据保障能力。

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
空间 R-Tree 索引
全文索引

- [](https://flacro.me/content/images/2016/07/btree-index.png)

B-Tree 索引可以很好地用于单行、范围或者前缀扫描，他们只有在查找使用了索引的最左前缀(Leftmost Prefix)的时候才有用。

P.s.

暂时不讨论非前导索引(non-leading index parts)
当然完全前缀匹配更有效率(complete prefix)
对于以下类型的索引，可以使用 B-Tree 索引：

匹配全部索引列
匹配最左前缀
匹配列前缀
匹配范围值
精确匹配一部分(前一部分)，并匹配某个范围内的另一部分
覆盖索引，B-Tree 索引通常能支持只访问索引的查询
此外，由于树的节点是排好序的，可以用于查找，那么也可以用同样的方式排序，所以也可以用于 ORDER BY。

B-Tree 索引存在一些限制：

如果查找不从索引列的最左边开始，索引就无法使用；同样，不能查找字符串结尾
不能跳过索引中的列
不能使用任何在第一个范围条件右边的列作为条件
因此 B-Tree 的列顺序非常重要，上述使用规则都和列顺序有关。对于实际的应用，一般要根据具体的需求，创建不同列和不同列顺序的索引。

举例如下： 假设有索引 index (A, B, C)

使用索引：

A>5 AND A<10 - 最左前缀匹配
A=5 AND B>6 - 最左前缀匹配
A=5 AND B=6 AND C=7 - 全列匹配
A=5 AND B IN (2,3) AND C>5 - 最左前缀匹配，填坑
不能使用索引：

B>5 - 没有包含最左前缀
B=6 AND C=7 - 没有包含最左前缀
使用部分索引：

A>5 AND B=2 - 使用索引 A 列
A=5 AND B>6 AND C=2 - 使用索引的 A 和 B 列
使用索引对结果进行排序，需要索引的顺序和 ORDER BY 子句中的顺序一致，并且所有列的升降序一致(ASC/DESC)。如果查询连接了多个表，只有在 ORDER BY 的列引用的是第一个表才可以(需要按序 JOIN)。

使用索引排序：

ORDER BY A - 最左前缀匹配
WHERE A=5 ORDER BY B,C - 最左前缀匹配
WHERE A=5 ORDER BY B DESC - 最左前缀匹配
WHERE A>5 ORDER BY A,B - 最左前缀匹配
不能使用索引排序：

WHERE A=5 ORDER BY B DESC,C ASC - 升降序不一致
WHERE A=5 ORDER BY B,D - D 不在索引中
WHERE A=5 ORDER BY C - 没有包含最左前缀
WHERE A>5 ORDER BY B,C - 第一列是范围条件，无法使用 BC 排序
WHERE A=5 AND B IN(1, 2) ORDER BY C - B 也是范围条件，无法用 C 排序

## NoSQL

NoSQL（Not Only SQL，不限于 SQL）是一类范围非常广泛的持久化解决方案，它们不遵循关系数据库模型，也不使用 SQL 作为查询语言。其数据存储可以不需要固定的表格模式，也经常会避免使用 SQL 的 JOIN 操作，一般有水平可扩展的特征。

简言之，NoSQL 数据库可以按照它们的数据模型分成 4 类：

- 键-值存储库（Key-Value-stores）;
- BigTable 实现（BigTable-implementations）;
- 文档库（Document-stores）;
- 图形数据库（Graph Database）;

# 索引

# 事务与锁

## 事务隔离

事务是由一组 SQL 语句组成的逻辑处理单元，事务具有以下 4 个属性，通常简称为事务的 ACID 属性。

- 原子性（Atomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。
- 一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。这意味着所有相关的数据规则都必须应用于事务的修改，以保持数据的完整性；事务结束时，所有的内部数据结构（如 B 树索引或双向链表）也都必须是正确的。
- 隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。这意味着事务处理过程中的中间状态对外部是不可见的，反之亦然。
- 持久性（Durable）：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。

- 表锁： 开销小，加锁快；不会出现死锁；锁定力度大，发生锁冲突概率高，并发度最低
- 行锁： 开销大，加锁慢；会出现死锁；锁定粒度小，发生锁冲突的概率低，并发度高
- 页锁： 开销和加锁速度介于表锁和行锁之间；会出现死锁；锁定粒度介于表锁和行锁之间，并发度一般

相对于串行处理来说，并发事务处理能大大增加数据库资源的利用率，提高数据库系统的事务吞吐量，从而可以支持更多的用户。但并发事务处理也会带来一些问题，主要包括以下几种情况。

- **更新丢失（Lost Update）：**当两个或多个事务选择同一行，然后基于最初选定的值更新该行时，由于每个事务都不知道其他事务的存在，就会发生丢失更新问题－－最后的更新覆盖了由其他事务所做的更新。例如，两个编辑人员制作了同一文档的电子副本。每个编辑人员独立地更改其副本，然后保存更改后的副本，这样就覆盖了原始文档。最后保存其更改副本的编辑人员覆盖另一个编辑人员所做的更改。如果在一个编辑人员完成并提交事务之前，另一个编辑人员不能访问同一文件，则可避免此问题。
- **脏读（Dirty Reads）：**一个事务正在对一条记录做修改，在这个事务完成并提交前，这条记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做"脏读"。
- **不可重复读（Non-Repeatable Reads）：**一个事务在读取某些数据后的某个时间，再次读取以前读过的数据，却发现其读出的数据已经发生了改变、或某些记录已经被删除了！这种现象就叫做“不可重复读”。
- **幻读（Phantom Reads）：**一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为“幻读”。

不同级别的现象描述如下：

- 脏读: 当一个事务读取另一个事务尚未提交的修改时，产生脏读。

- 不可重复读: 同一查询在同一事务中多次进行，由于其他提交事务所做的修改或删除，每次返回不同的结果集，此时发生不可重复读。

- 幻读: 同一查询在同一事务中多次进行，由于其他提交事务所做的插入操作，每次返回不同的结果集，此时发生幻读。

| 隔离级别                    | 脏读 / Dirty Read | 不可重复读 / Non Repeatable Read | 幻读 / Phantom Read | 锁             |
| --------------------------- | ----------------- | -------------------------------- | ------------------- | -------------- |
| 未提交读 / Read Uncommitted | 可能              | 可能                             | 可能                | 无事务         |
| 提交读 / Read Committed     | 不可能            | 可能                             | 可能                | 事务保护       |
| 可重复读 / Repeatable Read  | 不可能            | 不可能                           | 可能                | 行锁，顺序写   |
| 串行化 / Serializable       | 不可能            | 不可能                           | 不可能              | 表锁，顺序读写 |

Oracle 只提供 Read committed 和 Serializable 两个标准隔离级别，另外还提供自己定义的 Read only 隔离级别；SQL Server 除支持上述 ISO/ANSI SQL92 定义的 4 个隔离级别外，还支持一个叫做“快照”的隔离级别，但严格来说它是一个用 MVCC 实现的 Serializable 隔离级别。MySQL 支持全部 4 个隔离级别，但在具体实现时，有一些特点，比如在一些隔离级别下是采用 MVCC 一致性读，但某些情况下又不是，这些内容在后面的章节中将会做进一步介绍。

# 数据库扩展

数据库分片
垂直分片
字段拆分，将变化频率不同的字段拆分到不同表
水平分片
哈希算法来分，数据离散度高，降低热点可能性
通过时间范围分片，保证数据连续性
按照业务种类划分，比如数据分类，租户分离等
分片设计要点
分片要预留足够空间，避免重新分片
分片聚合要并行去做
业务尽可能不去做跨分片的事务

# 存储模式

传统的关系数据库如 Oracle、DB2、Mysql 等都是将数据以行记录为单位进行组织，所以数据读写操作需要遍历行记录中所有的列，在存储机制上，行存储将行记录中各列的数据值串在一起进行存储，并且先存完第一行再存第二行。而列存储是一种区别于传统行存储的新型读写模式，列存储以列数据集合为单位进行存储，是把一列中的数据值串在一起进行存储，先存完第一列，再存第二列。

|          | **行式存储**                                                                      | **列式存储**                                                                                       |
| -------- | --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| **优点** | 数据被保存在一起，INSERT/UPDATE 容易                                              | 查询时只有涉及到的列会被读取，投影(projection)很高效，任何列都能作为索引                           |
| **缺点** | 选择(Selection)时即使只涉及某几列，所有数据也都会被读取                           | 选择完成时，被选择的列要重新组装，INSERT/UPDATE 比较麻烦                                           |
| **场景** | 适合数据存储写入、更新较多的场景，比如 OLTP。当数据量很大时，查询性能远不及列存储 | 不适合数据频繁写入、更新的场景，主要适合频繁查询的场景，大数据环境下优势更明显，比如分布式实时查询 |

从写方面来看，行存储的写入是一次完成，也就是说行存储各字段写入是在一次 IO 中完成，而列存储由于需要把一行记录拆分成单列保存，写入次数明显比行存储多，也就是说要多次 IO 才能完成，而字段越多，IO 次数越多，所以在写入上行存储具有明显的优势。再从读方面来看，行存储通常将一行数据完全读出，如果只需要其中几列数据的情况，就会存在冗余列，出于缩短处理时间的考量，消除冗余列的过程通常是在内存中进行的，而列存储读取则只需要读取关心的几个列数据，明显减少了读取的数据量，如果行记录字段越多性能优势就越明显。

另外由于列存储的每一列数据类型是同质的，不存在二义性问题，比如说某列数据类型为整型（int），那么它的数据集合一定是整型数据。这种情况使数据解析变得十分容易，而且因为存储单元的数据结构一致，可以很容易利用差值编码等压缩方法来进行压缩存储而且压缩比可以非常高。相比之下，行存储则要复杂得多，因为在一行记录中保存了多种类型的数据，数据存储不容易压缩，数据解析需要在多种数据类型之间频繁转换，这个操作很消耗 CPU，增加了解析的时间。另外列存储相对行存储更为灵活，逻辑上一行数据可以按列将数据存放到不同的地方，也可以是不同机器上，这样可以在执行查询分析任务时可以按机器节点并行进行查询计算，而行存储不容易进行并行查询，需要一定中间件的支持。

这里以 Spark MaxCompute 为例，阐述列存储模式的压缩与检索技巧，经过字典表进行数据压缩后，表中的字符串才都变成数字了。正因为每个字符串在字典表里只出现一次了，所以达到了压缩的目的:

![image](https://user-images.githubusercontent.com/5803001/46093189-41c88080-c1e9-11e8-872d-8d1aad514772.png)

在查询过程中，查询引擎仅去字典表里找到字符串对应数字(只进行一次字符串比较)，然后用数字去列表里匹配，匹配上的位置设为 1；把不同列的匹配结果进行位运算得到符合所有条件的记录下标，使用这个下标组装出最终的结果集。

![image](https://user-images.githubusercontent.com/5803001/46093255-6ae91100-c1e9-11e8-8bbc-5bb8bf7936ab.png)
