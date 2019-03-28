[![返回目录](https://parg.co/UCb)](https://github.com/wx-chevalier/Awesome-CheatSheets)

> 整理自[官方文档]()以及 [Akka Links]() 中归档的参考资料。

# Akka CheatSheet

In today's world, computer hardware is becoming cheaper and more powerful, as we have multiple cores on a single CPU chip. As cores keep on increasing, with the increasing power of hardware, we need a state of the art software framework which can use these cores efficiently.

Akka is such a framework, or you can say, a toolkit, which utilizes the hardware cores efficiently and lets you write performant applications.

Thus, Akka provides a basic unit of abstraction of transparent distribution called actors, which form the basis for writing resilient, elastic, event-driven, and responsive systems.

Let's see what is meant by these properties:

Resilient: Applications that can heal themselves, which means they can recover from failure, and will always be responsive, even in case of failure like if we get errors or exceptions

Elastic: A system which is responsive under varying amount of workload, that is, the system always remains responsive, irrespective of increasing or decreasing traffic, by increasing or decreasing the resources allocated to service this workload

Message Driven: A system whose components are loosely coupled with each other and communicate using asynchronous message passing, and which reacts to those messages by taking an action

Responsive: A system that satisfies the preceding three properties is called responsive

Akka 是一个建立在 Actors 概念和可组合 Futures 之上的并发框架, Akka 设计灵感来源于 Erlang；Erlang 是基于 Actor 模型构建的。它通常被用来取代阻塞锁如同步、读写锁及类似的更高级别的异步抽象。Netty 是一个异步网络库，使 JAVA NIO 的功能更好用。Akka 针对 IO 操作有一个抽象，这和 netty 是一样的。使用 Akka 可以用来创建计算集群,Actor 在不同的机器之间传递消息。从这个角度来看,Akka 相对于 Netty 来说，是一个更高层次的抽象

Akka 是一种高度可扩展的软件，这不仅仅表现在性能方面，也表现在它所适用的应用的大小。Akka 的核心，Akka-actor 是非常小的，可以非常方便地放进你的应用中，提供你需要的异步无锁并行功能，不会有任何困扰。你可以任意选择 Akka 的某些部分集成到你的应用中，也可以使用完整的包——Akka 微内核，它是一个独立的容器，可以直接部署你的 Akka 应用。随着 CPU 核数越来越多，即使你只使用一台电脑，Akka 也可作为一种提供卓越性能的选择。 Akka 还同时提供多种并发范型，允许用户选择正确的工具来完成工作。

[Backend Boilerplate/akka]()

# Actor 基础

The Actor Model provides a higher level of abstraction for writing concurrent and distributed systems. It alleviates the developer from having to deal with explicit locking and thread management, making it easier to write correct concurrent and parallel systems. Actors were defined in the 1973 paper by Carl Hewitt but have been popularized by the Erlang language, and used for example at Ericsson with great success to build highly concurrent and reliable telecom systems.

Before starting with recipes, let's take a look at the following the actor properties:

State: An actor has internal state, which is mutated sequentially as messages are processed one by one.
Behavior: An Actor reacts to messages which are sent to it by applying behavior on it.
Communication: An actor communicates with other actors by sending and receiving messages to/from them.
Mailbox: A mailbox is the queue of messages from which an actor picks up the message and processes it.

Actors are message-driven, that is, they are passive and do nothing unless and until you send messages to them. Once you send them a message, they pick a thread from the thread pool which is also known as a dispatcher, process the message, and release the thread back to the thread pool.

Actors are also asynchronous by nature; they never block your current thread of execution, and continue to work on another thread.

## ActorSystem

An actor in Akka always belongs to a parent. Typically, you create an actor by calling getContext().actorOf(). Rather than creating a “freestanding” actor, this injects the new actor as a child into an already existing tree: the creator actor becomes the parent of the newly created child actor. You might ask then, who is the parent of the first actor you create?

As illustrated below, all actors have a common parent, the user guardian. New actor instances can be created under this actor using system.actorOf().

![image](https://user-images.githubusercontent.com/5803001/47984204-d12e5100-e110-11e8-892d-44bef8d6ba04.png)

- / the so-called root guardian. This is the parent of all actors in the system, and the last one to stop when the system itself is terminated.

- /user the guardian. This is the parent actor for all user created actors. Don’t let the name user confuse you, it has nothing to do with end users, nor with user handling. Every actor you create using the Akka library will have the constant path /user/ prepended to it.

- /system the system guardian.

## Actor 声明与实例化

最简单的 Actor 声明即是继承自 AbstractActor，并且实现了 createReceive 方法，该方法会自定义消息的处理方式。

```java
class PrintMyActorRefActor extends AbstractActor {
  @Override
  public Receive createReceive() {
    return receiveBuilder()
        .matchEquals("printit", p -> {
          // 创建新的子 Actor 实例
          ActorRef secondRef = getContext().actorOf(Props.empty(), "second-actor");
          System.out.println("Second: " + secondRef);
        })
        .build();
  }
}
```

```java
// 精确匹配某个值
.matchEquals
// 匹配某个类对象
.match(String.class, s -> {
log.info("Received String message: {}", s);
})
// 匹配任意对象
.matchAny(o -> log.info("received unknown message"))
```

然后在 Actor System 中注册该 Actor 的实例对象：

```java
ActorSystem system = ActorSystem.create("testSystem");

// 创建某个 Actor
ActorRef firstRef = system.actorOf(Props.create(PrintMyActorRefActor.class), "first-actor");
System.out.println("First: " + firstRef);

// 发送消息
firstRef.tell("printit", ActorRef.noSender());

// 终止该系统
system.terminate();

// First: Actor[akka://testSystem/user/first-actor#1053618476]
// Second: Actor[akka://testSystem/user/first-actor/second-actor#-1544706041]
```

Props is a configuration class to specify options for the creation of actors, think of it as an immutable and thus freely shareable recipe for creating an actor including associated deployment information.

```java
Props props1 = Props.create(MyActor.class);
// 该方法不建议使用
Props props2 = Props.create(ActorWithArgs.class,
  () -> new ActorWithArgs("arg"));
Props props3 = Props.create(ActorWithArgs.class, "arg");
```

## 消息传递

### 消息定义

在真实场景下，我们往往会使用 POJO 或者 Enum 做消息的结构化传递：

```java
// GreeterActor
public static enum Msg {
    GREET, DONE;
}

@Override
public Receive createReceive() {
    return receiveBuilder()
        .matchEquals(Msg.GREET, m -> {
        System.out.println("Hello World!");
        sender().tell(Msg.DONE, self());
    })
    .build();
}
```

### Future 异步调用

类似于 Java 中的 Future，可以将一个 actor 的返回结果重定向到另一个 actor 中进行处理，主 actor 或者进程无需等待 actor 的返回结果。

```java
ActorSystem system = ActorSystem.create("strategy", ConfigFactory.load("akka.config"));
ActorRef printActor = system.actorOf(Props.create(PrintActor.class), "PrintActor");
ActorRef workerActor = system.actorOf(Props.create(WorkerActor.class), "WorkerActor");

//等等future返回
Future<Object> future = Patterns.ask(workerActor, 5, 1000);
int result = (int) Await.result(future, Duration.create(3, TimeUnit.SECONDS));
System.out.println("result:" + result);

//不等待返回值，直接重定向到其他actor，有返回值来的时候将会重定向到printActor
Future<Object> future1 = Patterns.ask(workerActor, 8, 1000);
Patterns.pipe(future1, system.dispatcher()).to(printActor);


workerActor.tell(PoisonPill.getInstance(), ActorRef.noSender());]
```

## LifeCycle | 生命周期

![image](https://user-images.githubusercontent.com/5803001/47840701-1fcaab00-ddf2-11e8-93a9-a0cd34601dd0.png)

```java
class StartStopActor1 extends AbstractActor {
  @Override
  public void preStart() {
    ...
    getContext().actorOf(Props.create(StartStopActor2.class), "second");
  }

  @Override
  public void postStop() {
    ...
  }

  @Override
  public Receive createReceive() {
    return receiveBuilder()
        .matchEquals("stop", s -> {
          getContext().stop(getSelf());
        })
        .build();
  }
}
```

## 各类型 Actor

### Typed Actor & AbstractActor

早期 Akka 库中常见的是 TypedActor 与 UntypedActor，TypedActor 的优势在于其内置了静态协议，而不需要用户自定义消息类型；并且 UntypedActor 已经被弃用，而应该使用 AbstractActor。

### AbstractLoggingActor

```java
public static class Terminator extends AbstractLoggingActor {

    private final ActorRef ref;

    public Terminator(ActorRef ref) {
        this.ref = ref;
        getContext().watch(ref);
    }

    @Override
    public Receive createReceive() {
        return receiveBuilder()
        .match(Terminated.class, t -> {
            log().info("{} has terminated, shutting down system", ref.path());
            getContext().system().terminate();
        })
        .build();
    }
}

// 可以用来监控其他的 Actor
ActorRef actorRef = system.actorOf(Props.create(HelloWorld.class), "helloWorld");
system.actorOf(Props.create(Terminator.class, actorRef), "terminator");
```

# 邮箱与路由

## Mailbox

整个 akka 的 Actor 系统是通过消息进行传递的，其实还可以使用 Inbox 消息收件箱来给某个 actor 发消息，并且可以进行交互。

```java
// 创建 Inbox 收件箱
Inbox inbox = Inbox.create(system);
// 监听一个 actor
inbox.watch(inboxTest);

//通过inbox来发送消息
inbox.send(inboxTest, Msg.WORKING);
inbox.send(inboxTest, Msg.DONE);
inbox.send(inboxTest, Msg.CLOSE);
```

然后在 Inbox 中可以循环等待消息回复：

```java
while(true){
    try {
        Object receive = inbox.receive(Duration.create(1, TimeUnit.SECONDS));
        if(receive == Msg.CLOSE){//收到的inbox的消息
            System.out.println("inboxTextActor is closing");
        }else if(receive instanceof Terminated){//中断 ，和线程一个概念
            System.out.println("inboxTextActor is closed");
            system.terminate();
            break;
        }else {
            System.out.println(receive);
        }
    } catch (TimeoutException e) {
        e.printStackTrace();
    }
}
```

## 路由与投递

通常在分布式任务调度系统中会有这样的需求：一组 actor 提供相同的服务，我们在调用任务的时候只需要选择其中一个 actor 进行处理即可。其实这就是一个负载均衡或者说路由策略，akka 作为一个高性能支持并发的 actor 模型，可以用来作为任务调度集群使用，当然负载均衡就是其本职工作了，akka 提供了 Router 来进行消息的调度。

```java
// RouterActor，用于分发消息的 Actor
// 创建多个子 Actor
ArrayList<Routee> routees = new ArrayList<>();

for(int i = 0; i < 5; i ++) {
    //借用上面的 inboxActor
    ActorRef worker = getContext().actorOf(Props.create(InboxTest.class), "worker_" + i);
    getContext().watch(worker);//监听
    routees.add(new ActorRefRoutee(worker));
}

/**
* 创建路由对象
* RoundRobinRoutingLogic: 轮询
* BroadcastRoutingLogic: 广播
* RandomRoutingLogic: 随机
* SmallestMailboxRoutingLogic: 空闲
*/
router = new Router(new RoundRobinRoutingLogic(), routees);

@Override
public void onReceive(Object o) throws Throwable {
    if(o instanceof InboxTest.Msg){
        // 进行路由转发
        router.route(o, getSender());
    }else if(o instanceof Terminated){
        // 发生中断，将该actor删除
        router = router.removeRoutee(((Terminated)o).actor());
        System.out.println(((Terminated)o).actor().path() + " 该actor已经删除。router.size=" + router.routees().size());

        // 没有可用 actor 了
        if(router.routees().size() == 0){
            System.out.print("没有可用actor了，系统关闭。");
            flag.compareAndSet(true, false);
            getContext().system().shutdown();
        }
    }else {
        unhandled(o);
    }
}

// 外部系统照常发送消息
ActorRef routerActor = system.actorOf(Props.create(RouterActor.class), "RouterActor");
routerActor.tell(Msg.WORKING, ActorRef.noSender());
```

# Concurrency Control & Transaction | 并发控制与事务

## Memory Model | 内存模型

With the Actors implementation in Akka, there are two ways multiple threads can execute actions on shared memory:

if a message is sent to an actor (e.g. by another actor). In most cases messages are immutable, but if that message is not a properly constructed immutable object, without a “happens before” rule, it would be possible for the receiver to see partially initialized data structures and possibly even values out of thin air (longs/doubles).
if an actor makes changes to its internal state while processing a message, and accesses that state while processing another message moments later. It is important to realize that with the actor model you don’t get any guarantee that the same thread will be executing the same actor for different messages.
To prevent visibility and reordering problems on actors, Akka guarantees the following two “happens before” rules:

The actor send rule: the send of the message to an actor happens before the receive of that message by the same actor.
The actor subsequent processing rule: processing of one message happens before processing of the next message by the same actor.

## Dispatcher

## STM 软件事务内存

# Error Handling & Persistence | 异常处理与持久化

## Fault Tolerance | 容错机制

## Persistence | 持久化

Akka 持久化可以使有状态的 actor 能够保持其内部状态，以便在启动、JVM 崩溃后重新启动、或在集群中迁移时，恢复它们的内部状态。 Akka 持久性关键点在于，只有对 actor 内部状态的更改才会被持久化，而不会直接保持其当前状态（可选快照除外）。 这些更改只会追加到存储，没有任何修改，这允许非常高的事务速率和高效的复制；通过加载持久化的数据 stateful actors 可以重建内部状态。

Akka 持久性扩展依赖一些内置持久性插件，包括基于内存堆的日志，基于本地文件系统的快照存储和基于 LevelDB 的日志。

```xml
<dependency>
    <groupId>org.iq80.leveldb</groupId>
    <artifactId>leveldb</artifactId>
    <version>0.7</version>
</dependency>
```

然后我们还需要针对持久化策略添加相关的配置：

```
akka.persistence.journal.plugin = "akka.persistence.journal.leveldb"
akka.persistence.snapshot-store.plugin = "akka.persistence.snapshot-store.local"

akka.persistence.journal.leveldb.dir = "target/example/journal"
akka.persistence.snapshot-store.local.dir = "target/example/snapshots"

# DO NOT USE THIS IN PRODUCTION !!!
# See also https://github.com/typesafehub/activator/issues/287
akka.persistence.journal.leveldb.native = false
```

当我们声明某个可持久化的 Actor 时，需要使其继承自 AbstractPersistentActor:

```java
class ExamplePersistentActor extends AbstractPersistentActor {}
```

然后复写 createReceiveRecover 与 createReceive 方法；createReceive 是正常的处理消息的方法，而 createReceiveRecover 则是用于在恢复阶段处理接收到的消息的方法。

```java
@Override
public Receive createReceiveRecover() {
    return receiveBuilder()
        // 恢复之前在上一个快照点之后发布的 Event
        .match(Evt.class, e -> state.update(e))
        // 恢复之前保存的状态
        .match(SnapshotOffer.class, ss -> state = (ExampleState) ss.snapshot())
        .build();
}

@Override
public Receive createReceive() {
    return receiveBuilder()
        .match(Cmd.class, c -> {
            final String data = c.getData();
            final Evt evt = new Evt(data + "-" + getNumEvents());

            // 持久化消息
            persist(evt, (Evt event) -> {
                state.update(event);
                getContext().system().eventStream().publish(event);
            });
        })
        // 触发持久化当前状态
        .matchEquals("snap", s -> saveSnapshot(state.copy()))
        .matchEquals("print", s -> System.out.println(state))
        .build();
}
```

在外部调用时，我们可以手动地触发进行状态存储：

```java
persistentActor.tell(new Cmd("foo"), null);
persistentActor.tell("snap", null);
persistentActor.tell(new Cmd("buzz"), null);
persistentActor.tell("print", null);
```

# HTTP 请求处理

## WebSocket

# Spring Boot 与 Akka 集成

```java

```

# Todos

- https://doc.akka.io/docs/akka/current/actors.html#become-unbecome
