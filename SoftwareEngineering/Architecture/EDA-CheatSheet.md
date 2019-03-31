[![返回目录](https://i.postimg.cc/JzFTMvjF/image.png)](https://github.com/wx-chevalier/Awesome-CheatSheets)

# Event Driven Architecture Style CheatSheet

The difference being that messages are directed, events are not — a message has a clear addressable recipient while an event just happen for others (0-N) to observe it.

A message is an item of data that is sent to a specific destination. An event is a signal emitted by a component upon reaching a given state. In a message-driven system addressable recipients await the arrival of messages and react to them, otherwise lying dormant. In an event-driven system notification listeners are attached to the sources of events such that they are invoked when the event is emitted. This means that an event-driven system focuses on addressable event sources while a message-driven system concentrates on addressable recipients. A message can contain an encoded event as its payload.

事件代表过去发生的事件，事件既是技术架构概念，也是业务概念，以事件为驱动的编程模型称为事件驱动架构 EDA。事件代表过去发生的事件，事件既是技术架构概念，也是业务概念，以事件为驱动的编程模型称为事件驱动架构 EDA。

EDA 架构的三个特性：

异步
实时
彻底解耦
EDA 架构的核心是基于消息的发布订阅模式，通过发布订阅模式实现事件的一对多灵活分发。消息消费方对发送方而言完全透明，消息发送方只管把消息发送到消息中间件，其它事情全部不用关心，由于消息中间件中的 MQ 等技术，即使发送消息时候，消息接收方不可用，但仍然可以正常发送，这才叫彻底解耦。其次一对多的发布订阅模式也是一个核心重点，对于消息的订阅方和订阅机制，可以在消息中间件灵活的进行配置和管理，而对于消息发送方和发送逻辑基本没有任何影响。

EDA 要求我们的是通过业务流程，首先要识别出有价值的业务事件，这些事件符合异步、实时和发布订阅等基本的事件特征；其次是对事件进行详细的分析和定义，对事件对应的消息格式进行定义，对事件的发布订阅机制进行定义等，最后才是基于消息事件模式的开发和测试等工作。

Long-Running Process（Saga）－长时处理过程 说明它是一个需要耗时、多任务并行的处理过程，那在领域中，什么时候会有它的用武之地呢？比如一个看似简单的业务逻辑，可能会涉及到领域中很复杂的业务操作，而且对于这些处理需要耗费很长的时间。

https://images0.cnblogs.com/blog2015/435188/201504/221630054068285.png

上图描述的是：一个会议购买座位的业务过程，中间的 Order Process Manager 就是一个 Saga，在 CQRS 架构中的表现就是 Process Manager（过程管理），我们一般会用它处理多个聚合根交互的业务逻辑

Event Sourcing－事件溯源。而对于领域对象来说，我们也应该知晓其整个生命周期的演变过程，这样有利于查看并还原某一“时刻”的领域对象，在 EDA 架构中，对于领域对象状态的保存是通过领域事件进行完成，所以我们要想记录领域对象的状态记录，就需要把领域对象所经历的所有事件进行保存下来，这就是 Event Store（事件存储），这个东西就相当于 Git 代码服务器，存储所有领域对象所经历的事件，对于某一事件来说，可以看作是对应领域对象的“快照”。

提取最后的参考资料

http://www.cnblogs.com/xishuai/p/iddd-cqrs-and-eda.html#xishuai_h1
