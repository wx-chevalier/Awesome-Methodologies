[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# JVM 内部架构与运行机制盘点

所谓的内存管理(Memory Management)，即是如何在计算机系统中协调与控制内存的使用，其往往会关注三个层次：MMUs, RAM 这样的硬件内存管理；Virtual Memory, Protection 这样的操作系统级内存管理；以及内存的分配、垃圾回收等应用级的内存管理。

# 体系架构

Java 选择了基于栈的架构往往在单运算的操作次数上会多于基于寄存器的架构，而 Google 提出的 Android 操作系统设计的 Dalvik 虚拟机则是采用了基于寄存器的架构。

![image](https://user-images.githubusercontent.com/5803001/44001064-640aa6f0-9e5d-11e8-9990-8ec1fcad6666.png)

每当创建一个新的线程时，JVM 会为该线程创建一个 Java 栈，同时会为这个线程分配一个 PC 寄存器，并且这个 PC 寄存器会指向这个线程的第一行可执行代码。每当调用一个新方法时会在这个栈上创建一个新的栈帧数据结构，这个栈帧会保留这个方法的一些元信息，如在这个方法中定义的局部变量、一些用来支持常量池的解析、正常方法返回以及异常处理机制等等。

| 内存区域       | 存放内容                                                                                                                                                   | 大小                                                                                              |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| 堆             | 所有的对象实例和数组                                                                                                                                       | 根据虚拟机规范的规定，Java 堆可以是固定的大小也可以是按照需求动态扩展的，而且不需要保证是连续的。 |
| 方法区         | 类的结构信息，如类的字段、方法、接口、构造函数，还有运行时常量池等                                                                                         |                                                                                                   |
| 程序计数寄存器 | 如果线程执行的是一个 Java 方法，那么寄存器里面记录的就是正在执行的虚拟机字节码指令的地址，如果线程执行的是一个 native 方法，那么寄存器记录的值为 undefined | 虚拟机规范里面唯一一个没有规定任何 OutOfMemoryError 情况的区域                                    |
| 虚拟机栈       | 局部变量表、操作数栈、方法出口等信息。                                                                                                                     | 局部变量表存放了编译时期可知的各种基本数据类型、对象引用和指向了一条字节码指令的地址              |
| 本地方法栈     | 局部变量表、操作数栈、方法出口等信息                                                                                                                       |

如果线程清求的栈深度大 于虚拟机所允许的深度，将抛出 StackOverflowError 异常；如果虚拟机栈可以动态扩展 (当前大部分的 Java 虚拟机都可动态扩展，只不过 Java 虚拟机规范中也允许固定长度的 虚拟机栈)，当扩展时无法申请到足够的内存时会拋出 OutOfMemoryError 异常。

局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的。在方法运行期间不会改变局部变量表的大小。

一个 JVM 栈由多个帧组成，当一个方法被调用的时候，会 push 一个帧到栈顶，当方法执行完毕时（正常返回或者抛出异常），一个帧会从栈顶弹出。每个帧由两部分组成：

- 一个数组，用于存放本地变量，数组长度由编译器计算确定，一个局部变量可用于存储任意类型的值， long 和 double 值除外，它们需要两个局部变量；
- 一个操作栈，用于存放中间值，可作为指令的操作数，或者作为方法调用的参数。

# 字节码

```java
public static void main(String[] args) {
    int a = 1;
    int b = 2;
    int c = a + b;
}

// javac Test.java
// javap -v Test.class
```

```java
// main 方法的签名
public static void main(java.lang.String[]);

// 第二部分的 descriptor 表示方法拥有一个类型为 [Ljava/lang/String; 的参数，返回值类型是 V
descriptor: ([Ljava/lang/String;)V

// 一系列指示符，ACC_PUBLIC 表明方法是 public 类型， ACC_STATIC 表明方法是 static 类型
flags: (0x0009) ACC_PUBLIC, ACC_STATIC

// 代码区
Code:

// stack 表示操作栈的最大深度，locals 表示本地变量数组的长度，args_size 表示参数的个数。在指令执行过程中，所有局部变量会陆续被操作，但 args 除外，它固定放在本地变量数组索引等于 0 的位置。
stack=2, locals=4, args_size=1

// 常量 1 推入操作栈
0: iconst_1

// 从操作栈弹出一个 int 值，存入索引为 1 的本地变量中，对应源码中的变量 a
1: istore_1

// 将常量 2 推入操作栈
2: iconst_2

// 从操作栈弹出一个 int 值，存入索引为 2 的本地变量中，对应源码中的变量 b
3: istore_2

// 从索引为 1 的本地变量中取出 int 值，推入操作栈
4: iload_1

// 从索引为 2 的本地变量中取出 int 值，推入操作栈
5: iload_2

// 从操作栈弹出两个 int 值，然后相加，并将结果推入操作栈
6: iadd

// 从操作栈弹出 int 值，存入索引为 3 的本地变量中，对应源码中的变量c
7: istore_3

8: return

...
```

![image](https://user-images.githubusercontent.com/5803001/44001160-0d9cf780-9e5f-11e8-952f-6fa1fb29c0fe.png)

## 数据类型

| 数据类型                                | 描述                                                                                                                                                                                                                                               |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Integer                                 | 4 字节常量                                                                                                                                                                                                                                         |
| Long                                    | 8 字节常量                                                                                                                                                                                                                                         |
| Float                                   | 4 字节常量                                                                                                                                                                                                                                         |
| Double                                  | 8 字节常量                                                                                                                                                                                                                                         |
| String                                  | 字符串常量指向常量池的另外一个包含真正字节 Utf8 编码的实体                                                                                                                                                                                         |
| Utf8                                    | Utf8 编码的字符序列字节流                                                                                                                                                                                                                          |
| Class                                   | 一个 Class 常量，指向常量池的另一个 Utf8 实体，这个实体包含了符合 JVM 内部格式的类的全名(动态链接过程需要用到)                                                                                                                                     |
| NameAndType                             | 冒号(:)分隔的一组值，这些值都指向常量池中的其它实体。第一个值(“:”之前的)指向一个 Utf8 字符串实体，它是一个方法名或者字段名。第二个值指向表示类型的 Utf8 实体。对于字段类型，这个值是类的全名，对于方法类型，这个值是每个参数类型类的类全名的列表。 |
| Fieldref, Methodref, InterfaceMethodref | 点号(.)分隔的一组值，每个值都指向常量池中的其它的实体。第一个值(“.”号之前的)指向类实体，第二个值指向 NameAndType 实体。                                                                                                                            |

# GC 垃圾回收

在 JDK10 的代码中，路径为 openjdk/src/hotspot/share/gc/，各个 GC 实现共享依赖 shared 代码，GC 包括目前默认的 G1，也有经典的 Serial、Parallel、CMS 等 GC 实现。

## JVM 体系结构

## 基于栈的架构

## 执行引擎

每当创建一个新的线程时，JVM 会为该线程创建一个 Java 栈，同时会为这个线程分配一个 PC 寄存器，并且这个 PC 寄存器会指向这个线程的第一行可执行代码。每当调用一个新方法时会在这个栈上创建一个新的栈帧数据结构，这个栈帧会保留这个方法的一些元信息，如在这个方法中定义的局部变量、一些用来支持常量池的解析、正常方法返回以及异常处理机制等等。

![](http://hi.csdn.net/attachment/201009/25/0_1285381395C6iW.gif)

# JVM 内存结构

Java 虚拟机会将内存分为几个不同的管理区，这些区域各自有各自的用途，根据不同的特点，承担不同的任务以及在垃圾回收时运用不同的算法。总体分为下面几个部分：程序计数器(Program Counter Register)、JVM 虚拟机栈(JVM Stacks)、本地方法栈(Native Method Stacks)、堆(Heap)、方法区(Method Area)

**1、程序计数器(Program Counter Register)**

这是一块比较小的内存，不在 Ram 上，而是直接划分在 CPU 上的，程序员无法直接操作它，它的作用是：JVM 在解释字节码文件(.class)时，存储当前线程所执行的字节码的行号，只是一种概念模型，各种 JVM 所采用的方式不同，字节码解释器工作时，就是通过改变程序计数器的值来选取下一条要执行的指令，分支、循环、跳转、等基础功能都是依赖此技术区完成的。还有一种情况，就是我们常说的 Java 多线程方面的，多线程就是通过现程轮流切换而达到的，同一时刻，一个内核只能执行一个指令，所以，对于每一个程序来说，必须有一个计数器来记录程序的执行进度，这样，当线程恢复执行的时候，才能从正确的地方开始，所以，每个线程都必须有一个独立的程序计数器，这类计数器为线程私有的内存。如果一个线程正在执行一个 Java 方法，则计数器记录的是字节码的指令的地址，如果执行的一个 Native 方法，则计数器的记录为空，此内存区是唯一一个在 Java 规范中没有任何 OutOfMemoryError 情况的区域。

**2、JVM 虚拟机栈(JVM Stacks)**

JVM 虚拟机栈就是我们常说的堆栈的栈(我们常常把内存粗略分为堆和栈)，和程序计数器一样，也是线程私有的，生命周期和线程一样，每个方法被执行的时候会产生一个**栈帧**，用于存储局部变量表、动态链接、操作数、方法出口等信息。方法的执行过程就是栈帧在 JVM 中出栈和入栈的过程。局部变量表中存放的是各种基本数据类型，如 boolean、byte、char、等 8 种，及引用类型(存放的是指向各个对象的内存地址)，因此，它有一个特点：内存空间可以在编译期间就确定，运行期不在改变。这个内存区域会有两种可能的 Java 异常：StackOverFlowError 和 OutOfMemoryError。

**3、本地方法栈(Native Method Stacks)**

从名字即可看出，本地方法栈就是用来处理 Java 中的本地方法的，Java 类的祖先类 Object 中有众多 Native 方法，如 hashCode()、wait()等，他们的执行很多时候是借助于操作系统，但是 JVM 需要对他们做一些规范，来处理他们的执行过程。此区域，可以有不同的实现方法，向我们常用的 Sun 的 JVM 就是本地方法栈和 JVM 虚拟机栈是同一个。

**4、堆(Heap)**

堆内存是内存中最重要的一块，也是最有必要进行深究的一部分。因为 Java 性能的优化，主要就是针对这部分内存的。所有的对象实例及数组都是在堆上面分配的(随着 JIT 技术的逐渐成熟，这句话视乎有些绝对，不过至少目前还基本是这样的)，可通过-Xmx 和-Xms 来控制堆的大小。JIT 技术的发展产生了新的技术，如栈上分配和标量替换，也许在不久的几年里，即时编译会诞生及成熟，那个时候，“所有的对象实例及数组都是在堆上面分配的”这句话就应该稍微改改了。堆内存是垃圾回收的主要区域，所以在下文垃圾回收板块会重点介绍，此处只做概念方面的解释。在 32 位系统上最大为 2G，64 位系统上无限制。可通过-Xms 和-Xmx 控制，-Xms 为 JVM 启动时申请的最小 Heap 内存，-Xmx 为 JVM 可申请的最大 Heap 内存。

**5、方法区(Method Area)**

方法区是所有线程共享的内存区域，用于存储已经被 JVM 加载的类信息、常量、静态变量等数据，一般来说，方法区属于持久代(关于持久代，会在 GC 部分详细介绍，除了持久代，还有新生代和旧生代)，也难怪 Java 规范将方法区描述为堆的一个逻辑部分，但是它不是堆。方法区的垃圾回收比较棘手，就算是 Sun 的 HotSpot VM 在这方面也没有做得多么完美。此处引入方法区中一个重要的概念：运行时常量池。主要用于存放在编译过程中产生的字面量(字面量简单理解就是常量)和引用。一般情况，常量的内存分配在编译期间就能确定，但不一定全是，有一些可能就是运行时也可将常量放入常量池中，如 String 类中有个 Native 方法 intern()<关于intern()的详细说明，请看另一篇文章：[http://blog.csdn.net/zhangerqing/article/details/8093919](http://blog.csdn.net/zhangerqing/article/details/8093919)>

此处补充一个在 JVM 内存管理之外的一个内存区：直接内存。在 JDK1.4 中新加入类 NIO 类，引入了一种基于通道与缓冲区的 I/O 方式，它可以使用 Native 函数库直接分配堆外内存，即我们所说的直接内存，这样在某些场景中会提高程序的性能。

## JVM 内存分配策略

## JVM 内存回收策略

### JVM 引用类型

在 JDK 1.2 以前的版本中，若一个对象不被任何变量引用，那么程序就无法再使用这个对象。也就是说，只有对象处于可触及(reachable)状态，程序才能使用它。从 JDK 1.2 版本开始，把对象的引用分为 4 种级别，从而使程序能更加灵活地控制对象的生命周期。这 4 种级别由高到低依次为：强引用、软引用、弱引用和虚引用。

⑴ 强引用(StrongReference)

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

⑵ 软引用(SoftReference)

软引用是用来描述一些有用但并不是必需的对象，在 Java 中用 java.lang.ref.SoftReference 类来表示。对于软引用关联着的对象，只有在内存不足的时候 JVM 才会回收该对象。因此，这一点可以很好地用来解决 OOM 的问题，并且这个特性很适合用来实现缓存：比如网页缓存、图片缓存等。

软引用可以和一个引用队列(ReferenceQueue)联合使用，如果软引用所引用的对象被 JVM 回收，这个软引用就会被加入到与之关联的引用队列中。下面是一个使用示例：

```
import java.lang.ref.SoftReference;

public class Main {
    public static void main(String[] args) {

        SoftReference<String> sr = new SoftReference<String>(new String("hello"));
        System.out.println(sr.get());
    }
}
```

⑶ 弱引用(WeakReference)

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

第二个输出结果是 null，这说明只要 JVM 进行垃圾回收，被弱引用关联的对象必定会被回收掉。不过要注意的是，这里所说的被弱引用关联的对象是指只有弱引用与之关联，如果存在强引用同时与之关联，则进行垃圾回收时也不会回收该对象(软引用也是如此)。

弱引用可以和一个引用队列(ReferenceQueue)联合使用，如果弱引用所引用的对象被垃圾回收，Java 虚拟机就会把这个弱引用加入到与之关联的引用队列中。

⑷ 虚引用(PhantomReference)

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

```java
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

# JVM 内存分配

## 静态分配与动态分配

内存的静态分配和动态分配的区别主要是两个：一是时间不同。静态分配发生在程序编译和连接的时候。动态分配则发生在程序调入和执行的时候。

二是空间不同。堆都是动态分配的，没有静态分配的堆。栈有 2 种分配方式：静态分配和动态分配。静态分配是编译器完成的，比如局部变量的分配。动态分配由函数 malloc 进行分配。不过栈的动态分配和堆不同，他的动态分配是由编译器进行释放，无需我们手工实现。

对于一个进程的内存空间而言，可以在逻辑上分成 3 个部份：代码区，静态数据区和动态数据区。动态数据区一般就是“堆栈”。“栈(stack)”和“堆(heap)”是两种不同的动态数据区，栈是一种线性结构，堆是一种链式结构。进程的每个线程都有私有的“栈”，所以每个线程虽然代码一样，但本地变量的数据都是互不干扰。一个堆栈可以通过“基地址”和“栈顶”地址来描述。全局变量和静态变量分配在静态数据区，本地变量分配在动态数据区，即堆栈中。程序通过堆栈的基地址和偏移量来访问本地变量。

一般，用 static 修饰的变量，全局变量位于静态数据区。函数调用过程中的参数，返回地址，EBP 和局部变量都采用栈的方式存放。

# 垃圾收集器

## ZGC

与标记对象的传统算法相比，ZGC 在指针上做标记，在访问指针时加入 Load Barrier（读屏障），比如当对象正被 GC 移动，指针上的颜色就会不对，这个屏障就会先把指针更新为有效地址再返回，也就是，永远只有单个对象读取时有概率被减速，而不存在为了保持应用与 GC 一致而粗暴整体的 Stop The World。

# JIT

## 分层编译

随着代码的执行，JVM 的 JIT 编译器会将部分热点代码编译为目标机器代码；JVM 提供了一个参数 `-Xcomp` ,这个参数可以使 JVM 运行在纯编译的模式，所有的方法在第一次调用的时候就会编成机器代码，但是设置了这个参数之后系统启动负载的确没有上升，但是启动的时间是原来的两倍多。

除了纯编译和默认的 mixed 之外，jvm 从 jdk6u25 之后，引入了分层编译(-XX:+TieredCompilation)。HotSpot 内置两种编译器，分别是 client 启动时的 c1 编译器和 server 启动时的 c2 编译器，c2 在将代码编译成机器代码的时候需要搜集大量的统计信息以便在编译的时候进行优化，因此编译出来的代码执行效率比较高，代价是程序启动时间比较长，而且需要执行比较长的时间，才能达到最高性能；与之相反， c1 的目标是使程序尽快进入编译执行的阶段，所以在编译前需要搜集的信息比 c2 要少，编译速度因此提高很多，但是付出的代价是编译之后的代码执行效率比较低，但尽管如此，c1 编译出来的代码在性能上比解释执行的性能已经有很大的提升，所以所谓的分层编译，就是一种折中方式，在系统执行初期，执行频率比较高的代码先被 c1 编译器编译，以便尽快进入编译执行，然后随着时间的推移，执行频率较高的代码再被 c2 编译器编译，以达到最高的性能。

```cpp
// globalDefinitions.hpp
enum CompLevel {
  CompLevel_any               = -1,
  CompLevel_all                = -1,
  CompLevel_none               = 0,         // Interpreter
  CompLevel_simple             = 1,    // C1
  CompLevel_limited_profile  = 2,   // C1, invocation & backedge counters
  CompLevel_full_profile      = 3,   // C1, invocation & backedge counters + mdo
  CompLevel_full_optimization = 4,  // C2 or Shark
  ... ...
};
```

目前分层编译包括五个编译层次：Level 0 - Level 4。大家可以在启动参数中加入-XX:+PrintCompilation 来查看被编译的方法及其层次：

- Level0 即解释执行，由解释器负责执行 java 方法。这种方式无编译开销，但速度很慢；解释器并不开启性能监控功能，可触发第一层编译。

- Level1 程序由 C1 编译器编译为本地机器指令执行，C1 编译器会对字节码进行简单和可靠的优化，以达到更快的编译速度。编译方法受限，编译开销比较低，性能比 Level4 差，但强于其他层次。

- Level2 程序由 C1 编译器编译为本地机器指令执行，但 C2 编译器会启动一些编译耗时更长的优化（代码将有可能被重复编译多次），甚至有可能根据性能监控信息进行一些不可靠的激进优化。编译开销比较低，性能差于 Level4 和 Level1。

- Level3 由 C1 负责编译，除了方法执行次数和回边次数的统计外，还加入了对方法内部执行信息的统计，如一个分枝是否执行跳转，一个虚函数调用最终调用到哪个方法等信息。Level3 性能较差，仅比 Level0 快，但是 Level3 是 Level4 编译的必要步骤。程序由 C1 编译器编译为本地机器指令执行，采集性能数据进行优化措施；

- Level4 由 C2 负责编译，它利用 level3 收集的信息，对方法进行完全的优化，性能最好，但是编译开销也最大。程序由 C2 编译器编译为本地机器指令执行，进行完全优化。

下述列举了判断一个方法是否需要触发编译的及编译到哪个层次的具体公式:

| 参数                     | 说明                                                       | 默认值 |
| ------------------------ | ---------------------------------------------------------- | ------ |
| Tier3InvocationThreshold | 一个方法被调用多少次之后会进行 level3 编译                 | 200    |
| Tier4InvocationThreshold | 一个方法被调用多少次之后会进行 level4 编译                 | 5000   |
| Tier3CompileThreshold    | 考虑回边的情况下，一个方法执行多少次之后会进行 level3 编译 | 2000   |
| Tier4CompileThreshold    | 考虑回边的情况下，一个方法执行多少次之后会进行 level4 编译 | 15000  |
| Tier3BackEdgeThreshold   | 一个方法中回边执行多少次会进行 OSR level3 编译             | 60000  |
| Tier4BackEdgeThreshold   | 一个方法中回边执行多少次会进行 OSR level4 编译             | 40000  |
| CICompilerCount          | 编译线程数目                                               | 无     |
| Tier3LoadFeedback        | 用来动态调整 level3 编译阈值的值                           | 5      |
| Tier4LoadFeedback        | 用来动态调整 level4 编译阈值的值                           | 3      |
| Tier3DelayOn             | 平均每个 C2 编译线程排队个数达到多少个时停止 level3 编译   | 5      |
| Tier3DelayOff            | 平均每个 C2 编译线程排队个数降低到多少个时恢复 level3 编译 | 2      |

最常见的 0 -> 3 -> 4 编译，高 C2 编译压力下的 0 -> 2 -> 3 -> 4 编译；level2 的性能比 level3 好。当 C2 的编译压力很大的情况下，新编译的 level3 方法可能方法会长时间的运行在效率相对比较低的代码上而不能及时晋升到 level4。在这种情况下，将待编译的方法编译成 level2 而不是 level3 会是一个更好的方法。总体上，C2 编译压力比较大的情况下是先解释执行，达到一定次数编译成 level2，然后等 C2 编译压力变小的时候会晋升成 level3，最后如果执行次数足够多的话会晋升成 level4。

Level1 与 level2 和 level3 不同，它是一个不收集运行数据的最终编译状态。
当一个方法需要进行编译的时候，我们会首先判断它是否符合以下条件：

1. 这个方法只是用来获取类中的某个域的值的
2. 这个方法只是用来获取常量的
3. 这个方法很小

满足这三个条件其中之一的方法会直接被编译成 level1 并且不会晋升为其他 level。由于 C2 编译耗时较多，往往要达到几百毫秒甚至超过一秒，因此在高峰期行 Level4 编译通常并不划算，因此 JVM 参数的调整的思路是增大 level4 阈值，减少 level4 编译。具体的，可以通过增大 Tier4InvocationThreshold 和 Tier4CompileThreshold 来增大阈值。编译开销主要在于 C2 的编译线程，因此通过一定的手段限制 C2 编译线程的 CPU 使用可以减少高峰期编译开销。在本次测试中，使用在每编译完成一个方法后，sleep 200ms 的方法来限制 C2 编译线程 CPU 使用率。

# Class

## Class 文件结构

## 类加载

JVM 提供了 3 种类加载器： BootstrapClassLoader、 ExtClassLoader、 AppClassLoader 分别加载 Java 核心类库、扩展类库以及应用的类路径( CLASSPATH)下的类库。JVM 通过双亲委派模型进行类的加载，我们也可以通过继承 java.lang.classloader 实现自己的类加载器。何为双亲委派模型？当一个类加载器收到类加载任务时，会先交给自己的父加载器去完成，因此最终加载任务都会传递到最顶层的 BootstrapClassLoader，只有当父加载器无法完成加载任务时，才会尝试自己来加载。

采用双亲委派模型的一个好处是保证使用不同类加载器最终得到的都是同一个对象，这样就可以保证 Java 核心库的类型安全，比如，加载位于 rt.jar 包中的 java.lang.Object 类，不管是哪个加载器加载这个类，最终都是委托给顶层的 BootstrapClassLoader 来加载的，这样就可以保证任何的类加载器最终得到的都是同样一个 Object 对象。

```java
protected Class<?> loadClass(String name, boolean resolve) {

    synchronized (getClassLoadingLock(name)) {

    // 首先，检查该类是否已经被加载，如果从JVM缓存中找到该类，则直接返回
    Class<?> c = findLoadedClass(name);

    if (c == null) {
    try {

        // 遵循双亲委派的模型，首先会通过递归从父加载器开始找，

        // 直到父类加载器是BootstrapClassLoader为止

        if (parent != null) {

        c = parent.loadClass(name, false);

        } else {

        c = findBootstrapClassOrNull(name);

        }

    } catch (ClassNotFoundException e) {}

        if (c == null) {

        // 如果还找不到，尝试通过findClass方法去寻找

        // findClass是留给开发者自己实现的，也就是说

        // 自定义类加载器时，重写此方法即可

        c = findClass(name);

        }

    }

    if (resolve) {

        resolveClass(c);

    }

    return c;

    }

}
```
