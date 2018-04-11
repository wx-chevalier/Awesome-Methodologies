[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# JVM 内部架构与运行机制盘点

# 体系架构

# GC 垃圾回收

在 JDK10 的代码中，路径为 openjdk/src/hotspot/share/gc/，各个 GC 实现共享依赖 shared 代码，GC 包括目前默认的 G1，也有经典的 Serial、Parallel、CMS 等 GC 实现。

# Introduction

## Reference

### Tutorials & Docs

* [JAVA 虚拟机的生命周期](http://www.tuicool.com/articles/BVz2qqq)

### Practice

* [听阿里巴巴 JVM 工程师为你分析常见 Java 故障案例](http://dbaplus.cn/news-21-173-1.html?utm_source=tuicool&utm_medium=referral)

### Books & Tools

* [深入理解 Java 虚拟机：JVM 高级特性与最佳实践](http://www.linuxidc.com/Linux/2014-09/106869.htm)
* [JVM 内幕：Java 虚拟机详解](www.importnew.com/17770.html?utm_source=tuicool&utm_medium=referral)

# JVM

## JVM 参数管理

> [关键业务系统的 JVM 启动参数推荐](http://calvin1978.blogcn.com/articles/jvmoption-2.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

## JVM 体系结构

## 基于栈的架构

Java 选择了基于栈的架构往往在单运算的操作次数上会多于基于寄存器的架构，而 Google 提出的 Android 操作系统设计的 Dalvik 虚拟机则是采用了基于寄存器的架构。

## 执行引擎

每当创建一个新的线程时，JVM 会为该线程创建一个 Java 栈，同时会为这个线程分配一个 PC 寄存器，并且这个 PC 寄存器会指向这个线程的第一行可执行代码。每当调用一个新方法时会在这个栈上创建一个新的栈帧数据结构，这个栈帧会保留这个方法的一些元信息，如在这个方法中定义的局部变量、一些用来支持常量池的解析、正常方法返回以及异常处理机制等等。

![](http://hi.csdn.net/attachment/201009/25/0_1285381395C6iW.gif)

# JVM Operation

## Config

> * [JVM 监控与调优](http://my.oschina.net/91jason/blog/493870?p={{page}})

## Debug

> * [JVM 调试工具说明](http://blog.csdn.net/jiushuai/article/details/8455788)
> * [Java VisualVM ](http://ihuangweiwei.iteye.com/blog/1219302)

# Class

## Class 文件结构

> [实例探索 Class 文件](http://www.importnew.com/17086.html)

## ClassLoader

> [Java 类的连接与初始化 (及 2013 阿里初始化笔试题解析)](http://www.importnew.com/17105.html)

# 内存管理

Java 中，我们常见的需要内存的组件包括：

* Java 堆
* 线程
* 类和类加载器
* NIO
* JNI

## JVM 内存结构

Java 虚拟机会将内存分为几个不同的管理区，这些区域各自有各自的用途，根据不同的特点，承担不同的任务以及在垃圾回收时运用不同的算法。总体分为下面几个部分：程序计数器（Program Counter Register）、JVM 虚拟机栈（JVM Stacks）、本地方法栈（Native Method Stacks）、堆（Heap）、方法区（Method Area）

![](http://img.my.csdn.net/uploads/201211/23/1353648016_8668.jpg)

**1、程序计数器（Program Counter Register）**

这是一块比较小的内存，不在 Ram 上，而是直接划分在 CPU 上的，程序员无法直接操作它，它的作用是：JVM 在解释字节码文件（.class）时，存储当前线程所执行的字节码的行号，只是一种概念模型，各种 JVM 所采用的方式不同，字节码解释器工作时，就是通过改变程序计数器的值来选取下一条要执行的指令，分支、循环、跳转、等基础功能都是依赖此技术区完成的。还有一种情况，就是我们常说的 Java 多线程方面的，多线程就是通过现程轮流切换而达到的，同一时刻，一个内核只能执行一个指令，所以，对于每一个程序来说，必须有一个计数器来记录程序的执行进度，这样，当线程恢复执行的时候，才能从正确的地方开始，所以，每个线程都必须有一个独立的程序计数器，这类计数器为线程私有的内存。如果一个线程正在执行一个 Java 方法，则计数器记录的是字节码的指令的地址，如果执行的一个 Native 方法，则计数器的记录为空，此内存区是唯一一个在 Java 规范中没有任何 OutOfMemoryError 情况的区域。

**2、JVM 虚拟机栈（JVM Stacks）**

JVM 虚拟机栈就是我们常说的堆栈的栈（我们常常把内存粗略分为堆和栈），和程序计数器一样，也是线程私有的，生命周期和线程一样，每个方法被执行的时候会产生一个**栈帧**，用于存储局部变量表、动态链接、操作数、方法出口等信息。方法的执行过程就是栈帧在 JVM 中出栈和入栈的过程。局部变量表中存放的是各种基本数据类型，如 boolean、byte、char、等 8 种，及引用类型（存放的是指向各个对象的内存地址），因此，它有一个特点：内存空间可以在编译期间就确定，运行期不在改变。这个内存区域会有两种可能的 Java 异常：StackOverFlowError 和 OutOfMemoryError。

**3、本地方法栈（Native Method Stacks）**

从名字即可看出，本地方法栈就是用来处理 Java 中的本地方法的，Java 类的祖先类 Object 中有众多 Native 方法，如 hashCode()、wait()等，他们的执行很多时候是借助于操作系统，但是 JVM 需要对他们做一些规范，来处理他们的执行过程。此区域，可以有不同的实现方法，向我们常用的 Sun 的 JVM 就是本地方法栈和 JVM 虚拟机栈是同一个。

**4、堆（Heap）**

堆内存是内存中最重要的一块，也是最有必要进行深究的一部分。因为 Java 性能的优化，主要就是针对这部分内存的。所有的对象实例及数组都是在堆上面分配的（随着 JIT 技术的逐渐成熟，这句话视乎有些绝对，不过至少目前还基本是这样的），可通过-Xmx 和-Xms 来控制堆的大小。JIT 技术的发展产生了新的技术，如栈上分配和标量替换，也许在不久的几年里，即时编译会诞生及成熟，那个时候，“所有的对象实例及数组都是在堆上面分配的”这句话就应该稍微改改了。堆内存是垃圾回收的主要区域，所以在下文垃圾回收板块会重点介绍，此处只做概念方面的解释。在 32 位系统上最大为 2G，64 位系统上无限制。可通过-Xms 和-Xmx 控制，-Xms 为 JVM 启动时申请的最小 Heap 内存，-Xmx 为 JVM 可申请的最大 Heap 内存。

**5、方法区（Method Area）**

方法区是所有线程共享的内存区域，用于存储已经被 JVM 加载的类信息、常量、静态变量等数据，一般来说，方法区属于持久代（关于持久代，会在 GC 部分详细介绍，除了持久代，还有新生代和旧生代），也难怪 Java 规范将方法区描述为堆的一个逻辑部分，但是它不是堆。方法区的垃圾回收比较棘手，就算是 Sun 的 HotSpot VM 在这方面也没有做得多么完美。此处引入方法区中一个重要的概念：运行时常量池。主要用于存放在编译过程中产生的字面量（字面量简单理解就是常量）和引用。一般情况，常量的内存分配在编译期间就能确定，但不一定全是，有一些可能就是运行时也可将常量放入常量池中，如 String 类中有个 Native 方法 intern()<关于intern()的详细说明，请看另一篇文章：[http://blog.csdn.net/zhangerqing/article/details/8093919](http://blog.csdn.net/zhangerqing/article/details/8093919)>

此处补充一个在 JVM 内存管理之外的一个内存区：直接内存。在 JDK1.4 中新加入类 NIO 类，引入了一种基于通道与缓冲区的 I/O 方式，它可以使用 Native 函数库直接分配堆外内存，即我们所说的直接内存，这样在某些场景中会提高程序的性能。

## JVM 内存分配策略

## JVM 内存回收策略

### JVM 引用类型

在 JDK 1.2 以前的版本中，若一个对象不被任何变量引用，那么程序就无法再使用这个对象。也就是说，只有对象处于可触及（reachable）状态，程序才能使用它。从 JDK 1.2 版本开始，把对象的引用分为 4 种级别，从而使程序能更加灵活地控制对象的生命周期。这 4 种级别由高到低依次为：强引用、软引用、弱引用和虚引用。

⑴ 强引用（StrongReference）

强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，Java 虚拟机宁愿抛出 OutOfMemoryError 错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。

```java
Object object = new Object();
String str = "hello";
```

只要某个对象有强引用与之关联，JVM 必定不会回收这个对象，即使在内存不足的情况下，JVM 宁愿抛出 OutOfMemory 错误也不会回收这种对象。比如下面这段代码：

```java
public class Main {
    public static void main(String[] args) {
        new Main().fun1();
    }

    public void fun1() {
        Object object = new Object();
        Object[] objArr = new Object[1000];
    }
}
```

当运行至 Object[] objArr = new Object[1000];这句时，如果内存不足，JVM 会抛出 OOM 错误也不会回收 object 指向的对象。不过要注意的是，当 fun1 运行完之后，object 和 objArr 都已经不存在了，所以它们指向的对象都会被 JVM 回收。

如果想中断强引用和某个对象之间的关联，可以显示地将引用赋值为 null，这样一来的话，JVM 在合适的时间就会回收该对象。

比如 Vector 类的 clear 方法中就是通过将引用赋值为 null 来实现清理工作的：

```java
/**
     * Removes the element at the specified position in this Vector.
     * Shifts any subsequent elements to the left (subtracts one from their
     * indices).  Returns the element that was removed from the Vector.
     *
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= size()})
     * @param index the index of the element to be removed
     * @return element that was removed
     * @since 1.2
     */
    public synchronized E remove(int index) {
    modCount++;
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);
    Object oldValue = elementData[index];

    int numMoved = elementCount - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                 numMoved);
    elementData[--elementCount] = null; // Let gc do its work

    return (E)oldValue;
    }
```

⑵ 软引用（SoftReference）

软引用是用来描述一些有用但并不是必需的对象，在 Java 中用 java.lang.ref.SoftReference 类来表示。对于软引用关联着的对象，只有在内存不足的时候 JVM 才会回收该对象。因此，这一点可以很好地用来解决 OOM 的问题，并且这个特性很适合用来实现缓存：比如网页缓存、图片缓存等。

软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被 JVM 回收，这个软引用就会被加入到与之关联的引用队列中。下面是一个使用示例：

```
import java.lang.ref.SoftReference;

public class Main {
    public static void main(String[] args) {

        SoftReference<String> sr = new SoftReference<String>(new String("hello"));
        System.out.println(sr.get());
    }
}
```

⑶ 弱引用（WeakReference）

弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

```java
import java.lang.ref.WeakReference;

public class Main {
    public static void main(String[] args) {

        WeakReference<String> sr = new WeakReference<String>(new String("hello"));

        System.out.println(sr.get());
        System.gc();                //通知JVM的gc进行垃圾回收
        System.out.println(sr.get());
    }
}
```

输出结果为：

```
hello
null
```

第二个输出结果是 null，这说明只要 JVM 进行垃圾回收，被弱引用关联的对象必定会被回收掉。不过要注意的是，这里所说的被弱引用关联的对象是指只有弱引用与之关联，如果存在强引用同时与之关联，则进行垃圾回收时也不会回收该对象（软引用也是如此）。

弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java 虚拟机就会把这个弱引用加入到与之关联的引用队列中。

⑷ 虚引用（PhantomReference）

“虚引用”顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。

虚引用和前面的软引用、弱引用不同，它并不影响对象的生命周期。在 java 中用 java.lang.ref.PhantomReference 类表示。如果一个对象与虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收。

要注意的是，虚引用必须和引用队列关联使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之 关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

```java
import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;


public class Main {
    public static void main(String[] args) {
        ReferenceQueue<String> queue = new ReferenceQueue<String>();
        PhantomReference<String> pr = new PhantomReference<String>(new String("hello"), queue);
        System.out.println(pr.get());
    }
}
```

#### OOM

前面讲了关于软引用和弱引用相关的基础知识，那么到底如何利用它们来优化程序性能，从而避免 OOM 的问题呢？

下面举个例子，假如有一个应用需要读取大量的本地图片，如果每次读取图片都从硬盘读取，则会严重影响性能，但是如果全部加载到内存当中，又有可能造成内存溢出，此时使用软引用可以解决这个问题。

设计思路是：用一个 HashMap 来保存图片的路径 和 相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM 会自动回收这些缓存图片对象所占用的空间，从而有效地避免了 OOM 的问题。在 Android 开发中对于大量图片下载会经常用到。

```
.....
private Map<String, SoftReference<Bitmap>> imageCache = new HashMap<String, SoftReference<Bitmap>>();
<br>....
public void addBitmapToCache(String path) {

        // 强引用的Bitmap对象

        Bitmap bitmap = BitmapFactory.decodeFile(path);

        // 软引用的Bitmap对象

        SoftReference<Bitmap> softBitmap = new SoftReference<Bitmap>(bitmap);

        // 添加该对象到Map中使其缓存

        imageCache.put(path, softBitmap);

    }

 public Bitmap getBitmapByPath(String path) {

        // 从缓存中取软引用的Bitmap对象

        SoftReference<Bitmap> softBitmap = imageCache.get(path);

        // 判断是否存在软引用

        if (softBitmap == null) {

            return null;

        }

        // 取出Bitmap对象，如果由于内存不足Bitmap被回收，将取得空

        Bitmap bitmap = softBitmap.get();

        return bitmap;

    }
```

### 静态内存分配与回收

### 动态内存分配与回收

### 垃圾检测

### 基于分代的垃圾收集算法

## 内存问题分析

**1、内存溢出**

就是你要求分配的 java 虚拟机内存超出了系统能给你的，系统不能满足需求，于是产生溢出。

**2、内存泄漏**

是指你向系统申请分配内存进行使用(new)，可是使用完了以后却不归还(delete)，结果你申请到的那块内存你自己也不能再访问,该块已分配出来的内存也无法再使用，随着服务器内存的不断消耗，而无法使用的内存越来越多，系统也不能再次将它分配给需要的程序，产生泄露。一直下去，程序也逐渐无内存使用，就会溢出。

### GC 日志分析

### 堆快照文件分析

### JVM Crash 日志分析
