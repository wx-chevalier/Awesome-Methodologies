[![返回目录](https://i.postimg.cc/JzFTMvjF/image.png)](https://github.com/wx-chevalier/Awesome-CheatSheets)

# Go Concurrent Programming CheatSheet | Go 并发编程实践与机制探究

# 协程调度

Golang 简称 Go，Go 的协程(goroutine) 和我们常见的线程(Thread)一样，拥有其调度器。

G (Goroutine)，代表协程，也就是每次代码中使用 go 关键词时候会创建的一个对象
M (Work Thread)，工作线程
P (Processor)，代表一个处理器，又称上下文

每一个运行的 M 都必须绑定一个 P，线程 M 创建后会去检查并执行 G (goroutine)对象
每一个 P 保存着一个协程 G 的队列
除了每个 P 自身保存的 G 的队列外，调度器还拥有一个全局的 G 队列
M 从队列中提取 G，并执行
P 的个数就是 GOMAXPROCS（最大 256），启动时固定的，一般不修改，go 1.5 版本之前的 GOMAXPROCS 默认是 1，go 1.5 版本之后的 GOMAXPROCS 默认是 CPU 的数目。
M 的个数和 P 的个数不一定一样多（会有休眠的 M 或 P 不绑定 M ）（最大 10000）
P 是用一个全局数组（255）来保存的，并且维护着一个全局的 P 空闲链表

![image](https://user-images.githubusercontent.com/5803001/44627050-0c1a4880-a95a-11e8-97f7-28b2be630412.png)

## Goroutine 的入队与执行

入口 main 函数，其实是作为一个 goroutine 来执行，程序启动的时候，首先跑的是主线程，然后这个主线程会绑定第一个 P。

当我们创建一个 G 对象，就是 gorutine，它会加入到本地队列或者全局队列。如果还有空闲的 P，则创建一个 M 绑定该 P；注意，无论在哪个 M 中创建了一个 G，只要 P 有空闲的，就会引起新 M 的创建。新创建的 M 所绑的 P 的初始化队列会从其他 G 队列中取任务过来。

M 会启动一个底层线程，循环执行能找到的 G 任务，其依次从当前 M 所绑的 P 队列中找，去别的 P 的队列中找，去全局 G 队列中找。

协程的切换时间片是 10ms，也就是说 goroutine 最多执行 10ms 就会被 M 切换到下一个 G。这个过程，又被称为 中断，挂起。协程序启动时会首先创建一个特殊的内核线程 sysmon，用来监控和管理，其内部是一个循环：记录所有 P 的 G 任务的计数 schedtick，schedtick 会在每执行一个 G 任务后递增。如果检查到 schedtick 一直没有递增，说明这个 P 一直在执行同一个 G 任务，如果超过 10ms，就在这个 G 任务的栈信息里面加一个 tag 标记。然后这个 G 任务在执行的时候，如果遇到非内联函数调用，就会检查一次这个标记，然后中断自己，把自己加到队列末尾，执行下一个 G。如果没有遇到非内联函数 调用的话，那就会一直执行这个 G 任务，直到它自己结束；如果是个死循环，并且 GOMAXPROCS=1 的话。那么一直只会只有一个 P 与一个 M，且队列中的其他 G 不会被执行！

```go
func main(){
    runtime.GOMAXPROCS(1)
    go func(){
        // 永远不会输出
    	fmt.Println("hello world")
    }()
    go func(){
    	for {

    	}
    }()
    select {}
}
```

中断的时候将寄存器里的栈信息，保存到自己的 G 对象里面当再次轮到自己执行时，将自己保存的栈信息复制到寄存器里面，这样就接着上次之后运行。

# 同步

Channels are concurrency-safe communication objects, used in goroutines.

```go
func main() {
  // A "channel"
  ch := make(chan string)

  // Start concurrent routines
  go push("Moe", ch)
  go push("Larry", ch)
  go push("Curly", ch)

  // Read 3 results
  // (Since our goroutines are concurrent,
  // the order isn't guaranteed!)
  fmt.Println(<-ch, <-ch, <-ch)
}

func push(name string, ch chan string) {
  msg := "Hey, " + name
  ch <- msg
}
```

Buffered channels limit the amount of messages it can keep.

```go
ch := make(chan int, 2)
ch <- 1
ch <- 2
ch <- 3
// fatal error:
// all goroutines are asleep - deadlock!
```
