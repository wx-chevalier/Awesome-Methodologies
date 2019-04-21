# Java Concurrent Programming CheatSheet

# Concurrent Primitive | 并发单元

常见的 Runnable、Callable、Future、FutureTask 这几个与线程相关的类或者接口：

- Runnable 应该是我们最熟悉的接口，它只有一个 run()函数，用于将耗时操作写在其中，该函数没有返回值。然后使用某个线程去执行该 runnable 即可实现多线程，Thread 类在调用 start()函数后就是执行的是 Runnable 的 run()函数。

- Callable 与 Runnable 的功能大致相似，Callable 中有一个 call()函数，但是 call()函数有返回值，而 Runnable 的 run()函数不能将结果返回给客户程序。

- Executor 就是 Runnable 和 Callable 的调度容器，Future 就是对于具体的 Runnable 或者 Callable 任务的执行结果进行取消、查询是否完成、获取结果、设置结果操作。get 方法会阻塞，直到任务返回结果。

- FutureTask 则是一个 RunnableFuture<V>，而 RunnableFuture 实现了 Runnbale 又实现了 Futrue<V> 这两个接口。

## Threads & Runnables

# Thread Pool | 线程池

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

# Built-in ThreadSafe DataStructure | 内置的线程安全模型
