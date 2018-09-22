[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# RabbitMQ CheatSheet

RabbitMQ 是一个由 erlang 开发的 AMQP(Advanved Message Queue)的开源实现，是经典的消息代理(Message Broker)/消息队列(Message Queue)；其提供了点对点、请求/回复(Request/Reply)、发布/订阅(Pub/Sub)等多种通信模式。其可以被广泛应用于异步处理、应用解耦、流量削峰等场景。RabbitMQ 采用了智能代理(Smart Broker)与简单消费者(Dumb Consumer)的模式，Broker 会保持对于 Consumer 的状态追踪，而不需要 Consumer 自身记录消费进度。

![image](https://user-images.githubusercontent.com/5803001/45857187-33faa180-bd8a-11e8-8917-730d896c428b.png)

RabbitMQ 支持同步或者异步的通信模式，Producer 将消息发送到 Exchanges，Exchanges 根据规则将消息路由分发到不同的 Queue，Consumer 直接从 Queue 中拉取数据。

![image](https://user-images.githubusercontent.com/5803001/45831462-05e97300-bd32-11e8-812e-3ce1e958e4ed.png)

BindingKey 是 Exchange 和 Queue 绑定的规则描述，这个描述用来解析当 Exchange 接收到消息时，Exchange 接收到的消息会带有 RoutingKey 这个字段，Exchange 就是根据这个 RoutingKey 和当前 Exchange 所有绑定的 BindingKey 做匹配，如果满足要求，就往 BindingKey 所绑定的 Queue 发送消息，这样我们就解决了我们向 RabbitMQ 发送一次消息，可以分发到不同的 Queue 的过程。

- ConnectionFactory：与 RabbitMQ 服务器连接的管理器

- Connection：与 RabbitMQ 服务器的连接

- Channel：与 Exchange 的连接

- Exchange：接受消息提供者（生产者）的消息，并根据消息的 RoutingKey 和 Exchange 绑定的 BindingKey 分配消息

- Queue：存储消息接收者（消费者）的消息

- RoutingKey：指定当前消息被谁接受

- BindingKey：指定当前 Exchange 下，什么样的 RoutingKey 会被下派到当前绑定的 Queue 中

# 消息路由

## Direct

```java
final String QUEUE_NAME = "HELLO";
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");

Connection connection = factory.newConnection();
Channel channel = connection.createChannel();

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

## Task Queue
