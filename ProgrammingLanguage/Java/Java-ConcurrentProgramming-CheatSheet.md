[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Java 并发编程概览：内存模型，并发单元，并发控制与异步模式

参考[并发编程导论]()中的介绍，并发编程主要会考虑并发单元、并发控制与异步模式等方面；本文即是着眼于 Java，具体地讨论 Java 中并发编程相关的知识要点。Java 是典型的共享内存的并发模型，线程之间的通信往往是隐式进行。

本文参考了许多经典的文章描述/示例，统一声明在了 [Java Concurrent Programming Links](https://parg.co/UDS)。

# Java Memory Model: Java 内存模型

Java 内存模型与 [JVM 体系结构](./JVM-CheatSheet.md) 不尽相同，Java 内存模型着眼于描述 Java 中的线程是如何与内存进行交互，以及单线程中代码执行的顺序等，并提供了一系列基础的并发语义原则。最早的 Java 内存模型于 1995 年提出，致力于解决不同处理器/操作系统中线程交互/同步的问题。

## Abstract Memory Model: 抽象内存模型

![](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-5.png)

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

![](http://tutorials.jenkov.com/images/java-concurrency/java-memory-model-3.png)

## Cache Line & False Sharing: 缓存行与伪共享

缓存系统中是以缓存行（cache line）为单位存储的。缓存行是 2 的整数幂个连续字节，一般为 32-256 个字节。最常见的缓存行大小是 64 个字节。当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。图 1 说明了伪共享的问题。在核心 1 上运行的线程想更新变量 X，同时核心 2 上的线程想要更新变量 Y。不幸的是，这两个变量在同一个缓存行中。每个线程都要去竞争缓存行的所有权来更新变量。如果核心 1 获得了所有权，缓存子系统将会使核心 2 中对应的缓存行失效。当核心 2 获得了所有权然后执行更新操作，核心 1 就要使自己对应的缓存行失效。这会来来回回的经过 L3 缓存，大大影响了性能。如果互相竞争的核心位于不同的插槽，就要额外横跨插槽连接，问题可能更加严重。

![](http://ifeve.com/wp-content/uploads/2013/01/cache-line.png)

参考 [Java 内存布局]()可知，所有对象都有两个字长的对象头。第一个字是由 24 位哈希码和 8 位标志位（如锁的状态或作为锁对象）组成的 Mark Word。第二个字是对象所属类的引用。如果是数组对象还需要一个额外的字来存储数组的长度。每个对象的起始地址都对齐于 8 字节以提高性能。因此当封装对象的时候为了高效率，对象字段声明的顺序会被重排序成下列基于字节大小的顺序：

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
