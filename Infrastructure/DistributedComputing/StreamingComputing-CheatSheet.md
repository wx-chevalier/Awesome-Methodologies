[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# 流计算原理与机制概述

分布式的流处理也就是通常意义上的持续处理、数据富集以及对于无界数据的分析过程的组合。它是一个类似于 MapReduce 这样的通用计算模型，但是我们希望它能够在毫秒级别或者秒级别完成响应。这些系统经常被有向非循环图(Directed ACyclic Graphs,DAGs)来表示。

DAG 主要功能即是用图来表示链式的任务组合，而在流处理系统中，我们便常常用 DAG 来描述一个流工作的拓扑。笔者自己是从 Akka 的 Stream 中的术语得到了启发。如下图所示，数据流经过一系列的处理器从源点流动到了终点，也就是用来描述这流工作。谈到 Akka 的 Streams，我觉得要着重强调下分布式这个概念，因为即使也有一些单机的解决方案可以创建并且运行 DAG，但是我们仍然着眼于那些可以运行在多机上的解决方案。

![](http://cdn2.hubspot.net/hub/323094/hubfs/Petr/Screen_Shot_2015-12-28_at_16.36.48.png?t=1457003818430&width=877)

# 机制与考量

## Runtime Model | 运行模型

主要有两种不同的方法来构建流处理系统: Native Streaming 与 Micro-Batching。

### Native Streaming

Native Streaming 意味着所有输入的记录或者事件都会根据它们进入的顺序一个接着一个的处理。

![](http://cdn2.hubspot.net/hub/323094/hubfs/Petr/Screen_Shot_2015-12-28_at_16.39.08.png?t=1457003818430&width=877)

Native Streaming 的表现性会更好一点，因为它是直接处理输入的流本身的，并没有被一些不自然的抽象方法所限制住。同时，因为所有的记录都是在输入之后立马被处理，这样对于请求方而言响应的延迟就会优于那种 Micro-Batching 系统。处理这些，有状态的操作符也会更容易被实现，我们在下文中也会描述这个特点。不过 Native Streaming 系统往往吞吐量会比较低，并且因为它需要去持久化或者重放几乎每一条请求，它的容错的代价也会更高一些。并且负载均衡也是一个不可忽视的问题，举例而言，我们根据键对数据进行了分割并且想做进一步地处理。如果某些键对应的分区因为某些原因需要更多地资源去处理，那么这个分区往往就会变成整个系统的瓶颈。

### Micro-Batching

Micro-Batching，大量短的 Batches 会从输入的记录中创建出然后经过整个系统的处理，这些 Batches 会根据预设好的时间常量进行创建，通常是每隔几秒创建一批。

![](http://cdn2.hubspot.net/hub/323094/hubfs/Petr/Screen_Shot_2015-12-28_at_16.40.36.png?t=1457003818430&width=877)

对于 Micro-Batching 而言，将流切分为小的 Batches 不可避免地会降低整个系统的变现性，也就是可读性。而一些类似于状态管理的或者 joins、splits 这些操作也会更加难以实现，因为系统必须去处理整个 Batch。另外，每个 Batch 本身也将架构属性与逻辑这两个本来不应该被糅合在一起的部分相连接了起来。而 Micro-Batching 的优势在于它的容错与负载均衡会更加易于实现，它只要简单地在某个节点上处理失败之后转发给另一个节点即可。最后，值得一提的是，我们可以在 Native Streaming 的基础上快速地构建 Micro-Batching 的系统。

## State Management | 状态管理

大部分这些应用都有状态性的逻辑处理过程，因此，框架本身应该允许开发者去维护、访问以及更新这些状态信息。

## Message Delivery Guarantees | 消息的可达性

一般来说，对于消息投递而言，我们有至多一次(at most once)、至少一次(at least once)以及恰好一次(exactly once)这三种方案。

- At most once 投递保证每个消息会被投递 0 次或者 1 次，在这种机制下消息很有可能会丢失。

- At least once 投递保证了每个消息会被默认投递多次，至少保证有一次被成功接收，信息可能有重复，但是不会丢失。

- Exactly once 意味着每个消息对于接收者而言正好被接收一次，保证即不会丢失也不会重复。

## Back Pressure | 背压

流处理系统需要能优雅地处理反压(Back Pressure)问题。反压通常产生于这样的场景：短时负载高峰导致系统接收数据的速率远高于它处理数据的速率。许多日常问题都会导致反压，例如，垃圾回收停顿可能会导致流入的数据快速堆积，或者遇到大促或秒杀活动导致流量陡增。反压如果不能得到正确的处理，可能会导致资源耗尽甚至系统崩溃。目前主流的流处理系统 Storm/JStorm/Spark Streaming/Flink 都已经提供了反压机制，不过其实现各不相同。

- Storm 是通过监控 Bolt 中的接收队列负载情况，如果超过高水位值就会将反压信息写到 Zookeeper ，Zookeeper 上的 watch 会通知该拓扑的所有 Worker 都进入反压状态，最后 Spout 停止发送 tuple。具体实现可以看这个 JIRA STORM-886。

- JStorm 认为直接停止 Spout 的发送太过暴力，存在大量问题。当下游出现阻塞时，上游停止发送，下游消除阻塞后，上游又开闸放水，过了一会儿，下游又阻塞，上游又限流，如此反复，整个数据流会一直处在一个颠簸状态。所以 JStorm 是通过逐级降速来进行反压的，效果会较 Storm 更为稳定，但算法也更复杂。另外 JStorm 没有引入 Zookeeper 而是通过 TopologyMaster 来协调拓扑进入反压状态，这降低了 Zookeeper 的负载。

- Flink 没有使用任何复杂的机制来解决反压问题，因为根本不需要那样的方案！它利用自身作为纯数据流引擎的优势来优雅地响应反压问题。

## Failures Handling | 异常处理

在一个流处理系统中，错误可能经常在不同的层级发生，譬如网络分割、磁盘错误或者某个节点莫名其妙挂掉了。平台要能够从这些故障中顺利恢复，并且能够从最后一个正常的状态继续处理而不会损害结果。

## Programming Model | 编程模型

编程模型往往会决定很多它的特性，并且应该足够处理所有可能的用户案例。很多的流处理系统都会提供 Functional Primitives(函数式单元)，即能够在独立信息级别进行处理的函数，像 map、filter 这样易于实现与扩展的一些函数；同样也应提供像 aggregation 这样的跨信息处理函数以及像 join 这样的跨流进行操作的函数，虽然这样的操作会难以扩展。

对于编程模型而言，又可以分为 Compositional(组合式)与 Declarative(声明式)。组合式会提供一系列的基础构件，类似于源读取与操作符等等，开发人员需要将这些基础构件组合在一起然后形成一个期望的拓扑结构。新的构件往往可以通过继承与实现某个接口来创建。另一方面，声明式 API 中的操作符往往会被定义为高阶函数。声明式编程模型允许我们利用抽象类型和所有其他的精选的材料来编写函数式的代码以及优化整个拓扑图。同时，声明式 API 也提供了一些开箱即用的高等级的类似于窗口管理、状态管理这样的操作符。

# 生态圈

目前已经有了各种各样的流处理框架，自然也无法在本文中全部攘括。所以我必须将讨论限定在某些范围内，本文中是选择了所有 Apache 旗下的流处理的框架进行讨论，并且这些框架都已经提供了 Scala 的语法接口。主要的话就是 Storm 以及它的一个改进 Trident Storm，还有就是当下正火的 Spark。最后还会讨论下来自 LinkedIn 的 Samza 以及比较有希望的 Apache Flink。笔者个人觉得这是一个非常不错的选择，因为虽然这些框架都是出于流处理的范畴，但是他们的实现手段千差万别。

![](http://cdn2.hubspot.net/hub/323094/hubfs/Petr/Screen_Shot_2015-12-28_at_16.43.04.png?t=1457003818430&width=877)

- Apache Storm 最初由 Nathan Marz 以及他的 BackType 的团队在 2010 年创建。后来它被 Twitter 收购并且开源出来，并且在 2014 年变成了 Apache 的顶层项目。毫无疑问，Storm 是大规模流处理中的先行者并且逐渐成为了行业标准。Storm 是一个典型的 Native Streaming 系统并且提供了大量底层的操作接口。另外，Storm 使用了 Thrift 来进行拓扑的定义，并且提供了大量其他语言的接口。

- Trident 是一个基于 Storm 构建的上层的 Micro-Batching 系统，它简化了 Storm 的拓扑构建过程并且提供了类似于窗口、聚合以及状态管理等等没有被 Storm 原生支持的功能。另外，Storm 是实现了至多一次的投递原则，而 Trident 实现了恰巧一次的投递原则。Trident 提供了 Java, Clojure 以及 Scala 接口。

- 众所周知，Spark 是一个非常流行的提供了类似于 SparkSQL、Mlib 这样内建的批处理框架的库，并且它也提供了 Spark Streaming 这样优秀地流处理框架。Spark 的运行环境提供了批处理功能，因此，Spark Streaming 毫无疑问是实现了 Micro-Batching 机制。输入的数据流会被接收者分割创建为 Micro-Batches，然后像其他 Spark 任务一样进行处理。Spark 提供了 Java, Python 以及 Scala 接口。

- Samza 最早是由 LinkedIn 提出的与 Kafka 协同工作的优秀地流解决方案，Samza 已经是 LinkedIn 内部关键的基础设施之一。Samza 重负依赖于 Kafaka 的基于日志的机制，二者结合地非常好。Samza 提供了 Compositional 接口，并且也支持 Scala。

* 最后聊聊 Flink. Flink 可谓一个非常老的项目了，最早在 2008 年就启动了，不过目前正在吸引越来越多的关注。Flink 也是一个 Native Streaming 的系统，并且提供了大量高级别的 API。Flink 也像 Spark 一样提供了批处理的功能，可以作为流处理的一个特殊案例来看。Flink 强调万物皆流，这是一个绝对的更好地抽象，毕竟确实是这样。

下表就简单列举了上述几个框架之间的特性：
![](http://cdn2.hubspot.net/hub/323094/hubfs/Petr/Screen_Shot_2015-12-28_at_16.44.49.png?t=1457003818430&width=877)
