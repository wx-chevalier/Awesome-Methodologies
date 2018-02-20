# 数据库技术清单盘点

# Database

2017 年的 Oracle Open World 大会上，Oracle 总裁拉里·埃里森公布了新杀器，Oracle 自治数据库云。这款全球首款“自动驾驶”的数据库，集成了人工智能和自适应的机器学习技术，实现全面自动化。Oracle 自治数据库云，消除了复杂性、人为错误和人工管理，能够以更低的成本提供更高的可靠性、安全性和运营效率。

# NoSQL

NoSQL 家族主要分为键值（Key-Value）存储数据库、列存储数据库、文档型数据库和图数据库四大类，其产生就是为了解决大规模数据集合多重数据种类带来的挑战，故场景化也格外明显。

键值存储数据库适用于内容缓存，主要用于处理大量数据的高访问负载，也用于一些日志系统等。

列存储数据库适用于分布式的文件系统；文档型数据库适用于 Web 应用（与 Key-Value 类似，Value 是结构化的，不同的是数据库能够了解 Value 的内容）。

图数据库适用于社交网络、推荐系统等，专注于构建关系图谱，如果与 AI 结合起来，我们可以设想一下他们美好的未来。

# New SQL

2017 年，Amazon 在 SIGMOD 上发表了论文《Amazon Aurora: Design Considerations for High Throughput Cloud Native Relational Databases》，描述了 Amazon 的云数据库 Aurora 的架构。基于 MySQL 的 Aurora 对于单点写多点读的主从架构做了进一步的发展，使得事务和存储引擎分离，为数据库架构的发展提供了具有实战意义的已实践用例。其主要特点如下：

* 实践了“日志即数据库”（参考《High Performance Transactions in Deuteronomy》） 的理念。
* 事务引擎和存储引擎分离。数据缓冲区提前预热。
  * REDO 日志从事务引擎中剥离，归并到存储引擎中。
  * 储存层可以有 6 个副本，多个副本之间通过 Gossip 协议可以保障数据的“自愈”能力。
  * 主备服务的备机可达 15 份，提供强大的读服务能力。
* 持续可靠的云数据库服务能力。
  * 数据存储跨多个区：提供了多级别容灾能力。
  * 数据容灾能力：数据冗余、备份、实时恢复等多种能力集成到云服务，提高数据保障能力。

Aurora 对国内的计算与存储分离的产品研发影响深远，阿里的 PolarDB、X-DB，华为的 FusionInsight 系列等都在向 Aurora 对齐。相传，腾讯、京东等都跃跃欲试准备做类 Aurora 的产品。可见 Aurora 对国内的影响深远。

2012 年的《Spanner: Google’s Globally-Distributed Database》论文描述了基于 KV 系统（《Bigtable: A Distributed Storage System for Structured Data》）实现的一个半数据库式的“分布式系统”（《Spanner: Becoming a SQL System》：Spanner is built on ideas from both the systems and database communities.） ，这个系统具备了大规模的扩展性，具有如下几个方面的特色：可扩展性、自动分片、容错性、一致性复制、外部一致性，和数据广域分布。这些特色是通过提供了多行事务、外部一致性、跨数据中心的透明故障转移等功能实现的。Spanner 开创了 NoSQL 分布式数据库的新时代，主要解决了如下问题：

    1. 数据分布

    2. 多副本高可用：failover

    3. 分布式事务处理：外部一致

    4. 计算分布（通过F1支持SQL，松耦合结构）

    5. KV存储模型

2017 年，Google 发表了一篇题为《Spanner: Becoming a SQL System》的论文。这篇论文描述了查询执行的切分、瞬态故障情况下查询重新执行、驱动查询做路由和索引查找的范围查询，以及改进的基于块的列存等分布式查询优化技术。
