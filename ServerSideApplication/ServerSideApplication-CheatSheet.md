[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Server Side Application CheatSheet | 服务端编程基础

# 层级划分与实体类

![image](https://user-images.githubusercontent.com/5803001/44942628-05bc1e80-ade9-11e8-9aea-25c51404638a.png)

| 对象名                     | 层名       | 描述 |
| -------------------------- | ---------- | ---- |
| Transfer Object/TO         | Controller |      |
| Business Object/BO         | Service    |      |
| Database Access Object/DAO | Model      |      |

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

# 中间件

介于操作系统和应用程序之间的产品，中间件简单解释，你可以理解为面向信息系统交互，集成过程中的通用部分的集合，屏蔽了底层的通讯，交互，连接等复杂又通用化的功能，以产品的形式提供出来，系统在交互时，直接采用中间件进行连接和交互即可，避免了大量的代码开发和人工成本。其实，理论上来讲，中间件所提供的功能通过代码编写都可以实现，只不过开发的周期和需要考虑的问题太多，逐渐的，这些部分，以中间件产品的形式进行了替代。比如常见的消息中间件，即系统之间的通讯与交互的专用通道，类似于邮局，系统只需要把传输的消息交给中间件，由中间件负责传递，并保证传输过程中的各类问题，如网络问题，协议问题，两端的开发接口问题等均由消息中间件屏蔽了，出现了网络故障时，消息中间件会负责缓存消息，以避免信息丢失。中间件就是 非业务的技术类组件。

其实从广义来说 操作系统上，业务系统下与业务无关的 ，都是中间件，包括数据库，离线等。当然 实际上不会这么分。 不过利用这个讲法应该能够更容易的去理解中间件是什么。

阿里的中间件主要就包含这么几个: [分布式关系型数据库 DRDS\_ 水平拆分 \*\*](https://www.aliyun.com/product/drds) 做数据库扩展性的 [消息队列 \_ 云消息 \*\*](https://www.aliyun.com/product/ons/) 做消息的 MOM [企业级分布式应用服务 EDAS\_ 企业云计算解决方案 \*\*](https://www.aliyun.com/product/edas) 做分布式服务的还有一些其他的中间件，比如 JstormT , 配置服务 缓存 等等，也都会放在中间件里

# Todos

- [ ] [pragmatic-java-engineer](https://github.com/superhj1987/pragmatic-java-engineer/blob/master/book/chapter1-servertech/server-basic.md)
