# RPC CheatSheet

API 是应用程序编程接口（Application Programming Interface）的缩写。维基百科指出，“总的来说，它是各种组件之间的一组明确定义的通信方法”。它可以是软件框架或库的接口，也可以是操作系统为原生系统软件（如 POSIX）开发人员公开的底层接口。现如今，当人们谈论 API 时，他们通常指的是通过 HTTP 端点公开的远程接口。为了区分这些远程 API 和上面提到的本地系统 API，我将用术语“Web API”指代远程 API。

严格来说，API 仅用来描述接口，也就是客户端和服务器、API 消费者和 API 提供者之间用于交换信息的语言。对于 API 消费者来说，API 只不过是对接口和端点 URL 或 URL 集的描述。URL 是 Web 的基本构建块之一，客户端可以在不知道服务器性质或位置的情况下访问信息或服务。

我们通过底层设计范式（如查询、RPC 或 RESTful）或协议（如 SOAP、gRPC 或 GraphQL）进一步对远程 API（或 Web API）进行分类。除此之外，我们还通过目标受众来区分 API，将它们分为公共、合作伙伴或私有 / 内部 API；延伸阅读 [Architecture Style CheatSheet](https://parg.co/6NU)。
