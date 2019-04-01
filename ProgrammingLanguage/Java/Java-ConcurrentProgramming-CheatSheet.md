[![返回目录](https://i.postimg.cc/JzFTMvjF/image.png)](https://github.com/wx-chevalier/Awesome-CheatSheets)

> 本文参考了许多经典的文章描述/示例，统一声明在了 [Java Concurrent Programming Links](https://parg.co/UDS)。

# Java 并发编程概论：内存模型，并发单元，并发控制与异步模式

参考[并发编程导论]()中的介绍，并发编程主要会考虑并发单元、并发控制与异步模式等方面；本文即是着眼于 Java，具体地讨论 Java 中并发编程相关的知识要点。Java 是典型的共享内存的并发模型，线程之间的通信往往是隐式进行。

# Java Memory Model | Java 内存模型

Java 内存模型与 [JVM 体系结构](./JVM-CheatSheet.md) 不尽相同，Java 内存模型着眼于描述 Java 中的线程是如何与内存进行交互，以及单线程中代码执行的顺序等，并提供了一系列基础的并发语义原则；最早的 Java 内存模型于 1995 年提出，致力于解决不同处理器/操作系统中线程交互/同步的问题。

Prior to Java 5, the Java Memory Model (JMM) was ill defined. It was possible to get all kinds of strange results when shared memory was accessed by multiple threads, such as:

a thread not seeing values written by other threads: a visibility problem
a thread observing ‘impossible’ behavior of other threads, caused by instructions not being executed in the order expected: an instruction reordering problem.
With the implementation of JSR 133 in Java 5, a lot of these issues have been resolved. The JMM is a set of rules based on the “happens-before” relation, which constrain when one memory access must happen before another, and conversely, when they are allowed to happen out of order. Two examples of these rules are:

The monitor lock rule: a release of a lock happens before every subsequent acquire of the same lock.
The volatile variable rule: a write of a volatile variable happens before every subsequent read of the same volatile variable
Although the JMM can seem complicated, the specification tries to find a balance between ease of use and the ability to write performant and scalable concurrent data structures.

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

# Concurrent Primitive | 并发单元

常见的 Runnable、Callable、Future、FutureTask 这几个与线程相关的类或者接口：

- Runnable 应该是我们最熟悉的接口，它只有一个 run()函数，用于将耗时操作写在其中，该函数没有返回值。然后使用某个线程去执行该 runnable 即可实现多线程，Thread 类在调用 start()函数后就是执行的是 Runnable 的 run()函数。

- Callable 与 Runnable 的功能大致相似，Callable 中有一个 call()函数，但是 call()函数有返回值，而 Runnable 的 run()函数不能将结果返回给客户程序。

- Executor 就是 Runnable 和 Callable 的调度容器，Future 就是对于具体的 Runnable 或者 Callable 任务的执行结果进行取消、查询是否完成、获取结果、设置结果操作。get 方法会阻塞，直到任务返回结果。

- FutureTask 则是一个 RunnableFuture<V>，而 RunnableFuture 实现了 Runnbale 又实现了 Futrue<V> 这两个接口。

## Threads & Runnables

Timer 计时器具备使任务延迟执行以及周期性执行的功能，但是 Timer 天生存在一些缺陷，所以从 JDK 1.5 开始就推荐使用 ScheduledThreadPoolExecutor（ScheduledExecutorService 实现类）作为其替代工具。

首先 Timer 对提交的任务调度是基于绝对时间而不是相对时间的，所以通过其提交的任务对系统时钟的改变是敏感的（譬如提交延迟任务后修改了系统时间会影响其执行）；而 ScheduledThreadExecutor 只支持相对时间，对系统时间不敏感。

接着 Timer 的另一个问题是如果 TimerTask 抛出未检查异常则 Timer 将会产生无法预料的行为，因为 Timer 线程并不捕获异常，所以 TimerTask 抛出的未检查异常会使 Timer 线程终止，所以后续提交的任务得不到执行；而 ScheduledThreadPoolExecutor 不存在此问题。

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

## Fork/Join

# Concurrency Control | 并发控制

## Atomic Variables | 原子性与原子变量

## volatile | 可见性保障

## 锁与同步

# Async Programming | 异步编程

## Callable & Future

## CompletableFuture

## RxJava

# Thread Communication | 线程通信

# Built-in ThreadSafe DataStructure | 内置的线程安全模型

# Todos

- [concurrency-torture-testing-your-code-within-the-java-memory-model](http://zeroturnaround.com/rebellabs/concurrency-torture-testing-your-code-within-the-java-memory-model/)

- https://www.baeldung.com/java-fork-join
