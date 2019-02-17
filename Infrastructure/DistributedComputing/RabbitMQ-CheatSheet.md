[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheets)

# RabbitMQ CheatSheet

RabbitMQ 是一个由 Erlang 开发的 AMQP(Advanved Message Queue)的开源实现，是经典的消息代理(Message Broker)/消息队列(Message Queue)；AMQP 高级消息队列协议是应用层协议的一个开放标准，为面向消息的中间件设计，基于此协议的客户端与消息中间件可传递消息，并不受产品、开发语言等条件的限制。

RabbitMQ 最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。具体特点包括：

可靠性（Reliability） RabbitMQ 使用一些机制来保证可靠性，如持久化、传输确认、发布确认。

灵活的路由（Flexible Routing） 在消息进入队列之前，通过 Exchange 来路由消息的。对于典型的路由功能，RabbitMQ 已经提供了一些内置的 Exchange 来实现。针对更复杂的路由功能，可以将多个 Exchange 绑定在一起，也通过插件机制实现自己的 Exchange 。

消息集群（Clustering） 多个 RabbitMQ 服务器可以组成一个集群，形成一个逻辑 Broker 。

高可用（Highly Available Queues） 队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。

多种协议（Multi-protocol） RabbitMQ 支持多种消息队列协议，比如 STOMP、MQTT 等等。

多语言客户端（Many Clients） RabbitMQ 几乎支持所有常用语言，比如 Java、.NET、Ruby 等等。

管理界面（Management UI） RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息 Broker 的许多方面。

跟踪机制（Tracing） 如果消息异常，RabbitMQ 提供了消息跟踪机制，使用者可以找出发生了什么。

插件机制（Plugin System） RabbitMQ 提供了许多插件，来从多方面进行扩展，也可以编写自己的插件。

RabbitMQ 提供了点对点、请求/回复(Request/Reply)、发布/订阅(Pub/Sub)等多种通信模式。其可以被广泛应用于异步处理、应用解耦、流量削峰等场景。RabbitMQ 采用了智能代理(Smart Broker)与简单消费者(Dumb Consumer)的模式，Broker 会保持对于 Consumer 的状态追踪，而不需要 Consumer 自身记录消费进度。

