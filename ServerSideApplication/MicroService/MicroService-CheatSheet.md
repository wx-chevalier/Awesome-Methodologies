# 微服务理念、架构与实践速览

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

服务发现是大部分分布式系统和面向服务架构的核心组件。最初问题看起来很简单：_客户如何决定服务的 IP 地址和端口，这些服务已存在于多个服务器上的。_

通常，你开始一些静态的配置，这些配置离你需要做的还挺远的。当你开始布署越来越多的服务时，事情会越来越复杂。在一个上线的系统中，由于自动的或人为的规模变化，服务的位置会经常的变化，例如布署新的服务，服务器宕机或者被替换。

在这些应用场景中为了避免服务冲突，动态的服务注册和发现会越来越重要。

定位服务的问题划分为两类。服务注册与服务发现。

* **服务注册** - 服务进程在注册中心注册自己的位置。它通常注册自己的主机和端口号，有时还有身份验证信息，协议，版本号，以及运行环境的详细资料。
* **服务发现** - 客户端应用进程向注册中心发起查询，来获取服务的位置。

任何服务注册、服务发现也有其它开发、操作层面的考虑：

* **监控** - 如果服务注册失败会发生什么？有时会因为超时、或者其它进程而突然处于未注册状态。通常会要求服务实现心跳机制来确保其活跃性，并且通常要求客户端有能力可靠地处理失效的服务。
* **负载均衡** -如果多个服务被注册，怎样来处理所有的客户端跨服务的均衡问题？如果有个主服务，它能被客户端正确的判断吗？
* **集成风格** - 注册中心仅仅提供了少量语言的绑定，例如仅仅支持 Java 吗？集成需要嵌入注册与发现的代码到程应用程序中，还是可以选择一个辅助进程？
* **运行时依赖** - 它需要 JVM， Ruby 或者其它与你的运行环境不兼容的软件吗？
* **可用性考虑** - 丢失一个节点能继续工作吗？升级时不会中断服务吗？注册处会成为架构的中心部分，会变成单点故障吗？

## Service Mesh

# 微服务的红与黑
