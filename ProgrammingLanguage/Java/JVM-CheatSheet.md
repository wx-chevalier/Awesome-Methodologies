[![返回目录](https://i.postimg.cc/JzFTMvjF/image.png)](https://github.com/wx-chevalier/Awesome-CheatSheets)

## JVM 内存分配策略

## JVM 内存回收策略

### JVM 引用类型

⑴

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
