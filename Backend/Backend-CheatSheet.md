[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Server Side Application CheatSheet | 服务端编程基础

# 层级划分与实体类

![image](https://user-images.githubusercontent.com/5803001/44942628-05bc1e80-ade9-11e8-9aea-25c51404638a.png)

| 对象名                     | 层名              | 描述                                                |
| -------------------------- | ----------------- | --------------------------------------------------- |
| Transfer Object/TO         | Controller        | 接入与返回层，提供视图数据聚合与统一的查询/返回格式 |
| Business Object/BO         | Service/Connector | 数据业务层，提供业务数据的聚合                      |
| Database Access Object/DAO | Model             | 元数据访问层，与数据库进行直接交互                  |

在设计数据库的时候，我们尽量避免给属性列添加额外的前缀，并且使用嵌套的结构返回多表联查的数据：

```json
{
  "user": {
    "uuid": "{uuid}",
    "name": "{name}"
  },
  "asset": {
    "uuid": "{uuid}",
    "name": "{name}"
  },
  "lessonss": []
}
```

```sh
/api/resource/get
/api/resource/getByIds

# 在交互层级上同样应该有所隐藏
/api/resource/getRelatedResourceById
/api/related-resource/getRelatedResourceByResourceId
```

```gql
query {
  Resources{
    id
  }
}

query {
  Resource($id: 1){
    id
    statisticsField(){}
    oneToOneField() {}
    oneToManyField(){}
  }
}
```

# 中间件

介于操作系统和应用程序之间的产品，中间件简单解释，你可以理解为面向信息系统交互，集成过程中的通用部分的集合，屏蔽了底层的通讯，交互，连接等复杂又通用化的功能，以产品的形式提供出来，系统在交互时，直接采用中间件进行连接和交互即可，避免了大量的代码开发和人工成本。其实，理论上来讲，中间件所提供的功能通过代码编写都可以实现，只不过开发的周期和需要考虑的问题太多，逐渐的，这些部分，以中间件产品的形式进行了替代。比如常见的消息中间件，即系统之间的通讯与交互的专用通道，类似于邮局，系统只需要把传输的消息交给中间件，由中间件负责传递，并保证传输过程中的各类问题，如网络问题，协议问题，两端的开发接口问题等均由消息中间件屏蔽了，出现了网络故障时，消息中间件会负责缓存消息，以避免信息丢失。中间件就是 非业务的技术类组件。

其实从广义来说 操作系统上，业务系统下与业务无关的 ，都是中间件，包括数据库，离线等。当然 实际上不会这么分。 不过利用这个讲法应该能够更容易的去理解中间件是什么。

阿里的中间件主要就包含这么几个: [分布式关系型数据库 DRDS\_ 水平拆分 \*\*](https://www.aliyun.com/product/drds) 做数据库扩展性的 [消息队列 \_ 云消息 \*\*](https://www.aliyun.com/product/ons/) 做消息的 MOM [企业级分布式应用服务 EDAS\_ 企业云计算解决方案 \*\*](https://www.aliyun.com/product/edas) 做分布式服务的还有一些其他的中间件，比如 JstormT , 配置服务 缓存 等等，也都会放在中间件里

# API

# ApiRPC CheatSheet

RPC(Remote Procedure Call)，远程过程调用，从最简单最抽象的模式来看，就是下面这个图这样。客户端调用某个方法，然后中间经过一系列的过程，调用到服务端的某个方法。服务端进行处理之后，做出相应，然后逐层原路返回到客户端。

![640](https://user-images.githubusercontent.com/5803001/39872709-72837628-549b-11e8-83a7-2dde4ac41db9.png)

一般来说，开发者只需要关注蓝色( functions )部分，而至于红色部分( stub 句柄 ) 和黄色部分(socket 网络)部分呢，框架层面会把它解决掉。蓝色部分，服务端开发者要做的事情就是定义某个接口，客户端开发者要做的事情就是调用某个接口，一切开发模式都跟本地调用无差别。

# RPC

# WebAPI

# BFF: 用户体验适配层

随着无线技术的发展和各种智能设备的兴起，互联网应用已经从单一 Web 浏览器时代演进到以 API 驱动的无线优先(Mobile First)和面向全渠道体验(omni-channel experience oriented)的时代。

应用架构的新挑战是：

用户接入形式的多样性，无线(手机、Pad)，Web，互联网电视，第三方合作应用等等，各种用户设备的屏幕大小，操控体验方式各不相同，例如，手机设备的屏幕较小，能够展示的数据量小，交互方式以触控为主，也可通过条形码扫描器；

有些用户设备的带宽受限，同时应用的 UI 一般宿主在客户端，有些页面需要组合好几个后台业务服务的数据和功能，如果直接在客户端发起对多个后台服务的调用，势必造成大量网络开销影响性能，这个有点类似数据库查询中的 n+1 问题。

BFF(Backend For Frontend)是应对上述应用架构挑战的一种模式和最佳实践，2015 年底，ThoughtWorks 在其网站上刊登了一篇称为 BFF@SoundCloud(SoundCloud 是一个类似音频 YouTube 的网站)的文章[附录 1]，讲述 SoundCloud 如何利用 BFF 模式逐步将其单块 Rails 应用迁移改造为支持无线等多种用户体验的微服务架构。同期，ThoughtWorks 的顾问 Sam Newman，也就是《Building Microservices》那本书的作者，在 SoundCloud 等业界实践的基础上，写了一篇博客总结了 BFF 模式的原理、场景和用法[附录 2]，建议大家阅读。

![640](https://user-images.githubusercontent.com/5803001/39958394-d90d8f42-5634-11e8-9bd7-61925f210d36.png)

BFF 本质上是一个后端中间层，但是它的作用主要是适配前端不同用户体验(无线，桌面，智能终端等等)，所以称为用户体验适配层，它的适配作用主要是：

裁剪和格式化，对后台的通用数据模型进行适当的裁剪和格式化，以适应不同的用户体验展示的需要；

聚合编排，对后台服务数据进行编排和预聚合，这样可以有效简化客户端逻辑和减少网络调用开销。

Sam 推荐理想情况下针对每种用户体验类型需要一个 BFF(one BFF per user experience)，例如 Mobile BFF，Desktop BFF，这可以做到职责单一和关注分离(遵循有界上下文原则)，但是 BFF 过多也会造成代码逻辑重复冗余的问题，需要权衡。UI 和 BFF 理想是同一个团队负责，这样可以减少沟通协调成本，加快变更迭代速度，这是遵循康威定律的体现。下图展示了一种 BFF 和团队职责边界划分方案。

![default](https://user-images.githubusercontent.com/5803001/39958388-bfbe68ae-5634-11e8-97e4-fa4186183522.png)

Sam 还指出，对于一些跨横切面的关注点(cross cutting concerns)，例如路由，安全认证，日志监控分析，限流等等，通常可由独立的网关(Gateway)层负责(如 Fig 6 所示)，由独立基础设施团队运维，置于 BFF 之前，这样在架构上可以做到职责单一和关注分离，让 BFF 开发人员专注于聚合裁剪等业务功能，无需考虑跨横切面功能。但是如果对运维成本和调用性能有额外考虑，跨横切面的功能也可以直接做在 BFF 一层。一般来说，开发者只需要关注蓝色（ functions ）部分，而至于红色部分（ stub 句柄 ） 和黄色部分（socket 网络）部分呢，框架层面会把它解决掉。蓝色部分，服务端开发者要做的事情就是定义某个接口，客户端开发者要做的事情就是调用某个接口，一切开发模式都跟本地调用无差别。

## 查询

### 分页

# RPC

# Todos

- [ ] [pragmatic-java-engineer](https://github.com/superhj1987/pragmatic-java-engineer/blob/master/book/chapter1-servertech/server-basic.md)
