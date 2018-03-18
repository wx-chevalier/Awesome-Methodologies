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

## Service Mesh

# 微服务的红与黑
