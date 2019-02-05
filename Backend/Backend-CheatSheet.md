[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheets)

# Server Side Application CheatSheet | 服务端编程基础

# 中间件

介于操作系统和应用程序之间的产品，中间件简单解释，你可以理解为面向信息系统交互，集成过程中的通用部分的集合，屏蔽了底层的通讯，交互，连接等复杂又通用化的功能，以产品的形式提供出来，系统在交互时，直接采用中间件进行连接和交互即可，避免了大量的代码开发和人工成本。其实，理论上来讲，中间件所提供的功能通过代码编写都可以实现，只不过开发的周期和需要考虑的问题太多，逐渐的，这些部分，以中间件产品的形式进行了替代。比如常见的消息中间件，即系统之间的通讯与交互的专用通道，类似于邮局，系统只需要把传输的消息交给中间件，由中间件负责传递，并保证传输过程中的各类问题，如网络问题，协议问题，两端的开发接口问题等均由消息中间件屏蔽了，出现了网络故障时，消息中间件会负责缓存消息，以避免信息丢失。中间件就是 非业务的技术类组件。

其实从广义来说 操作系统上，业务系统下与业务无关的 ，都是中间件，包括数据库，离线等。当然 实际上不会这么分。 不过利用这个讲法应该能够更容易的去理解中间件是什么。

阿里的中间件主要就包含这么几个: [分布式关系型数据库 DRDS\_ 水平拆分 \*\*](https://www.aliyun.com/product/drds) 做数据库扩展性的 [消息队列 \_ 云消息 \*\*](https://www.aliyun.com/product/ons/) 做消息的 MOM [企业级分布式应用服务 EDAS\_ 企业云计算解决方案 \*\*](https://www.aliyun.com/product/edas) 做分布式服务的还有一些其他的中间件，比如 JstormT , 配置服务 缓存 等等，也都会放在中间件里

# 架构风格

完整的软件体系结构由不同的模式或者风格构成，常见的风格有架构设计模式、框架设计模式与驱动模式等。

Architectural styles tell us, in very broad strokes, how to organise our code. It’s the highest level of granularity and it specifies layers, high-level modules of the application and how those modules and layers interact with each other, the relations between them. Examples of Architectural Styles:

Component-based
Monolithic application
Layered
Pipes and filters
Event-driven
Publish-subscribe
Plug-ins
Client-server
Service-oriented
An Architectural Style can be implemented in various ways, with a specific technical environment, specific policies, frameworks or practices.

A pattern is a recurring solution to a recurring problem. In the case of Architectural Patterns, they solve the problems related to the Architectural Style. For example, “what classes will we have and how will they interact, in order to implement a system with a specific set of layers“, or “what high-level modules will have in our Service-Oriented Architecture and how will they communicate“, or “how many tiers will our Client-server Architecture have“.

Architectural Patterns have an extensive impact on the code base, most often impacting the whole application either horizontally (ie. how to structure the code inside a layer) or vertically (ie. how a request is processed from the outer layers into the inner layers and back). Examples of Architectural Patterns:

Three-tier
Microkernel
Model-View-Controller
Model-View-ViewModel

# Todos

- [ ] [pragmatic-java-engineer](https://github.com/superhj1987/pragmatic-java-engineer/blob/master/book/chapter1-servertech/server-basic.md)