![image](https://user-images.githubusercontent.com/5803001/45857187-33faa180-bd8a-11e8-8917-730d896c428b.png)

RabbitMQ 支持同步或者异步的通信模式，Producer 将消息发送到 Exchanges，Exchanges 根据规则将消息路由分发到不同的 Queue，单条消息可以根据规则发送到不同的 Queue 中。

![image](https://user-images.githubusercontent.com/5803001/45831462-05e97300-bd32-11e8-812e-3ce1e958e4ed.png)

Consumer 则是直接从 Queue 中拉取数据，每个 Consumer 可以绑定不同的 Queue，而每个 Queue 也可以被多个 Consumer 绑定。如果有多个消费者同时订阅同一个队列的话，RabbitMQ 是采用循环的方式分发消息的，每一条消息只能被一个订阅者接收。

## Components | 组件

RabbitMQ 主要包括以下组件：

1. Server(broker): 接受客户端连接，实现 AMQP 消息队列和路由功能的进程。

2. Virtual Host: 其实是一个虚拟概念，类似于权限控制组，一个 Virtual Host 里面可以有若干个 Exchange 和 Queue，但是权限控制的最小粒度是 Virtual Host。表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

3.Exchange:接受生产者发送的消息，并根据 Binding 规则将消息路由给服务器中的队列。ExchangeType 决定了 Exchange 路由消息的行为，例如，在 RabbitMQ 中，ExchangeType 有 direct、Fanout 和 Topic 三种，不同类型的 Exchange 路由的行为是不一样的。

4.Message Queue：消息队列，用于存储还未被消费者消费的消息。消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括 routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。

5.Message: 由 Header 和 Body 组成，Header 是由生产者添加的各种属性的集合，包括 Message 是否被持久化、由哪个 Message Queue 接受、优先级是多少等。而 Body 是真正需要传输的 APP 数据。

6.Binding:Binding 联系了 Exchange 与 Message Queue。Exchange 在与多个 Message Queue 发生 Binding 后会生成一张路由表，路由表中存储着 Message Queue 所需消息的限制条件即 Binding Key。当 Exchange 收到 Message 时会解析其 Header 得到 Routing Key，Exchange 根据 Routing Key 与 Exchange Type 将 Message 路由到 Message Queue。Binding Key 由 Consumer 在 Binding Exchange 与 Message Queue 时指定，而 Routing Key 由 Producer 发送 Message 时指定，两者的匹配方式由 Exchange Type 决定。

7.Connection:连接，对于 RabbitMQ 而言，其实就是一个位于客户端和 Broker 之间的 TCP 连接。

8.Channel:信道，仅仅创建了客户端到 Broker 之间的连接后，客户端还是不能发送消息的。需要为每一个 Connection 创建 Channel，AMQP 协议规定只有通过 Channel 才能执行 AMQP 的命令。一个 Connection 可以包含多个 Channel。之所以需要 Channel，是因为 TCP 连接的建立和释放都是十分昂贵的，如果一个客户端每一个线程都需要与 Broker 交互，如果每一个线程都建立一个 TCP 连接，暂且不考虑 TCP 连接是否浪费，就算操作系统也无法承受每秒建立如此多的 TCP 连接。RabbitMQ 建议客户端线程之间不要共用 Channel，至少要保证共用 Channel 的线程发送消息必须是串行的，但是建议尽量共用 Connection。多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的 TCP 连接内地虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。

9.Command:AMQP 的命令，客户端通过 Command 完成与 AMQP 服务器的交互来实现自身的逻辑。例如在 RabbitMQ 中，客户端可以通过 publish 命令发送消息，txSelect 开启一个事务，txCommit 提交一个事务。

![image](https://user-images.githubusercontent.com/5803001/45918017-01989380-beb3-11e8-91f4-19a7c6fed0e2.png)

![image](https://user-images.githubusercontent.com/5803001/51668768-e32bdb80-1ffd-11e9-9a32-486690a335d7.png)

# 消息路由

AMQP 中消息的路由过程和 Java 开发者熟悉的 JMS 存在一些差别，AMQP 中增加了 Exchange 和 Binding 的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到那个队列。

![image](https://user-images.githubusercontent.com/5803001/51668736-d6a78300-1ffd-11e9-8195-15142165a29a.png)

Exchange 分发消息时根据类型的不同分发策略有区别，目前共四种类型：direct、fanout、topic、headers 。headers 匹配 AMQP 消息的 header 而不是路由键，此外 headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型：

消息中的路由键（routing key）如果和 Binding 中的 binding key 一致， 交换器就将消息发到对应的队列中。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为“dog”，则只转发 routing key 标记为“dog”的消息，不会转发“dog.puppy”，也不会转发“dog.guard”等等。它是完全匹配、单播的模式。

每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发消息是最快的。

topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符：符号“#”和符号“”。#匹配 0 个或多个单词，匹配不多不少一个单词。

在 RabbitMQ 中，无论是生产者发送消息还是消费者接受消息，都首先需要声明一个 MessageQueue。消费者是无法订阅或者获取不存在的 MessageQueue 中信息；消息被 Exchange 接受以后，如果没有匹配的 Queue，则会被丢弃，因此如果是消费者去声明 Queue，就有可能会出现在声明 Queue 之前，生产者已发送的消息被丢弃的隐患。

```java
final String QUEUE_NAME = "HELLO";
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");

Connection connection = factory.newConnection();
Channel channel = connection.createChannel();

// 通用生成消息的方法
private static String getMessage(String[] strings) {
    if (strings.length < 2)
        return "Hello World!";
    return String.join(" ",Arrays.copyOfRange(strings, 1, strings.length));
}
```

通过 queue.declare 命令声明一个队列，可以设置该队列以下属性:

- Exclusive：排他队列，如果一个队列被声明为排他队列，该队列仅对首次声明它的连接可见，并在连接断开时自动删除。这里需要注意三点：其一，排他队列是基于连接可见的，同一连接的不同信道是可以同时访问同一个连接创建的排他队列的。其二，“首次”，如果一个连接已经声明了一个排他队列，其他连接是不允许建立同名的排他队列的，这个与普通队列不同。其三，即使该队列是持久化的，一旦连接关闭或者客户端退出，该排他队列都会被自动删除的。这种队列适用于只限于一个客户端发送读取消息的应用场景。

- Auto-delete:自动删除，如果该队列没有任何订阅的消费者的话，该队列会被自动删除。这种队列适用于临时队列。

- Durable: 持久化。

## Direct

```java
// Producer
channel.queueDeclare(QUEUE_NAME, false, false, false, null);
String message = "Hello World!";
channel.basicPublish("", QUEUE_NAME, null, message.getBytes("UTF-8"));
System.out.println(" [x] Sent '" + message + "'");

// Consumer
channel.queueDeclare(QUEUE_NAME, false, false, false, null);
System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

Consumer consumer = new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body)
        throws IOException {
    String message = new String(body, "UTF-8");
    System.out.println(" [x] Received '" + message + "'");
    }
};
channel.basicConsume(QUEUE_NAME, true, consumer);
```

## Direct Exchange

在 AMQP 模型中，Exchange 是接受生产者消息并将消息路由到消息队列的关键组件。ExchangeType 和 Binding 决定了消息的路由规则。所以生产者想要发送消息，首先必须要声明一个 Exchange 和该 Exchange 对应的 Binding。可以通过 ExchangeDeclare 和 BindingDeclare 完成。在 Rabbit MQ 中，声明一个 Exchange 需要三个参数：ExchangeName，ExchangeType 和 Durable。ExchangeName 是该 Exchange 的名字，该属性在创建 Binding 和生产者通过 publish 推送消息时需要指定。

- ExchangeType，指 Exchange 的类型，在 RabbitMQ 中，有三种类型的 Exchange：direct ，fanout 和 topic，不同的 Exchange 会表现出不同路由行为。
- Durable 是该 Exchange 的持久化属性。

```java
// Producer
channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

// 表示消息严重性，这里使用严重性表示 BindingKey 与 RoutingKey
String severity = getSeverity(argv);
String message = getMessage(argv);

// 生产者可以动态指定 RoutingKey
channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes("UTF-8"));
System.out.println(" [x] Sent '" + severity + "':'" + message + "'");

// Consumer
channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
String queueName = channel.queueDeclare().getQueue();

if (argv.length < 1){
    System.err.println("Usage: ReceiveLogsDirect [info] [warning] [error]");
    System.exit(1);
}

// 绑定不同严重性的消息
for(String severity: argv){
    // Consumer 可以绑定多个 BindingKey
    channel.queueBind(queueName, EXCHANGE_NAME, severity);
}
System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

Consumer consumer = new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope,
                                AMQP.BasicProperties properties, byte[] body) throws IOException {
    String message = new String(body, "UTF-8");
    System.out.println(" [x] Received '" + envelope.getRoutingKey() + "':'" + message + "'");
    }
};
channel.basicConsume(queueName, true, consumer);
```

声明一个 Binding 需要提供一个 QueueName，ExchangeName 和 BindingKey。生产者在发送消息时，都需要指定一个 RoutingKey 和 Exchange，Exchange 在接到该 RoutingKey 以后，会判断该 ExchangeType。如果是 Direct 类型，则会将消息中的 RoutingKey 与该 Exchange 关联的所有 Binding 中的 BindingKey 进行比较，如果相等，则发送到该 Binding 对应的 Queue 中。

![image](https://user-images.githubusercontent.com/5803001/51668709-c8596700-1ffd-11e9-873f-8745eaf7594c.png)

## Fanout

如果是 Fanout 类型，则会将消息发送给所有与该 Exchange 定义过 Binding 的所有 Queues 中去，其实是一种广播行为。

![image](https://user-images.githubusercontent.com/5803001/51665862-a65ce600-1ff7-11e9-961c-789c62d1c470.png)

## Topic Exchange

如果是 Topic 类型，则会按照正则表达式，对 RoutingKey 与 BindingKey 进行匹配，如果匹配成功，则发送到对应的 Queue 中。

![image](https://user-images.githubusercontent.com/5803001/51665694-54b45b80-1ff7-11e9-9971-fd3c7333ce05.png)

## Task Queue

```java
// Publisher
channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);

String message = getMessage(argv);

channel.basicPublish("", TASK_QUEUE_NAME,
        MessageProperties.PERSISTENT_TEXT_PLAIN,
        message.getBytes("UTF-8"));
System.out.println(" [x] Sent '" + message + "'");

// Worker
channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

channel.basicQos(1);

final Consumer consumer = new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
    String message = new String(body, "UTF-8");

    System.out.println(" [x] Received '" + message + "'");
    try {
        doWork(message);
    } finally {
        System.out.println(" [x] Done");
        channel.basicAck(envelope.getDeliveryTag(), false);
    }
    }
};

channel.basicConsume(TASK_QUEUE_NAME, false, consumer);

private static void doWork(String task) {
    for (char ch : task.toCharArray()) {
        if (ch == '.') {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException _ignored) {
            Thread.currentThread().interrupt();
        }
        }
    }
}
```

# 消费者

在 RabbitMQ 中消费者有 2 种方式获取队列中的消息:

a) 一种是通过 basic.consume 命令，订阅某一个队列中的消息,channel 会自动在处理完上一条消息之后，接收下一条消息。（同一个 channel 消息处理是串行的）。除非关闭 channel 或者取消订阅，否则客户端将会一直接收队列的消息。

b) 另外一种方式是通过 basic.get 命令主动获取队列中的消息，但是绝对不可以通过循环调用 basic.get 来代替 basic.consume，这是因为 basic.get RabbitMQ 在实际执行的时候，是首先 consume 某一个队列，然后检索第一条消息，然后再取消订阅。如果是高吞吐率的消费者，最好还是建议使用 basic.consume。

如果有多个消费者同时订阅同一个队列的话，RabbitMQ 是采用循环的方式分发消息的，每一条消息只能被一个订阅者接收。例如，有队列 Queue，其中 ClientA 和 ClientB 都 Consume 了该队列，MessageA 到达队列后，被分派到 ClientA，ClientA 回复服务器收到响应，服务器删除 MessageA；再有一条消息 MessageB 抵达队列，服务器根据“循环推送”原则，将消息会发给 ClientB，然后收到 ClientB 的确认后，删除 MessageB；等到再下一条消息时，服务器会再将消息发送给 ClientA。

这里我们可以看出，消费者再接到消息以后，都需要给服务器发送一条确认命令，这个即可以在 handleDelivery 里显示的调用 basic.ack 实现，也可以在 Consume 某个队列的时候，设置 autoACK 属性为 true 实现。这个 ACK 仅仅是通知服务器可以安全的删除该消息，而不是通知生产者，与 RPC 不同。 如果消费者在接到消息以后还没来得及返回 ACK 就断开了连接，消息服务器会重传该消息给下一个订阅者，如果没有订阅者就会存储该消息。

既然 RabbitMQ 提供了 ACK 某一个消息的命令，当然也提供了 Reject 某一个消息的命令。当客户端发生错误，调用 basic.reject 命令拒绝某一个消息时，可以设置一个 requeue 的属性，如果为 true，则消息服务器会重传该消息给下一个订阅者；如果为 false，则会直接删除该消息。当然，也可以通过 ack，让消息服务器直接删除该消息并且不会重传。

# 存储

## 持久化

Rabbit MQ 默认是不持久队列、Exchange、Binding 以及队列中的消息的，这意味着一旦消息服务器重启，所有已声明的队列，Exchange，Binding 以及队列中的消息都会丢失。通过设置 Exchange 和 MessageQueue 的 durable 属性为 true，可以使得队列和 Exchange 持久化，但是这还不能使得队列中的消息持久化，这需要生产者在发送消息的时候，将 delivery mode 设置为 2，只有这 3 个全部设置完成后，才能保证服务器重启不会对现有的队列造成影响。这里需要注意的是，只有 durable 为 true 的 Exchange 和 durable 为 ture 的 Queues 才能绑定，否则在绑定时，RabbitMQ 都会抛错的。持久化会对 RabbitMQ 的性能造成比较大的影响，可能会下降 10 倍不止。

## 事务

对事务的支持是 AMQP 协议的一个重要特性。假设当生产者将一个持久化消息发送给服务器时，因为 consume 命令本身没有任何 Response 返回，所以即使服务器崩溃，没有持久化该消息，生产者也无法获知该消息已经丢失。如果此时使用事务，即通过 txSelect()开启一个事务，然后发送消息给服务器，然后通过 txCommit()提交该事务，即可以保证，如果 txCommit()提交了，则该消息一定会持久化，如果 txCommit()还未提交即服务器崩溃，则该消息不会服务器就收。当然 Rabbit MQ 也提供了 txRollback()命令用于回滚某一个事务。

## Confirm

使用事务固然可以保证只有提交的事务，才会被服务器执行。但是这样同时也将客户端与消息服务器同步起来，这背离了消息队列解耦的本质。Rabbit MQ 提供了一个更加轻量级的机制来保证生产者可以感知服务器消息是否已被路由到正确的队列中——Confirm。如果设置 channel 为 confirm 状态，则通过该 channel 发送的消息都会被分配一个唯一的 ID，然后一旦该消息被正确的路由到匹配的队列中后，服务器会返回给生产者一个 Confirm，该 Confirm 包含该消息的 ID，这样生产者就会知道该消息已被正确分发。对于持久化消息，只有该消息被持久化后，才会返回 Confirm。Confirm 机制的最大优点在于异步，生产者在发送消息以后，即可继续执行其他任务。而服务器返回 Confirm 后，会触发生产者的回调函数，生产者在回调函数中处理 Confirm 信息。如果消息服务器发生异常，导致该消息丢失，会返回给生产者一个 nack，表示消息已经丢失，这样生产者就可以通过重发消息，保证消息不丢失。Confirm 机制在性能上要比事务优越很多。但是 Confirm 机制，无法进行回滚，就是一旦服务器崩溃，生产者无法得到 Confirm 信息，生产者其实本身也不知道该消息吃否已经被持久化，只有继续重发来保证消息不丢失，但是如果原先已经持久化的消息，并不会被回滚，这样队列中就会存在两条相同的消息，系统需要支持去重。

# Transaction | 事务

# Internals | 内部实现

## MessageQueue | 消息队列

RabbitMQ 完全实现了 AMQP 协议，类似于一个邮箱服务。Exchange 负责根据 ExchangeType 和 RoutingKey 将消息投递到对应的消息队列中，消息队列负责在消费者获取消息前暂存消息。在 RabbitMQ 中，MessageQueue 主要由两部分组成，一个为 AMQQueue，主要负责实现 AMQP 协议的逻辑功能。

![image](https://user-images.githubusercontent.com/5803001/45955430-74844480-c042-11e8-8f5b-3899fdaa161c.png)

在 RabbitMQ 中 BackingQueue 又由 5 个子队列组成：Q1、Q2、Delta、Q3 和 Q4。RabbitMQ 中的消息一旦进入队列，不是固定不变的，它会随着系统的负载在队列中不断流动，消息的状态不断发生变化。RabbitMQ 中的消息一共有 5 种状态：

a)Alpha：消息的内容和消息索引都保存在内存中；

b)Beta：消息内容保存在磁盘上，消息索引保存在内存中；

c)Gamma：消息内容保存在磁盘上，消息索引在磁盘和内存都有；

d)Delta：消息内容和索引都在磁盘上；

对于持久化的消息，消息内容和消息索引都必须先保存到磁盘上，才会处于上述状态中的一种，而 Gamma 状态的消息只有持久化的消息才会有该状态。

BackingQueue 中的 5 个子队列中的消息状态， Q1 和 Q4 对应的是 Alpha 状态， Q2 和 Q3 是 Beta 状态， Delta 对应的是 Delta 状态。上述就是 RabbitMQ 的多层队列结构的设计，我们可以看出从 Q1 到 Q4 ，基本经历的是由 RAM 到 DISK，再到 RAM 的设计。这样的设计的好处就是当队列负载很高的情况下，能够通过将一部分消息由磁盘保存来节省内存空间，当负载降低的时候，这部分消息又渐渐回到内存，被消费者获取，使得整个队列有很好的弹性。下面我们就来看一下，整个消息队列的工作流程。

引起消息流动主要有两方面的因素：其一是消费者获取消息；其二是由于内存不足，引起消息的换出到磁盘上（ Q1-.>Q2 、 Q2->Delta 、 Q3->Delta 、 Q4->Q3 ）。 RabbitMQ 在系统运行时会根据消息传输的速度计算一个当前内存中能够保存的最大消息数量（ Target_RAM_Count ），当内存中的消息数量大于该值时，就会引起消息的流动。进入队列的消息，一般会按着 Q1->Q2->Delta->Q3->Q4 的顺序进行流动，但是并不是每条消息都一定会经历所有的状态，这个取决于当时系统的负载状况。

当消费者获取消息时，首先会从 Q4 队列中获取消息，如果 Q4 获取成功，则返回，如果 Q4 为空，则尝试从 Q3 获取消息；首先，系统会判断 Q3 队列是否为空，如果为空，则直接返回队列为空，即此时队列中无消息（后续会论证）。如果不为空，则取出 Q3 的消息，然后判断此时 Q3 和 Delta 队列的长度，如果都为空，则可认为 Q2 、 Delta 、 Q3 和 Q4 全部为空 (后续说明 ) ，此时将 Q1 中消息直接转移到 Q4 中，下次直接从 Q4 中获取消息。如果 Q3 为空， Delta 不空，则将 Delta 中的消息转移到 Q3 中；如果 Q3 非空，则直接下次从 Q3 中获取消息。在将 Delta 转移到 Q3 的过程中， RabbitMQ 是按照索引分段读取的，首先读取某一段，直到读到的消息非空为止，然后判断读取的消息个数与 Delta 中的消息个数是否相等，如果相等，则断定此时 Delta 中已无消息，则直接将 Q2 和刚读到的消息一并放入 Q3 中。如果不相等，则仅将此次读到的消息转移到 Q3 中。这就是消费者引起的消息流动过程。

![image](https://user-images.githubusercontent.com/5803001/45955475-9e3d6b80-c042-11e8-9752-c588d777b89c.png)

由于内存不足引起的消息换出。消息换出的条件是内存中保存的消息数量 + 等待 ACK 的消息的数量 >Target_RAM_Count 。当条件触发时，系统首先会判断如果当前进入等待 ACK 的消息的速度大于进入队列的消息的速度时，会先处理等待 ACK 的消息。步骤基本上 Q1->Q2 或者 Q3 移动，取决于 Delta 队列是否为空。 Q4->Q3 移动， Q2 和 Q3 向 Delta 移动。

为什么 Q3 队列为空即可认定整个队列为空。试想如果 Q3 为空，Delta 不空，则在 Q3 取出最后一条消息时， Delta 上的消息就会被转移到 Q3 上，与 Q3 空矛盾。如果 Q2 不空，则在 Q3 取出最后一条消息，如果 Delta 为空时，会将 Q2 的消息并入 Q3 ，与 Q3 为空矛盾。如果 Q1 不空，则在 Q3 取出最后一条消息，如果 Delta 和 Q3 均为空时，则将 Q1 的消息转移到 Q4 中，与 Q4 为空矛盾。这也解释了另外一个问题，即为什么 Q3 和 Delta 为空， Q2 就为空。
