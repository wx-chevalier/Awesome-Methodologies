[TOC]

# Introduction

完整的软件体系结构由不同的模式或者风格构成，常见的风格有架构设计模式、框架设计模式与驱动模式等。

## Reference

### Books & Tutorials

* [ Software Architecture and Design](https://msdn.microsoft.com/en-us/library/ee658098.aspx)

### Practice

* [从无到有：微信后台系统的演进之路](https://mp.weixin.qq.com/s?__biz=MzI5MDAwOTIzOQ==&mid=402045684&idx=1&sn=5690281c941cd8eb203b6980cdae73ce)

# 架构风格(Architectural Style)

## MDA

## SOA

* Boundaries are explicit
* Services are autonomous
* Services share schema and contract, not class
* Service compatibility is based on policy

## MicroService(微服务)

微服务架构模式(Microservices Architecture Pattern)的目的是将大型的、复杂的、长期运行的应用程序构建为一组相互配合的服务，每个服务都可以很容易得局部改良。Micro 这个词意味着每个服务都应该足够小，但是，这里的小不能用代码量来比较，而应该是从业务逻辑上比较——符合 SRP 原则的才叫微服务。微服务可以认为是 SOA 的一种实现方案，去除了 ESB 的 SOA。ESB 是 SOA 架构中的中心总线，设计图形应该是星形的，而微服务是去中心化的分布式软件架构。

#### Reference

* Tutorials & Docs

- Practice & Resources

> * [an-introduction-to-microservices](https://auth0.com/blog/2015/09/04/an-introduction-to-microservices-part-1/)
> * [微服务(Microservices)](http://blog.csdn.net/wurenhai/article/details/37659335)
> * [微服务实战](http://kb.cnblogs.com/page/521880/)
> * [Getting-Started-With-Microservices](https://dzone.com/refcardz/getting-started-with-microservices)
> * [Getting started with microservices](https://blog.ruxit.com/microservices/)

根据 Oracle 大神的指导，MicroService 可以认为是 SOA 的一种实现方案：

> Microservices are the kind of SOA we have been talking about for the last decade. Microservices must be independently deployable, whereas SOA services are often implemented in deployment monoliths. Classic SOA is more platform driven, so microservices offer more choices in all dimensions.

**1. 优点**

* 每个服务足够内聚，足够小，代码容易理解、开发效率提高
* 服务之间可以独立部署，微服务架构让持续部署成为可能；
* 每个服务可以各自进行 x 扩展和 z 扩展，而且，每个服务可以根据自己的需要部署到合适的硬件服务器上；
* 容易扩大开发团队，可以针对每个服务(service)组件开发团队；
* 提高容错性(fault isolation)，一个服务的内存泄露并不会让整个系统瘫痪；
* 系统不会被长期限制在某个技术栈上。

**2. 缺点**

《人月神话》中讲到：没有银弹，意思是只靠一把锤子是盖不起摩天大楼的，要根据业务场景选择设计思路和实现工具。我们看下为了换回上面提到的好处，我们付出(trade)了什么？

* 开发人员要处理分布式系统的复杂性；开发人员要设计服务之间的通信机制，对于需要多个后端服务的 user case，要在没有分布式事务的情况下实现代码非常困难；涉及多个服务直接的自动化测试也具备相当的挑战性；
* 服务管理的复杂性，在生产环境中要管理多个不同的服务的实例，这意味着开发团队需要全局统筹(_PS：现在 docker 的出现适合解决这个问题_)
* 应用微服务架构的时机如何把握？对于业务还没有理清楚、业务数据和处理能力还没有开始爆发式增长之前的创业公司，不需要考虑微服务架构模式，这时候最重要的是快速开发、快速部署、快速试错。

### 巨石(monolith)

在 Web 应用程序发展的早期，大部分 web 工程是将所有的功能模块(service side)打包到一起并放在一个 web 容器中运行，很多企业的 Java 应用程序打包为 war 包。其他语言(Ruby，Python 或者 C++)写的程序也有类似的问题。譬如对于一个最简单的电商系统，可能会形成如下的架构：

![](http://mmbiz.qpic.cn/mmbiz/sXiaukvjR0RA4LTYdW8vIiaBFNyKAP0khnnroHVqgaicN0dpkyrDsZoASF716LHE6OSv0KgykUc3T5ia1YhcEJJuVA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

这种将所有功能都部署在一个 web 容器中运行的系统就叫做巨石型应用。巨石型应用有很多好处：IDE 都是为开发单个应用设计的、容易测试——在本地就可以启动完整的系统、容易部署——直接打包为一个完整的包，拷贝到 web 容器的某个目录下即可运行。

但是，上述的好处是有条件的：应用不那么复杂。对于大规模的复杂应用，巨石型应用会显得特别笨重：要修改一个地方就要将整个应用全部部署(_PS：在不同的场景下优势也变成了劣势_)；编译时间过长；回归测试周期过长；开发效率降低等。另外，巨石应用不利于更新技术框架，除非你愿意将系统全部重写(代价太高你愿意老板也不愿意)。

### 应用拆分

![](http://mmbiz.qpic.cn/mmbiz/sXiaukvjR0RA4LTYdW8vIiaBFNyKAP0khnKvib5CIjF3ibc9ytWgB6Z6Ggo9d4D2aarWJQEiauicBBffEtlMI44yzIfw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

这张图从三个维度概括了一个系统的扩展过程：(1)x 轴，水平复制，即在负载均衡服务器后增加多个 web 服务器；(2)z 轴扩展，是对数据库的扩展，即分库分表(分库是将关系紧密的表放在一台数据库服务器上，分表是因为一张表的数据太多，需要将一张表的数据通过 hash 放在不同的数据库服务器上)；(3)y 轴扩展，是功能分解，将不同职能的模块分成不同的服务。从 y 轴这个方向扩展，才能将巨型应用分解为一组不同的服务，例如订单管理中心、客户信息管理中心、商品管理中心等等。

将系统划分为不同的服务有很多方法：(1)按照用例划分，例如在线商店系统中会划分出一个 checkout UI 服务，这个服务实现了 checkout 这个用例；(2)按照资源划分，例如可以划分出一个 catlog 服务来存储产品目录。

服务划分有两个原则要遵循：(1)每个服务应该尽可能符合单一职责原则——Single Responsible Principle，即每个服务只做一件事，并把这件事做好；(2)参考 Unix 命令行工具的设计，Unix 提供了大量的简单易用的工具，例如 grep、cat 和 find。每个工具都小而美。

最后还要强调：系统分解的目标并不仅仅是搞出一堆很小的服务，这不是目标；真正的目标是解决巨石型应用在业务急剧增长时遇到的问题。

对于上面的例子，按照功能和资源划分后，就形成下面图 3 的架构图。分解后的微服务架构包含多个前端服务和后端服务。前端服务包括 Catalog UI(用于商品搜索和浏览)、Checkout UI(用于实现购物车和下单操作)；后端服务包括一些业务逻辑模块，我们将在巨石应用中的每个服务模块重构为一个单独的服务。这么做有什么问题呢？

![](http://mmbiz.qpic.cn/mmbiz/sXiaukvjR0RA4LTYdW8vIiaBFNyKAP0khnWXC4ic9Awg5hWwrs0lzqvX7qibm2riaxQCsmfZ0zreu1jV57Ds7Wwju3A/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

### 微服务之间的通信

**(1)客户端与服务器之间的通信**

在巨石型架构下，客户端应用程序(web 或者 app)通过向服务端发送 HTTP 请求；但是，在微服务架构下，原来的巨石型服务器被一组微服务替代，这种情况下客户端如何发起请求呢？

如图 4 中所示，客户端可以向 micro service 发起 RESTful HTTP 请求，但是会有这种情况发生：客户端为了完成一个业务逻辑，需要发起多个 HTTP 请求，从而造成系统的吞吐率下降，再加上无线网络的延迟高，会严重影响客户端的用户体验。

![](http://mmbiz.qpic.cn/mmbiz/sXiaukvjR0RA4LTYdW8vIiaBFNyKAP0khneb6U1PnJkcwcOC4lKTQcoOcibszIxOHjyDvysLMkWnyDAAeGicGicELfQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

Fig4 - calling services directly

为了解决这个问题，一般会在服务器集群前面再加一个角色：API gateway，由它负责与客户度对接，并将客户端的请求转化成对内部服务的一系列调用。这样做还有个好处是，服务升级不会影响到客户端，只需要修改 API gateway 即可。加了 API gateway 之后的系统架构图如图 5 所示。

![](http://mmbiz.qpic.cn/mmbiz/sXiaukvjR0RA4LTYdW8vIiaBFNyKAP0khnevaeY61h6sciaW9IRF6pPClhkGNiaQCOqRq6zzjSQ07MZVcmA7W1HiakA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

Fig5 - API gateway

**(2)内部服务之间的通信**

内部服务之间的通信方式有两种：基于 HTTP 协议的同步机制(REST、RPC)；基于消息队列的异步消息处理机制(AMQP-based message broker)。

Dubbo 是阿里巴巴开源的分布式服务框架，属于同步调用，当一个系统的服务太多时，需要一个注册中心来处理**服务发现**问题，例如使用 ZooKeeper 这类配置服务器进行服务的地址管理：服务的发布者要向 ZooKeeper 发送请求，将自己的服务地址和函数名称等信息记录在案；服务的调用者要知道服务的相关信息，具体的机器地址在 ZooKeeper 查询得到。这种同步的调用机制足够直观简单，只是没有“订阅——推送”机制。

AMQP-based 的代表系统是 kafka、RabbitMQ 等。这类分布式消息处理系统将订阅者和消费者解耦合，消息的生产者不需要消费者一直在线；消息的生产者只需要把消息发送给消息代理，因此也不需要服务发现机制。

两种通信机制都有各自的优点和缺点，实际中的系统经常包含两种通信机制。例如，在分布式数据管理中，就需要同时用到同步 HTTP 机制和异步消息处理机制。

**2. 分布式数据管理**

**(1)处理读请求**

在线商店的客户账户有限额，当客户试图下单时，系统必须判断总的订单金额是否超过他的信用卡额度。信用卡额度由 CustomerService 管理、下订单的操作由 OrderService 负责，因此 Order Service 要通过 RPC 调用向 Customer Service 请求数据；这种方法能够保证每次 Order Service 都获取到准确的额度，单缺点是多一次 RPC 调用、而且 Customer Service 必须保持在线。

还有一种处理方式是，在 OrderService 这边存放一份信用卡额度的副本，这样就不需要实时发起 RPC 请求，但是还需要一种机制保证——当 Customer Service 拥有的信用卡额度发生变化时，要及时更新存放在 Order Service 这边的副本。

**(2)处理更新请求**

当一份数据位于多个服务上时，必须保证数据的一致性。

* 分布式事务(Distributed transactions)

  使用分布式事务非常直观，即要更新 Customer Service 上的信用卡额度，就必须同时更新其他服务上的副本，这些操作要么全做要么全不做。使用分布式事务能够保证数据的强一致，但是会降低系统的可用性——所有相关的服务必须始终在线；而且，很多现代的技术栈并不支持事务，例如 REST、NoSQL 数据库等。

* 基于事件的异步更新(Event-driven asynchronous updates)

  Customer Service 中的信用卡额度改变时，它对外发布一个事件到“message broker(消息代理人)”；其他订阅了这个事件的服务受到提示后就更新数据。事件流如图 6 所示。

  ![](http://mmbiz.qpic.cn/mmbiz/sXiaukvjR0RA4LTYdW8vIiaBFNyKAP0khnfsbdEH4nNZ4iapSQ1HKsbJKDkLuOtp7ib5rSDRGjz16H9wBpWumjbZwA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

  Fig 6 - replicating the credit limit using events

## REST

# 框架设计模式(Framework Pattern)

# 驱动模式(Driven Pattern)

## TDD

测试驱动开发的基本思想就是在开发功能代码之前，先编写测试代码。也就是说在明确要开发某个功能后，首先思考如何对这个功能进行测试，并完成测试代码的编写，然后编写相关的代码满足这些测试用例。然后循环进行添加其他功能，直到完全部功能的开发。我们这里把这个技术的应用领域从代码编写扩展到整个开发过程。应该对整个开发过程的各个阶段进行测试驱动，首先思考如何对这个阶段进行测试、验证、考核，并编写相关的测试文档，然后开始下一步工作，最后再验证相关的工作。下图是一个比较流行的测试模型：V 测试模型。

## BDD

## DDD

> [领域模型驱动设计][1]

# 敏捷开发(Agile)

# Reference

## 软件架构

* [Architectural Styles and

  the Design of Network-based Software Architectures][2]

[1]: http://blog.csdn.net/johnstrive/article/details/16805121
[2]: http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm
