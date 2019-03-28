[![返回目录](https://parg.co/UCb)](https://github.com/wx-chevalier/Awesome-CheatSheets)

# RPC CheatSheet

API 是应用程序编程接口（Application Programming Interface）的缩写。维基百科指出，“总的来说，它是各种组件之间的一组明确定义的通信方法”。它可以是软件框架或库的接口，也可以是操作系统为原生系统软件（如 POSIX）开发人员公开的底层接口。现如今，当人们谈论 API 时，他们通常指的是通过 HTTP 端点公开的远程接口。为了区分这些远程 API 和上面提到的本地系统 API，我将用术语“Web API”指代远程 API。

严格来说，API 仅用来描述接口，也就是客户端和服务器、API 消费者和 API 提供者之间用于交换信息的语言。对于 API 消费者来说，API 只不过是对接口和端点 URL 或 URL 集的描述。URL 是 Web 的基本构建块之一，客户端可以在不知道服务器性质或位置的情况下访问信息或服务。

我们通过底层设计范式（如查询、RPC 或 RESTful）或协议（如 SOAP、gRPC 或 GraphQL）进一步对远程 API（或 Web API）进行分类。除此之外，我们还通过目标受众来区分 API，将它们分为公共、合作伙伴或私有 / 内部 API；延伸阅读 [Architecture Style CheatSheet](https://parg.co/6NU)。

互联网上的机器大都通过 TCP/IP 协议相互访问，但 TCP/IP 只是往远端发送了一段二进制数据，为了建立服务还有很多问题需要抽象：

数据以什么格式传输？不同机器间，网络间可能是不同的字节序，直接传输内存数据显然是不合适的；随着业务变化，数据字段往往要增加或删减，怎么兼容前后不同版本的格式？
一个 TCP 连接可以被多个请求复用以减少开销么？多个请求可以同时发往一个 TCP 连接么?
如何管理和访问很多机器？
连接断开时应该干什么？
万一 server 不发送回复怎么办？

RPC 可以解决这些问题，它把网络交互类比为“client 访问 server 上的函数”：client 向 server 发送 request 后开始等待，直到 server 收到、处理、回复 client 后，client 又再度恢复并根据 response 做出反应

我们来看看上面的一些问题是如何解决的：

数据需要序列化，protobuf 在这方面做的不错。用户填写 protobuf::Message 类型的 request，RPC 结束后，从同为 protobuf::Message 类型的 response 中取出结果。protobuf 有较好的前后兼容性，方便业务调整字段。http 广泛使用 json 作为序列化方法。
用户无需关心连接如何建立，但可以选择不同的连接方式：短连接，连接池，单连接。
大量机器一般通过命名服务被发现，可基于 DNS, ZooKeeper, etcd 等实现。在百度内，我们使用 BNS (Baidu Naming Service)。brpc 也提供"list://"和"file://"。用户可以指定负载均衡算法，让 RPC 每次选出一台机器发送请求，包括: round-robin, randomized, consistent-hashing(murmurhash3 or md5)和 locality-aware.
连接断开时可以重试。
如果 server 没有在给定时间内回复，client 会返回超时错误。

既然是基于消息，那么跑不掉两种模式： Point-to-Point， Message Broker。 P2P 的模式大家都了解的，就是启动箭头端口，然后等其他应用连接上来，load balance 等都是由 SDK 或者 sidecar 来搞定。Message Broker 就不一样啦，你只需要将消息发送到 broker，然后就不用管啦，这些路由等，都是由 message broker 搞定，完全透明。 这篇文章我们主要讲基于 Message Broker 来做 RPC 通讯，也就是 RSocket Broker 负责完成所有的路由等功能。

基于消息的 RPC 通讯，这个消息的格式还有一点点不同，之前文章介绍过，我们这里再细化一下。 RPC Message 主要有三个部分：

消息元信息(message metadata): 元信息主要提供一些基础信息，对于 RPC 消息来说，最主要的就是路由元信息(Routing metadata)和消息数据的格式(data encoding metadata)。 路由是标明你要调用哪个服务，broker 来帮你发送到指定的服务提供方。 消息数据的格式，主要就是对象的序列化和反序列化，服务方可以获得真实的对象。
消息数据： 要发送的数据，对于 RPC 来说，就是参数和返回结果。 你调用一个服务时，需要提供调用的参数，服务逻辑执行完毕后，将结果再返回回来。 数据通常有一定的编码格式，这个由 data encoding metadata 来控制，RSocket Broker 目前主要支持三种: json, hessian 和 protobuf。
消息 ID： 消息是异步发送出去的，服务方逻辑执行完毕后，也是消息发送回来。RPC 是要求知道结果的，所以返回的消息和发出去的消息是要匹配的，这样才能知道调用的结果。 在 RSocket 中，这个 ID 你不需要自己设置，这个是 SDK 自动帮你匹配的，完全透明，我们只需要设置元信息和数据就可以。

RPC(Remote Procedure Call)，远程过程调用，从最简单最抽象的模式来看，就是下面这个图这样。客户端调用某个方法，然后中间经过一系列的过程，调用到服务端的某个方法。服务端进行处理之后，做出相应，然后逐层原路返回到客户端。

![640](https://user-images.githubusercontent.com/5803001/39872709-72837628-549b-11e8-83a7-2dde4ac41db9.png)

一般来说，开发者只需要关注蓝色( functions )部分，而至于红色部分( stub 句柄 ) 和黄色部分(socket 网络)部分呢，框架层面会把它解决掉。蓝色部分，服务端开发者要做的事情就是定义某个接口，客户端开发者要做的事情就是调用某个接口，一切开发模式都跟本地调用无差别。
