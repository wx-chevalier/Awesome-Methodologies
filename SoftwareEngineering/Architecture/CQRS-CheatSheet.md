[![返回目录](https://parg.co/UCb)](https://github.com/wx-chevalier/Awesome-CheatSheets)

# CQRS

CQRS（Command & Query Responsibility Segregation）命令查询职责分离，和 REST 同属于架构风格，如果单纯理解 CQRS，是比较容易的，另一种方式解释就是，一个方法要么是执行某种动作的命令，要么是返回数据的查询，命令的体现是对系统状态的修改，而查询则不会，职责的分离更加有利于领域模型的提炼，系统的灵活性和可扩展性也得到进一步加强。

![](https://images0.cnblogs.com/blog2015/435188/201504/211614435006687.png)

来自用户 UI 的请求分为 Query（查询）和 Command（命令），这些请求操作都会被 Service Interfaces（服务接口，只是一个统称）接收，然后再进行分发处理，对于命令操作会更新 Update Data store，因为读与写分离，为了保持数据的一致性，我们还需要把数据更新应用到 Read Data store。对于一般的应用系统来说，查询会占很大的比重，因为读与写分离了，所以我们可以针对查询进行进一步性能优化，而且还可以保持查询的灵活性和独立性，这种方式在应对大型业务系统来说是非常重要的，从这种层面上来说，CQRS 不用于 DDD 架构好像也是可以的，因为它是一种风格，并不局限于一种架构实现，所以你可以把它有价值的东西进行提炼，应用到合适的一个架构系统中也是可以的。

Command Bus（命令总线）：图中没有，应该放在 Command Handler 之前，可以看作是 Command 发布者。
Command Handler（命令处理器）：处理来自 Command Bus 分发的请求，可以看作是 Command 订阅者、处理者。
Event Bus（事件总线）：一般在 Command Handler 完成之后，可以看作是 Event 发布者。
Event Handler（事件处理器）：处理来自 Event Bus 分发的请求，可以看作是 Event 订阅者、处理者。
Event Store（事件存储）：对应概念 Event Sourcing（事件溯源），可以用于事件回放处理，还原指定对象状态。

首先抽离两个重要概念：Command（命令）和 Event（事件），Command 是一种命令的语气（本身就是命令的意思，呵呵），它的效果就是对某种对象状态的修改，Command Bus 收集来自 UI 的 Command 命令，并根据具体命令分发给具体的 Command Handler 进行处理，这时候就会产生一些领域操作，并对相应的领域对象进行修改，Command Handler 只是修改操作，并不会涉及到修改之后的操作（比如保存、事件发布等），Command Handler 完成之后并不表示这个 Command 命令就此结束，它需要把接下来的操作交给 Event Bus（完成之后的操作），并分发给相应的 Event Handler 订阅者进行处理，一般是数据保存、事件存储等。

VO（View Object）：视图对象，用于展示层，它的作用是把某个指定页面（或组件）的所有数据封装起来。

DTO（Data Transfer Object）：数据传输对象，这个概念来源于 J2EE 的设计模式，原来的目的是为了 EJB 的分布式应用提供粗粒度的数据实体，以减少分布式调用的次数，从而提高分布式调用的性能和降低网络负载，但在这里，我泛指用于展示层与服务层之间的数据传输对象。

DO（Domain Object）：领域对象，就是从现实世界中抽象出来的有形或无形的业务实体。

PO（Persistent Object）：持久化对象，它跟持久层（通常是关系型数据库）的数据结构形成一一对应的映射关系，如果持久层是关系型数据库，那么，数据表中的每个字段（或若干个）就对应 PO 的一个（或若干个）属性。

梳理 Command 整个流程，你会发现一个关键词：状态（Status），在上一篇博文讲 REST 概念时，也有一个相似的概念：应用状态（Application State），REST 其中的一个含义就是状态转换，从客气端的发起请求开始，到服务端响应请求结束，应用状态在其过程中会进行不断的转换，请求响应的整个过程也就是应用状态转换的过程，对于 Command 处理流程来说，领域对象的状态和应用状态其实是相类似。我举一个例子，在 REST 架构风格中，应用状态是不会保存到服务端的，客户端发起请求（包含应用状态信息），服务端做出相应处理，此时的状态会转换成资源状态呈现给客户端，这就是表现层状态转换的意思，回到 Command 处理流程上，Command Bus 接收来自 UI 的请求，分发给相应的 Command Handler 进行处理，在处理过程中，就会对领域对象进行修改操作，但它不会保存修改之后的状态信息，而是交给 Event Handler 进行保存状态信息。

和 Command 相比，Query 的处理流程就简单很多了，Query Service 接收来自 UI 的查询请求，这个查询处理可以用各种方式实现，你可以使用 ORM，也可以直接写 SQL 代码
