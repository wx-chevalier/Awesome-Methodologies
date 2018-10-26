[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Java 并发编程概览：内存模型，并发单元，并发控制与异步模式

参考[并发编程导论]()中的介绍，并发编程主要会考虑并发单元、并发控制与异步模式等方面；本文即是着眼于 Java，具体地讨论 Java 中并发编程相关的知识要点。Java 是典型的共享内存的并发模型，线程之间的通信往往是隐式进行。

本文参考了许多经典的文章描述/示例，统一声明在了 [Java Concurrent Programming Links](https://parg.co/UDS)。

# Java Memory Model | Java 内存模型

Java 内存模型与 [JVM 体系结构](./JVM-CheatSheet.md) 不尽相同，Java 内存模型着眼于描述 Java 中的线程是如何与内存进行交互，以及单线程中代码执行的顺序等，并提供了一系列基础的并发语义原则；最早的 Java 内存模型于 1995 年提出，致力于解决不同处理器/操作系统中线程交互/同步的问题。

## Abstract Memory Model | 抽象内存模型

现代计算机通常有两个或者更多的 CPU，一些 CPU 还有多个核。在这样的计算机中，可能同时运行着多个线程，而每个 CPU 在某个时间片内运行其中的一个线程。

每个 CPU 包含多个寄存器，这些寄存器本质上就是 CPU 内存。CPU 在寄存器中执行操作的速度会比在主内存中操作快非常多。因为寄存器的访问速度比主内存的访问速度快很多。

每个 CPU 可能还拥有 CPU 缓存层，CPU 访问缓存层的速度比访问主内存块很多，但是却比访问寄存器要慢。计算机还包括主内存（RAM），所有的 CPU 都可以访问这个主内存，主内存一般都比 CPU 缓存大很多，但速度要比 CPU 缓存慢。

通常情况，当一个 CPU 需要访问主内存的时候，会把主内存中的部分数据读取到 CPU 缓存，甚至进一步把缓存中的部分数据读取到内部的寄存器，然后对其进行操作。当 CPU 需要向主内存写数据的时候，会将寄存器中的数据写入缓存，某些时候会将数据从缓存刷入主内存。无论从缓存读还是写数据，都没有必要一次性全部读出或者写入，而是部分数据。

![](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-5.png)

Java 虚拟机内运行的每个线程都拥有一个属于自己的线程栈（调用栈），随着线程代码的执行，调用栈会随之改变。

线程栈中包含每个正在执行的方法的局部变量。每个线程只能访问属于自己的栈。调用栈中的局部变量，只有创建这个栈的线程才可以访问，其他线程都不能访问。即使两个线程在执行一段相同的代码，这两个线程也会在属于各自的线程栈中创建局部变量。因此，每个线程拥有属于自己的局部变量。

所有基本类型的局部变量全部存放在线程栈中，对其他线程不可见。一个线程可以把基本类型拷贝到其他线程，但是不能共享给其他线程。

无论哪个线程创建的对象都存放在堆中。

```java
public class MySharedObject {

    //static variable pointing to instance of MySharedObject

    public static final MySharedObject sharedInstance =
        new MySharedObject();


    //member variables pointing to two objects on the heap

    public Integer object2 = new Integer(22);
    public Integer object4 = new Integer(44);

    public long member1 = 12345;
    public long member1 = 67890;
}
```

```java
public class MyRunnable implements Runnable() {

    public void run() {
        methodOne();
    }

    public void methodOne() {
        int localVariable1 = 45;

        MySharedObject localVariable2 =
            MySharedObject.sharedInstance;

        //... do more with local variables.

        methodTwo();
    }

    public void methodTwo() {
        Integer localVariable1 = new Integer(99);

        //... do more with local variable.
    }
}
```

Java 内存模型和硬件内存架构不一样。硬件内存架构不区分线程栈和堆内存。在硬件中，线程栈和堆内存都分配在主内存中。部分线程栈和堆数据有时可能出现在 CPU 缓存中，有时可能出现在寄存器中。

![](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-3.png)

如果多个线程共享一个对象，如果没有合理的使用 volatile 声明和线程同步，一个线程更新共享对象后，另一个线程可能无法取到对象的最新值。

如图，共享变量存储在主内存。运行在某个 CPU 中的线程将共享变量读取到自己的 CPU 缓存。在 CPU 缓存中，修改了共享对象的值，由于 CPU 并未将缓存中的数据刷回主内存，导致对共享变量的修改对于在另一个 CPU 中运行的线程而言是不可见的。这样每个线程都会拥有一份属于自己的共享变量的拷贝，分别存于各自对应的 CPU 缓存中。

如果多个线程共享一个对象，当多个线程更新这个变量时，会引发竞争条件。
想象下，如果有两个线程分别运行在两个 CPU 中，两个线程分别将同一个共享变量读取到各自的 CPU 缓存。现在线程一将变量加一，线程二也将变量加一，当两个 CPU 缓存的数据刷回主内存时，变量的值只加了一，并没有加二。同步锁可以确保一段代码块同时只有一个线程可以进入。同步锁可以确保被保护代码块内所有的变量都是从主内存获取的，当被保护代码块执行完毕时，所有变量的更新都会刷回主内存，无论这些变量是否用 volatile 修饰。

## Cache Line & False Sharing | 缓存行与伪共享

缓存系统中是以缓存行(cache line)为单位存储的。缓存行是 2 的整数幂个连续字节，一般为 32-256 个字节。最常见的缓存行大小是 64 个字节。当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。图 1 说明了伪共享的问题。在核心 1 上运行的线程想更新变量 X，同时核心 2 上的线程想要更新变量 Y。不幸的是，这两个变量在同一个缓存行中。每个线程都要去竞争缓存行的所有权来更新变量。如果核心 1 获得了所有权，缓存子系统将会使核心 2 中对应的缓存行失效。当核心 2 获得了所有权然后执行更新操作，核心 1 就要使自己对应的缓存行失效。这会来来回回的经过 L3 缓存，大大影响了性能。如果互相竞争的核心位于不同的插槽，就要额外横跨插槽连接，问题可能更加严重。

![](http://ifeve.com/wp-content/uploads/2013/01/cache-line.png)

参考 [Java 内存布局]()可知，所有对象都有两个字长的对象头。第一个字是由 24 位哈希码和 8 位标志位(如锁的状态或作为锁对象)组成的 Mark Word。第二个字是对象所属类的引用。如果是数组对象还需要一个额外的字来存储数组的长度。每个对象的起始地址都对齐于 8 字节以提高性能。因此当封装对象的时候为了高效率，对象字段声明的顺序会被重排序成下列基于字节大小的顺序：

```
doubles (8) 和 longs (8)
ints (4) 和 floats (4)
shorts (2) 和 chars (2)
booleans (1) 和 bytes (1)
references (4/8)
<子类字段重复上述顺序>
```

```java
public final static class VolatileLong
{
    public volatile long value = 0L;
    public long p1, p2, p3, p4, p5, p6; // 添加该行，错开缓存行，避免伪共享
}
```

注释前后程序的执行时间可能有数倍差距，一条缓存行有 64 字节, 而 Java 程序的对象头固定占 8 字节(32 位系统)或 12 字节(64 位系统默认开启压缩, 不开压缩为 16 字节)。我们只需要填 6 个无用的长整型补上`6*8=48`字节, 让不同的 VolatileLong 对象处于不同的缓存行, 就可以避免伪共享了(64 位系统超过缓存行的 64 字节也无所谓,只要保证不同线程不要操作同一缓存行就可以). 这个办法叫做补齐(Padding).某些 Java 编译器会将没有使用到的补齐数据, 即示例代码中的 6 个长整型在编译时优化掉, 可以在程序中加入一些代码防止被编译优化。

```java
public static long preventFromOptimization(VolatileLong v) {
	return v.p1 + v.p2 + v.p3 + v.p4 + v.p5 + v.p6;
}
```

# Concurrent Primitive: 并发单元

# Concurrency Control: 并发控制

[TOC]

# Introduction

## Reference

### Books & Tutorials

- [Java 并发编程的艺术-By 方腾飞]()
- [Java 并发编程实战](http://book.51cto.com/art/201203/323171.htm)
- [Java 多线程编程核心技术]()

# Concurrence(并发之线程)

> - [Java 并发的四种风味：Thread、Executor、ForkJoin 和 Actor](http://www.open-open.com/lib/view/open1421202894171.html)
> - [java8-concurrency-tutorial](http://winterbe.com/posts/2015/04/07/java8-concurrency-tutorial-thread-executor-examples/)
> - [java-concurrency](http://tutorials.jenkov.com/java-concurrency/index.html)
> - [java-util-concurrent](http://tutorials.jenkov.com/java-util-concurrent/index.html)

## Threads & Runnables

Timer 计时器具备使任务延迟执行以及周期性执行的功能，但是 Timer 天生存在一些缺陷，所以从 JDK 1.5 开始就推荐使用 ScheduledThreadPoolExecutor（ScheduledExecutorService 实现类）作为其替代工具。

首先 Timer 对提交的任务调度是基于绝对时间而不是相对时间的，所以通过其提交的任务对系统时钟的改变是敏感的（譬如提交延迟任务后修改了系统时间会影响其执行）；而 ScheduledThreadExecutor 只支持相对时间，对系统时间不敏感。

接着 Timer 的另一个问题是如果 TimerTask 抛出未检查异常则 Timer 将会产生无法预料的行为，因为 Timer 线程并不捕获异常，所以 TimerTask 抛出的未检查异常会使 Timer 线程终止，所以后续提交的任务得不到执行；而 ScheduledThreadPoolExecutor 不存在此问题。

> 并发是同一时间应对(dealing with)多件事情的能力；并行是同一时间动手做(doing)多件事情的能力。

所有的现代操作系统都通过进程和线程来支持并发。进程是通常彼此独立运行的程序的实例，比如，如果你启动了一个 Java 程序，操作系统产生一个新的进程，与其他程序一起并行执行。在这些进程的内部，我们使用线程并发执行代码，因此，我们可以最大限度的利用 CPU 可用的核心(core)。 Java 从 JDK1.0 开始执行线程。在开始一个新的线程之前，你必须指定由这个线程执行的代码，通常称为 task。这可以通过实现 Runnable——一个定义了一个无返回值无参数的 run()方法的函数接口，如下面的代码所示：

```java
Runnable task = () -> {
    String threadName = Thread.currentThread().getName();
    System.out.println("Hello " + threadName);
};

task.run();

Thread thread = new Thread(task);
thread.start();

System.out.println("Done!");
```

因为 Runnable 是一个函数接口，所以我们利用 lambda 表达式将当前的线程名打印到控制台。首先，在开始一个线程前我们在主线程中直接运行 runnable。 控制台输出的结果可能像下面这样：

```
Hello main
Hello Thread-0
Done!
```

或者这样：

```
Hello main
Done!
Hello Thread-0
```

## Executors

并发 API 引入了 ExecutorService 作为一个在程序中直接使用 Thread 的高层次的替换方案。Executos 支持运行异步任务，通常管理一个线程池，这样一来我们就不需要手动去创建新的线程。在不断地处理任务的过程中，线程池内部线程将会得到复用，因此，在我们可以使用一个 executor service 来运行和我们想在我们整个程序中执行的一样多的并发任务。 下面是使用 executors 的第一个代码示例：

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.submit(() -> {
	String threadName = Thread.currentThread().getName();
	System.out.println("Hello " + threadName);
});
// => Hello pool-1-thread-1
```

Executors 类提供了便利的工厂方法来创建不同类型的 executor services。在这个示例中我们使用了一个单线程线程池的 executor。 代码运行的结果类似于上一个示例，但是当运行代码时，你会注意到一个很大的差别：Java 进程从没有停止！Executors 必须显式的停止-否则它们将持续监听新的任务。 ExecutorService 提供了两个方法来达到这个目的——shutdwon()会等待正在执行的任务执行完而 shutdownNow()会终止所有正在执行的任务并立即关闭 execuotr。

```java
try {
    System.out.println("attempt to shutdown executor");
    executor.shutdown();
    executor.awaitTermination(5, TimeUnit.SECONDS);
    }
catch (InterruptedException e) {
    System.err.println("tasks interrupted");
}
finally {
    if (!executor.isTerminated()) {
        System.err.println("cancel non-finished tasks");
    }
    executor.shutdownNow();
    System.out.println("shutdown finished");
}
```

executor 通过等待指定的时间让当前执行的任务终止来“温柔的”关闭 executor。在等待最长 5 分钟的时间后，execuote 最终会通过中断所有的正在执行的任务关闭。

### invokeAll:调用所有的 Callable

Executors 支持通过 invokeAll()一次批量提交多个 callable。这个方法结果一个 callable 的集合，然后返回一个 future 的列表。

```java
ExecutorService executor = Executors.newWorkStealingPool();

List<Callable<String>> callables = Arrays.asList(
        () -> "task1",
        () -> "task2",
        () -> "task3");

executor.invokeAll(callables)
    .stream()
    .map(future -> {
        try {
            return future.get();
        }
        catch (Exception e) {
            throw new IllegalStateException(e);
        }
    })
    .forEach(System.out::println);
```

在这个例子中，我们利用 Java8 中的函数流(stream)来处理 invokeAll()调用返回的所有 future。我们首先将每一个 future 映射到它的返回值，然后将每个值打印到控制台。如果你还不属性 stream，可以阅读我的[Java8 Stream 教程](http://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/)。

### invokeAny

批量提交 callable 的另一种方式就是 invokeAny()，它的工作方式与 invokeAll()稍有不同。在等待 future 对象的过程中，这个方法将会阻塞直到第一个 callable 中止然后返回这一个 callable 的结果。 为了测试这种行为，我们利用这个帮助方法来模拟不同执行时间的 callable。这个方法返回一个 callable，这个 callable 休眠指定 的时间直到返回给定的结果。

```java
//这个callable方法是用来构造不同的Callable对象
Callable<String> callable(String result, long sleepSeconds) {
    return () -> {
        TimeUnit.SECONDS.sleep(sleepSeconds);
        return result;
    };
}
```

我们利用这个方法创建一组 callable，这些 callable 拥有不同的执行时间，从 1 分钟到 3 分钟。通过 invokeAny()将这些 callable 提交给一个 executor，返回最快的 callable 的字符串结果-在这个例子中为任务 2：

```
ExecutorService executor = Executors.newWorkStealingPool();

List<Callable<String>> callables = Arrays.asList(
callable("task1", 2),
callable("task2", 1),
callable("task3", 3));

String result = executor.invokeAny(callables);
System.out.println(result);

// => task2
```

### Scheduled Executors

为了持续的多次执行常见的任务，我们可以利用调度线程池 ScheduledExecutorService 支持任务调度，持续执行或者延迟一段时间后执行。 下面的实例，调度一个任务在延迟 3 分钟后执行：

```java
ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);

Runnable task = () -> System.out.println("Scheduling: " + System.nanoTime());
ScheduledFuture<?> future = executor.schedule(task, 3, TimeUnit.SECONDS);

TimeUnit.MILLISECONDS.sleep(1337);

long remainingDelay = future.getDelay(TimeUnit.MILLISECONDS);
System.out.printf("Remaining Delay: %sms", remainingDelay);
```

调度一个任务将会产生一个专门的 future 类型——ScheduleFuture，它除了提供了 Future 的所有方法之外，他还提供了 getDelay()方法来获得剩余的延迟。在延迟消逝后，任务将会并发执行。 为了调度任务持续的执行，executors 提供了两个方法 scheduleAtFixedRate()和 scheduleWithFixedDelay()。第一个方法用来以固定频率来执行一个任务，比如，下面这个示例中，每分钟一次：

```java
ScheduledExecutorService executor =     Executors.newScheduledThreadPool(1);

Runnable task = () -> System.out.println("Scheduling: " + System.nanoTime());

int initialDelay = 0;
int period = 1;
executor.scheduleAtFixedRate(task, initialDelay, period, TimeUnit.SECONDS);
```

另外，这个方法还接收一个初始化延迟，用来指定这个任务首次被执行等待的时长。 请记住：scheduleAtFixedRate()并不考虑任务的实际用时。所以，如果你指定了一个 period 为 1 分钟而任务需要执行 2 分钟，那么线程池为了性能会更快的执行。在这种情况下，你应该考虑使用 scheduleWithFixedDelay()。这个方法的工作方式与上我们上面描述的类似。不同之处在于等待时间 period 的应用是在一次任务的结束和下一个任务的开始之间。例如：

## Concurrence Test

# 线程安全

## Atomic Variables(原子性与原子变量)

## 锁与同步

## 可见性

Java 中的 volatile 关键字主要即是保证了变量的可见性，而不是原子性，譬如[Java](http://cpro.baidu.com/cpro/ui/uijs.php?adclass=0&app_id=0&c=news&cf=1001&ch=0&di=128&fv=19&is_app=0&jk=9220db91f2f6efed&k=java&k0=java&kdi0=0&luki=5&mcpm=0&n=10&p=baidu&q=65035100_cpr&rb=0&rs=1&seller_id=1&sid=edeff6f291db2092&ssp2=1&stid=0&t=tpclicked3_hc&td=1836545&tu=u1836545&u=http%3A%2F%2Fwww%2Ebubuko%2Ecom%2Finfodetail%2D481580%2Ehtml&urlid=0)语言规范描述：

> 每一个变量都有一个主内存。为了保证最佳性能，JVM 允许线程从主内存拷贝一份私有拷贝，然后在线程读取变量的时候从主内存里面读，退出的时候，将修改的值同步到主内存。

形象而言，对于变量 t。A 线程对 t 变量修改的值，对 B 线程是可见的。但是 A 获取到 t 的值加 1 之后，突然挂起了，B 获取到的值还是最新的值，volatile 能保证 B 能获取到的 t 是最新的值，因为 A 的 t+1 并没有写到主内存里面去。这个[逻辑](http://cpro.baidu.com/cpro/ui/uijs.php?adclass=0&app_id=0&c=news&cf=1001&ch=0&di=128&fv=19&is_app=0&jk=9220db91f2f6efed&k=%C2%DF%BC%AD&k0=%C2%DF%BC%AD&kdi0=0&luki=2&mcpm=0&n=10&p=baidu&q=65035100_cpr&rb=0&rs=1&seller_id=1&sid=edeff6f291db2092&ssp2=1&stid=0&t=tpclicked3_hc&td=1836545&tu=u1836545&u=http%3A%2F%2Fwww%2Ebubuko%2Ecom%2Finfodetail%2D481580%2Ehtml&urlid=0)是没有问题的。

在实际的编程中，要注意，除非是在保证仅有一个线程处于写，而其他线程处于读的状态下的时候，才可以使用 volatile 来保证可见性，而不需要使用原子变量或者锁来保证原子性。

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicInteger;

public class Counter {

	public static AtomicInteger count = new AtomicInteger();//原子操作
	public static CountDownLatch latch= new CountDownLatch(1000);//线程协作处理
	public static volatile int countNum = 0;//volatile    只能保证可见性，不能保证原子性
	public static int synNum = 0;//同步处理计算

	public static void inc() {

		try {
			Thread.sleep(1);
		} catch (InterruptedException e) {
		}
		countNum++;
		int c = count.addAndGet(1);
		add();
		System.out.println(Thread.currentThread().getName() + "------>" + c);
	}

	public static synchronized void add(){
		synNum++;
	}

	public static void main(String[] args) {

		//同时启动1000个线程，去进行i++计算，看看实际结果

		for (int i = 0; i < 1000; i++) {
			new Thread(new Runnable() {
				@Override
				public void run() {
					Counter.inc();
					latch.countDown();
				}
			},"thread" + i).start();
		}
		try {
			latch.await();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println(Thread.currentThread().getName());

		System.out.println("运行结果:Counter.count=" + count.get() + ",,," + countNum + ",,," + synNum);
	}
}
```

# 线程通信

# Built-in ThreadSafe DataStructure(内置的线程安全模型)

## ConcurrentMap

## Sync Utils:同步辅助

### **CountDownLatch**

一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。用给定的计数 初始化 CountDownLatch。由于调用了 countDown() 方法，所以在当前计数到达零之前，await 方法会一直受阻塞。之后，会释放所有等待的线程，await 的所有后续调用都将立即返回。这种现象只出现一次——计数无法被重置。  一个线程(或者多个)， 等待另外 N 个线程完成某个事情之后才能执行。在一些应用场合中，需要等待某个条件达到要求后才能做后面的事情；同时当线程都完成后也会触发事件，以便进行后面的操作。 这个时候就可以使用 CountDownLatch。CountDownLatch 最重要的方法是 countDown()和 await()，前者主要是倒数一次，后者是等待倒数到 0，如果没有到达 0，就只有阻塞等待了。

```java
public void countDown()
```

递减锁存器的计数，如果计数到达零，则释放所有等待的线程。如果当前计数大于零，则将计数减少。如果新的计数为零，出于线程调度目的，将重新启用所有的等待线程。如果当前计数等于零，则不发生任何操作。

```java
public boolean await(long timeout,
                     TimeUnit unit)
              throws InterruptedException
```

使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断或超出了指定的等待时间。如果当前计数为零，则此方法立刻返回 true 值。如果当前计数大于零，则出于线程调度目的，将禁用当前线程，且在发生以下三种情况之一前，该线程将一直处于休眠状态：

- 由于调用 countDown() 方法，计数到达零；或者

* 其他某个线程中断当前线程；或者

- 已超出指定的等待时间。

如果计数到达零，则该方法返回 true 值。如果当前线程在进入此方法时已经设置了该线程的中断状态；或者在等待时被中断，则抛出 InterruptedException，并且清除当前线程的已中断状态。如果超出了指定的等待时间，则返回值为 false。如果该时间小于等于零，则此方法根本不会等待。

```java
public class CountDownLatchTest {

    // 模拟了100米赛跑，10名选手已经准备就绪，只等裁判一声令下。当所有人都到达终点时，比赛结束。
    public static void main(String[] args) throws InterruptedException {

        // 开始的倒数锁
        final CountDownLatch begin = new CountDownLatch(1);

        // 结束的倒数锁
        final CountDownLatch end = new CountDownLatch(10);

        // 十名选手
        final ExecutorService exec = Executors.newFixedThreadPool(10);

        for (int index = 0; index < 10; index++) {
            final int NO = index + 1;
            Runnable run = new Runnable() {
                public void run() {
                    try {
                        // 如果当前计数为零，则此方法立即返回。
                        // 等待
                        begin.await();
                        Thread.sleep((long) (Math.random() * 10000));
                        System.out.println("No." + NO + " arrived");
                    } catch (InterruptedException e) {
                    } finally {
                        // 每个选手到达终点时，end就减一
                        end.countDown();
                    }
                }
            };
            exec.submit(run);
        }
        System.out.println("Game Start");
        // begin减一，开始游戏
        begin.countDown();
        // 等待end变为0，即所有选手到达终点
        end.await();
        System.out.println("Game Over");
        exec.shutdown();
    }
}
```

结果如下：

```
Game Start
No.9 arrived
No.6 arrived
No.8 arrived
No.7 arrived
No.10 arrived
No.1 arrived
No.5 arrived
No.4 arrived
No.2 arrived
No.3 arrived
Game Over
```

# Concurrence-Asynchronous(并发之异步)

## Callables&Futures

Executors 本身提供了一种对于多线程的封装，而 Executor 还支持另一种类型的任务——Callable。Callables 也是类似于 runnables 的函数接口，不同之处在于，Callable 返回一个值。 Callable 接口本身是一个 Lambda 表达式(函数式接口)：

```java
Callable<Integer> task = () -> {
    try {
        TimeUnit.SECONDS.sleep(1);
        return 123;
    }
    catch (InterruptedException e)
        throw new IllegalStateException("task interrupted", e);
    }
};
```

Callbale 也可以像 runnbales 一样提交给 executor services。但是 callables 的结果怎么办？因为 submit()不会等待任务完成，executor service 不能直接返回 callable 的结果。不过，executor 可以返回一个 Future 类型的结果，它可以用来在稍后某个时间取出实际的结果。

```java
ExecutorService executor = Executors.newFixedThreadPool(1);
Future<Integer> future = executor.submit(task);

System.out.println("future done? " + future.isDone());

Integer result = future.get();

System.out.println("future done? " + future.isDone());
System.out.print("result: " + result);
```

在调用 get()方法时，当前线程会阻塞等待，直到 callable 在返回实际的结果 123 之前执行完成。现在 future 执行完毕，我们可以在控制台看到如下的结果：

```
future done? false
future done? true
result: 123
```

Future 与底层的 executor service 紧密的结合在一起。记住，如果你关闭 executor，所有的未中止的 future 都会抛出异常。

```
executor.shutdownNow();
future.get();
```

你可能注意到我们这次创建 executor 的方式与上一个例子稍有不同。我们使用 newFixedThreadPool(1)来创建一个单线程线程池的 execuot service。 这等同于使用 newSingleThreadExecutor，不过使用第二种方式我们可以稍后通过简单的传入一个比 1 大的值来增加线程池的大小。

### Timeouts

任何 future.get()调用都会阻塞，然后等待直到 callable 中止。在最糟糕的情况下，一个 callable 持续运行——因此使你的程序将没有响应。我们可以简单的传入一个时长来避免这种情况。

```java
ExecutorService executor = Executors.newFixedThreadPool(1);

Future<Integer> future = executor.submit(() -> {
    try {
        TimeUnit.SECONDS.sleep(2);
        return 123;
    }
    catch (InterruptedException e) {
        throw new IllegalStateException("task interrupted", e);
    }
});

future.get(1, TimeUnit.SECONDS);
```

运行上面的代码将会产生一个 TimeoutException：

```
Exception in thread "main" java.util.concurrent.TimeoutException
    at java.util.concurrent.FutureTask.get(FutureTask.java:205)
```

## Promise

# [RxJava](https://github.com/ReactiveX/RxJava)-Reactive Programming(响应式编程)

## Quick Start

笔者在 J2EE 领域还是倾向于使用 Maven，直接在 pom 文件中添加如下依赖即可：

```xml
<dependency>
    <groupId>io.reactivex</groupId>
    <artifactId>rxjava</artifactId>
    <version>1.0.10</version>
</dependency>
```

添加了 Pom 依赖项之后，即可以引入 Observable 以及 Subscribe 对象：

```java
import rx.Observable;

import java.util.ArrayList;
import java.util.List;

public class RxUsingJava8 {

    public static void main(String args[]) {

        /*
         * Example using single-value lambdas (Func1)
         */
        Observable.from(1, 2, 3, 4, 5)
                .filter((v) -> {
                    return v < 4;
                })
                .subscribe((value) -> {
                    System.out.println("Value: " + value);
                });

        /*
         * Example with 'reduce' that takes a lambda with 2 arguments (Func2)
         */
        Observable.from(1, 2, 3, 4, 5)
                .reduce((seed, value) -> {
                    // sum all values from the sequence
                    return seed + value;
                })
                .map((v) -> {
                    return "DecoratedValue: " + v;
                })
                .subscribe((value) -> {
                    System.out.println(value);
                });

    }
}
```

# Todos

- [concurrency-torture-testing-your-code-within-the-java-memory-model](http://zeroturnaround.com/rebellabs/concurrency-torture-testing-your-code-within-the-java-memory-model/)
