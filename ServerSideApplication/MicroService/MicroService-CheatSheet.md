[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# MicroService CheatSheet | 微服务理念、架构与实践速览

就像高级编程语言一样，微服务以更抽象能力提供了更好地描述问题的方式。独立部署、独立开发

微服务是一个简单而泛化的概念，不同的行业领域、技术背景、业务架构对于微服务的理解与实践也是不一致的。

微服务架构同时还是一个组织原理的体现，这个原理就是康威定律(Conway’s Law)，设计系统的组织，其产生的设计和架构等价于组织间的沟通结构。这些系统在建成之后反过来还会约束和限制组织的改变。

无论是微服务，还是 SOA，体现相通的架构原理：单一职责，有界上下文，关注分离，分而治之。区别在于微服务粒度更细，同时融入了近几年一线互联网公司的一些最佳实践，是服务化的新提法。微服务原理和软件工程，面向对象设计中的基本原理相通，体现如下：

单一职责(Single Responsibility)，一个服务应当承担尽可能单一的职责，服务应基于有界的上下文(bounded context，通常是边界清晰的业务领域)构建，服务理想应当只有一个变更的理由(类似 Robert C. Martin 讲的：A class should have only one reason to change)，当一个服务承担过多职责，就会产生各种耦合性问题，需要进一步拆分使其尽可能职责单一化。

关注分离(Separation of Concerns)，跨横切面逻辑，例如日志分析、监控、限流、安全等等，尽可能与具体的业务逻辑相互分离，让开发人员能专注于业务逻辑的开发，减轻他们的思考负担，这个也是有界上下文(bounded context)的一个体现。

模块化(Modularity)和分而治之(Divide & Conquer)，这个是解决复杂性问题的一般性方法，将大问题(如单块架构)大而化小(模块化和微服务化)，然后分而治之。

随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需一个治理系统确保架构有条不紊的演进。 ![](http://dubbo.io/dubbo-architecture-roadmap.jpg-version=1&modificationDate=1331143666000.jpg)

- 单一应用架构
  - 当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。
  - 此时，用于简化增删改查工作量的 数据访问框架 (ORM) 是关键。
- 垂直应用架构
  - 当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。
  - 此时，用于加速前端页面开发的 Web 框架 (MVC) 是关键。
- 分布式服务架构
  - 当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。
  - 此时，用于提高业务复用及整合的 分布式服务框架 (RPC) 是关键。
- 流动计算架构 - 当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。 - 此时，用于提高机器利用率的 资源调度和治理中心 (SOA) 是关键。

# Monolithic 与 MicroService

微服务应用往往由多个粒度较小，版本独立，有明确边界并可扩展的服务构成，各个服务之间通过定义好的标准协议相互通信。

## 存储分隔与无状态服务

在编码中，我们往往倾向于使用纯函数来描述抽象逻辑，以保证代码的可读性与可测性；并且纯函数也可以非常方便地使用并发编程或者结果缓存等方式来进行扩展与

The monolithic approach on the left has a single database and tiers of specific technologies.

The microservices approach on the right has a graph of interconnected microservices where state is typically scoped to the microservice and various technologies are used.

In a monolithic approach, typically the application uses a single database. The advantage is that it is a single location, which makes it easy to deploy. Each component can have a single table to store its state. Teams need to strictly separate state, which is a challenge. Inevitably there are temptations to add a new column to an existing customer table, do a join between tables, and create dependencies at the storage layer. After this happens, you can't scale individual components.

In the microservices approach, each service manages and stores its own state. Each service is responsible for scaling both code and state together to meet the demands of the service. A downside is that when there is a need to create views, or queries, of your application’s data, you need to query across disparate state stores. Typically, this is solved by having a separate microservice that builds a view across a collection of microservices. If you need to perform multiple impromptu queries on the data, each microservice should consider writing its data to a data warehousing service for offline analytics.

![](https://docs.microsoft.com/en-us/azure/service-fabric/media/service-fabric-overview-microservices/statemonolithic-vs-micro.png)

# 微服务技术栈

## 服务网关

网关一词较早出现在网络设备里面，比如两个相互独立的局域网段之间通过路由器或者桥接设备进行通信， 这中间的路由或者桥接设备我们称之为网关。

相应的 API 网关将各系统对外暴露的服务聚合起来，所有要调用这些服务的系统都需要通过 API 网关进行访问，基于这种方式网关可以对 API 进行统一管控，例如：认证、鉴权、流量控制、协议转换、监控等等。API 网关的流行得益于近几年微服务架构的兴起，原本一个庞大的业务系统被拆分成许多粒度更小的系统进行独立部署和维护，这种模式势必会带来更多的跨系统交互，企业 API 的规模也会成倍增加，API 网关(或者微服务网关)就逐渐成为了微服务架构的标配组件。

1、面向 Web 或者移动 App

这类场景，在物理形态上类似前后端分离，前端应用通过 API 调用后端服务，需要网关具有认证、鉴权、缓存、服务编排、监控告警等功能。

2、面向合作伙伴开放 API

这类场景，主要为了满足业务形态对外开放，与企业外部合作伙伴建立生态圈，此时的 API 网关注重安全认证、权限分级、流量管控、缓存等功能的建设。

3、企业内部系统互联互通

对于中大型的企业内部往往有几十、甚至上百个系统，尤其是微服务架构的兴起系统数量更是急剧增加。系统之间相互依赖，逐渐形成网状调用关系不便于管理和维护，需要 API 网关进行统一的认证、鉴权、流量管控、超时熔断、监控告警管理，从而提高系统的稳定性、降低重复建设、运维管理等成本。

服务发现是大部分分布式系统和面向服务架构的核心组件。最初问题看起来很简单：_客户如何决定服务的 IP 地址和端口，这些服务已存在于多个服务器上的。_

通常，你开始一些静态的配置，这些配置离你需要做的还挺远的。当你开始布署越来越多的服务时，事情会越来越复杂。在一个上线的系统中，由于自动的或人为的规模变化，服务的位置会经常的变化，例如布署新的服务，服务器宕机或者被替换。

在这些应用场景中为了避免服务冲突，动态的服务注册和发现会越来越重要。

定位服务的问题划分为两类。服务注册与服务发现。

- **服务注册** - 服务进程在注册中心注册自己的位置。它通常注册自己的主机和端口号，有时还有身份验证信息，协议，版本号，以及运行环境的详细资料。
- **服务发现** - 客户端应用进程向注册中心发起查询，来获取服务的位置。

任何服务注册、服务发现也有其它开发、操作层面的考虑：

- **监控** - 如果服务注册失败会发生什么？有时会因为超时、或者其它进程而突然处于未注册状态。通常会要求服务实现心跳机制来确保其活跃性，并且通常要求客户端有能力可靠地处理失效的服务。
- **负载均衡** -如果多个服务被注册，怎样来处理所有的客户端跨服务的均衡问题？如果有个主服务，它能被客户端正确的判断吗？
- **集成风格** - 注册中心仅仅提供了少量语言的绑定，例如仅仅支持 Java 吗？集成需要嵌入注册与发现的代码到程应用程序中，还是可以选择一个辅助进程？
- **运行时依赖** - 它需要 JVM， Ruby 或者其它与你的运行环境不兼容的软件吗？
- **可用性考虑** - 丢失一个节点能继续工作吗？升级时不会中断服务吗？注册处会成为架构的中心部分，会变成单点故障吗？

## Service Mesh

# 微服务的红与黑
