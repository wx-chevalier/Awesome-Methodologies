# Java 并发编程概览

# Java Memory Model: Java 内存模型

## Cache Line & False Sharing: 缓存行与伪共享

缓存系统中是以缓存行（cache line）为单位存储的。缓存行是2的整数幂个连续字节，一般为32-256个字节。最常见的缓存行大小是64个字节。当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。图1说明了伪共享的问题。在核心1上运行的线程想更新变量X，同时核心2上的线程想要更新变量Y。不幸的是，这两个变量在同一个缓存行中。每个线程都要去竞争缓存行的所有权来更新变量。如果核心1获得了所有权，缓存子系统将会使核心2中对应的缓存行失效。当核心2获得了所有权然后执行更新操作，核心1就要使自己对应的缓存行失效。这会来来回回的经过L3缓存，大大影响了性能。如果互相竞争的核心位于不同的插槽，就要额外横跨插槽连接，问题可能更加严重。

![](http://ifeve.com/wp-content/uploads/2013/01/cache-line.png)

参考 [Java 内存布局]()可知，所有对象都有两个字长的对象头。第一个字是由24位哈希码和8位标志位（如锁的状态或作为锁对象）组成的Mark Word。第二个字是对象所属类的引用。如果是数组对象还需要一个额外的字来存储数组的长度。每个对象的起始地址都对齐于8字节以提高性能。因此当封装对象的时候为了高效率，对象字段声明的顺序会被重排序成下列基于字节大小的顺序：

doubles (8) 和 longs (8)
ints (4) 和 floats (4)
shorts (2) 和 chars (2)
booleans (1) 和 bytes (1)
references (4/8)
<子类字段重复上述顺序>

```java
public final static class VolatileLong
{
    public volatile long value = 0L;
    public long p1, p2, p3, p4, p5, p6; // 添加该行，错开缓存行，避免伪共享
}
```

多线程

# Concurrent Primitive: 并发单元

# Concurrency Control: 并发控制
