[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Backend DevOps CheatSheet

本文囊括了日常的持续集成、应用部署、服务器维护等方面的命令与技巧，相较于 [基础命令/DevOps/安全加固/Shell 编程](./Linux-Commands-CheatSheet.md) 等相关清单，首先介绍了如何查看服务器的状态与关键指标，然后介绍了指标的含义、评判标准，最后从进程、存储、网络等细分角度进行展开。

# Metrics | 指标

吞吐量(Throughout)与时延(Latency)是衡量软件系统的最常见的两个指标，系统的吞度量（承压能力）与请求对 CPU 的消耗、外部接口、IO 等等紧密关联；单个请求对 CPU 消耗越高，外部系统接口、IO 影响速度越慢，系统吞吐能力越低，反之越高。

从客户端来看，延迟就是从发送请求到接收响应的整体耗时，包括：请求的网络耗时，请求在服务端的处理耗时以及响应的网络耗时。吞吐量则是服务在一定的并发下，每秒可以处理的请求数。延迟和吞吐天生是矛盾的。对于服务来说，请求的处理是一个排队系统，且排队可能发生在请求路径上的任何环节，比如：请求的 TCP 包在路由器排队或者是在服务器的接收缓冲区等等。当吞吐量增加，那么必然会导致同一时间请求并发的增加。由于资源的限制，同一时刻可以处理的请求数是固定的，取决于整个请求处理过程中最小的那个环节。当并发请求数大于这个值时，就会有请求排队等待被处理。所以，要提升服务的吞吐量，必定会增加整体延迟。另外，如果服务的延迟（单个请求的耗时）减少，由于排队的请求的等待时间也减少了，所以吞吐量会上升。

## 系统指标

MTTF, Mean Time To Failure，系统平均运行多长时间才发生故障，越长越好

MTTR,Mean Time To Recover, 故障平均修复时间，越短越好

可用性计算公式， Availability= MTTF /（MTTF+MTTR）

CPU 利用率是业务系统利用到 CPU 的比率，因为往往一个系统上会有一些其他的线程，这些线程会和 CPU 竞争计算资源，那么此时留给业务的计算资源比例就会下降，典型的像，GC 线程的 GC 过程、锁的竞争过程都是消耗 CPU 的过程。甚至一些 IO 的瓶颈，也会导致 CPU 利用率下降(CPU 都在 Wait IO，利用率当然不高)。

## 访问量指标

PV    即 page view，页面浏览量          用户每一次对网站中的每个页面访问均被记录 1 次。用户对同一页面的多次刷新，访问量累计。

UV  即 Unique visitor，独立访客      通过客户端的 cookies 实现。即同一页面，客户端多次点击只计算一次，访问量不累计。

IP    即 Internet Protocol，本意本是指网络协议，在数据统计这块指通过 ip 的访问量。     即同一页面，客户端使用同一个 IP 访问多次只计算一次，访问量不累计。

## 性能指标

### QPS & TPS & RPS

并发连接数（The number of concurrent connections）

概念：某个时刻服务器所接受的请求数目，简单的讲，就是一个会话。

并发用户数（The number of concurrent users，Concurrency Level）

概念：要注意区分这个概念和并发连接数之间的区别，一个用户可能同时会产生多个会话，也即连接数。

QPS(Queries Per Second)每秒能处理查询数目。是一台服务器每秒能够相应的查询次数，是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准。QPS 是每秒钟处理完请求的次数。这里的请求不是指一个查询或者数据库查询，是包括一个业务逻辑的整个流程，也就是说每秒钟响应的请求次数。

TPS(Transactions Per Second)指每秒处理的事务数目；事务是指一个客户机向服务器发送请求然后服务器做出反应的过程。TPS 的过程包括：客户端请求服务端、服务端内部处理、服务端返回客户端，客户机在发送请求时开始计时，收到服务器响应后结束计时，以此来计算使用的时间和完成的事务个数，最终利用这些信息作出的评估分。

RPS(Requests Per Second) | 吞吐率: 服务器并发处理能力的量化描述，单位是 reqs/s，指的是某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。

### RT | 响应时间

响应时间即 RT，处理一次请求所需要的平均处理时间。对于 RT，客户端和服务端是大不相同的，因为请求从客户端到服务端，需要经过广域网，所以客户端 RT 往往远大于服务端 RT，同时客户端的 RT 往往决定着用户的真实体验，服务端 RT 往往是评估我们系统好坏的一个关键因素。

假设我们的服务端只有一个线程，那么所有的请求都是串行执行，我们可以很简单的算出系统的 QPS，也就是：QPS = 1000ms/RT。假设一个 RT 过程中 CPU 计算的时间为 49ms，CPU Wait Time 为 200ms，那么 QPS 就为 1000/49+200 = 4.01。CPU Time 就是一次请求中，实际用到计算资源。CPU Time 的消耗是全流程的，涉及到请求到应用服务器，再从应用服务器返回的全过程。实际上这取决于你的计算的复杂度。

CPU Wait Time 是一次请求过程中对于 IO 的操作，CPU 这段时间可以理解为空闲的，那么此时要尽量利用这些空闲时间，也就是增加线程数。

CPU 利用率是业务系统利用到 CPU 的比率，因为往往一个系统上会有一些其他的线程，这些线程会和 CPU 竞争计算资源，那么此时留给业务的计算资源比例就会下降，典型的像，GC 线程的 GC 过程、锁的竞争过程都是消耗 CPU 的过程。甚至一些 IO 的瓶颈，也会导致 CPU 利用率下降(CPU 都在 Wait IO，利用率当然不高)。

用户平均请求等待时间（Time per request）

计算公式：处理完成所有请求数所花费的时间/ （总请求数 / 并发用户数），即
Time per request = Time taken for tests /（ Complete requests / Concurrency Level）

服务器平均请求等待时间（Time per request: across all concurrent requests）
计算公式：处理完成所有请求数所花费的时间 / 总请求数，即
Time taken for / testsComplete requests
可以看到，它是吞吐率的倒数。
同时，它也=用户平均请求等待时间/并发用户数，即
Time per request / Concurrency Level

➊ 在命令行输入 top，然后 shift+p 查看占用 CPU 最高的进程，记下进程号
➋ 在命令行输入 top -Hp 进程号，查看占用 CPU 最高的线程
➌ 使用 printf 0x%x 线程号，得到其 16 进制线程号
➍ 使用 jstack 进程号得到 java 执行栈，然后 grep16 进制找到相应的信息

ps -eo %cpu,pid |sort -n -k1 -r | head -n 1 | awk '{print $2}' |xargs  top -b -n1 -Hp | grep COMMAND -A1 | tail -n 1 | awk '{print $1}' | xargs printf 0x%x

找到使用 CPU 最高的进程之使用 CPU 最高的线程的 16 进制号。

➊ 如果 load 超过了 cpu 核数，则负载过高
➋ 如果 wa 过高，可初步判断 I/O 有问题
➌ sy,si,hi,st，任何一个超过 5%，都有问题
➍ 进程状态长时处于 D、Z、T 状态，提高注意度
➎ cpu 不均衡，判断亲和性和优先级问题 ➊ 如果 load 超过了 cpu 核数，则负载过高
➋ 如果 wa 过高，可初步判断 I/O 有问题
➌ sy,si,hi,st，任何一个超过 5%，都有问题
➍ 进程状态长时处于 D、Z、T 状态，提高注意度
➎ cpu 不均衡，判断亲和性和优先级问题

除了关注类似 top 的一些指标，还有：
➊ b 置于等待队列（等待资源、等待输入/输出）的内核线程数目。数字过大则 cpu 太忙。
➋ cs 如果频繁的进行上下文切换，则考虑是否是线程数开的过多
➌ si/so 显示了交换分区的现状，有时候会造成 cpu 问题，一并关注

除了关注类似 top 的一些指标，还有：
➊ b 置于等待队列（等待资源、等待输入/输出）的内核线程数目。数字过大则 cpu 太忙。
➋ cs 如果频繁的进行上下文切换，则考虑是否是线程数开的过多
➌ si/so 显示了交换分区的现状，有时候会造成 cpu 问题，一并关注

sar

是目前 Linux 上最为全面的系统性能分析工具之一，但可能没有预装。在 centos 上使用以下命令即可安装。

yum install sysstat -y
sar 主要的好处是可以看到历史，显示友好，可以对结果进行二次处理。sar 还有图形化工具，执行 sar -A 即可获得所有数据。

https://github.com/vlsi/ksar
针对于 CPU 方面，我们关注：
➊ sar -u 默认
➋ sar -P ALL 每颗 cpu 的使用状态信息
➌ sar -q cpu 队列的长度，runq-sz>cpu count 就表明有瓶颈了
➍ sar -w 每秒上下文交换

mpstat

还有 pidstat，包括彩色的 dstat，功能都差不多

load 为 1 代表的是啥

针对这个问题，误解还是比较多的。很多同学认为，load 达到 1，系统就到了瓶颈，这不完全正确。
load 的值和 cpu 核数息息相关：
➊ 单核的 cpu 达到 100%，load 约 1
➋ 双核的 cpu 都达到 100%，load 约 2
➌ 四核的 cpu 都达到 100%，load 约为 4

Linux 进程状态：R (TASK_RUNNING)，可执行状态。

只有在该状态的进程才可能在 CPU 上运行。而同一时刻可能有多个进程处于可执行状态，这些进程的 task_struct 结构（进程控制块）被放入对应 CPU 的可执行队列中（一个进程最多只能出现在一个 CPU 的可执行队列中）。进程调度器的任务就是从各个 CPU 的可执行队列中分别选择一个进程在该 CPU 上运行。

Linux 进程状态：S (TASK_INTERRUPTIBLE)，可中断的睡眠状态。

处于这个状态的进程因为等待某某事件的发生（比如等待 socket 连接、等待信号量），而被挂起。这些进程的 task_struct 结构被放入对应事件的等待队列中。当这些事件发生时（由外部中断触发、或由其他进程触发），对应的等待队列中的一个或多个进程将被唤醒。
通过 ps 命令我们会看到，一般情况下，进程列表中的绝大多数进程都处于 TASK_INTERRUPTIBLE 状态（除非机器的负载很高）。

D
Linux 进程状态：D (TASK_UNINTERRUPTIBLE)，不可中断的睡眠状态。

与 TASK_INTERRUPTIBLE 状态类似，进程处于睡眠状态，但是此刻进程是不可中断的。不可中断，指的并不是 CPU 不响应外部硬件的中断，而是指进程不响应异步信号。TASK_UNINTERRUPTIBLE 状态存在的意义就在于，内核的某些处理流程是不能被打断的。如果响应异步信号，程序的执行流程中就会被插入一段用于处理异步信号的流程（这个插入的流程可能只存在于内核态，也可能延伸到用户态）在进程对某些硬件进行操作时（比如进程调用 read 系统调用对某个设备文件进行读操作，而 read 系统调用最终执行到对应设备驱动的代码，并与对应的物理设备进行交互），可能需要使用 TASK_UNINTERRUPTIBLE 状态对进程进行保护，以避免进程与设备交互的过程被打断，造成设备陷入不可控的状态。这种情况下的 TASK_UNINTERRUPTIBLE 状态总是非常短暂的，通过 ps 命令基本上不可能捕捉到。

Linux 进程状态：T (TASK_STOPPED or TASK_TRACED)，暂停状态或跟踪状态。

向进程发送一个 SIGSTOP 信号，它就会因响应该信号而进入 TASK_STOPPED 状态（除非该进程本身处于 TASK_UNINTERRUPTIBLE 状态而不响应信号）。（SIGSTOP 与 SIGKILL 信号一样，是非常强制的。不允许用户进程通过 signal 系列的系统调用重新设置对应的信号处理函数。）
向进程发送一个 SIGCONT 信号，可以让其从 TASK_STOPPED 状态恢复到 TASK_RUNNING 状态。
当进程正在被跟踪时，它处于 TASK_TRACED 这个特殊的状态。“正在被跟踪”指的是进程暂停下来，等待跟踪它的进程对它进行操作。比如在 gdb 中对被跟踪的进程下一个断点，进程在断点处停下来的时候就处于 TASK_TRACED 状态。而在其他时候，被跟踪的进程还是处于前面提到的那些状态。
对于进程本身来说，TASK_STOPPED 和 TASK_TRACED 状态很类似，都是表示进程暂停下来。

Linux 进程状态：Z (TASK_DEAD – EXIT_ZOMBIE)，退出状态，进程成为僵尸进程。

进程在退出的过程中，处于 TASK_DEAD 状态。
在这个退出过程中，进程占有的所有资源将被回收，除了 task_struct 结构（以及少数资源）以外。于是进程就只剩下 task_struct 这么个空壳，故称为僵尸。
之所以保留 task_struct，是因为 task_struct 里面保存了进程的退出码、以及一些统计信息。而其父进程很可能会关心这些信息。比如在 shell 中，\$?变量就保存了最后一个退出的前台进程的退出码，而这个退出码往往被作为 if 语句的判断条件。
当然，内核也可以将这些信息保存在别的地方，而将 task_struct 结构释放掉，以节省一些空间。但是使用 task_struct 结构更为方便，因为在内核中已经建立了从 pid 到 task_struct 查找关系，还有进程间的父子关系。释放掉 task_struct，则需要建立一些新的数据结构，以便让父进程找到它的子进程的退出信息。
父进程可以通过 wait 系列的系统调用（如 wait4、waitid）来等待某个或某些子进程的退出，并获取它的退出信息。然后 wait 系列的系统调用会顺便将子进程的尸体（task_struct）也释放掉。
子进程在退出的过程中，内核会给其父进程发送一个信号，通知父进程来“收尸”。这个信号默认是 SIGCHLD，但是在通过 clone 系统调用创建子进程时，可以设置这个信号。

当进程退出的时候，会将它的所有子进程都托管给别的进程（使之成为别的进程的子进程）。托管给谁呢？可能是退出进程所在进程组的下一个进程（如果存在的话），或者是 1 号进程。所以每个进程、每时每刻都有父进程存在。除非它是 1 号进程。
1 号进程，pid 为 1 的进程，又称 init 进程。
linux 系统启动后，第一个被创建的用户态进程就是 init 进程。它有两项使命：

执行系统初始化脚本，创建一系列的进程（它们都是 init 进程的子孙）；
在一个死循环中等待其子进程的退出事件，并调用 waitid 系统调用来完成“收尸”工作；
init 进程不会被暂停、也不会被杀死（这是由内核来保证的）。它在等待子进程退出的过程中处于 TASK_INTERRUPTIBLE 状态，“收尸”过程中则处于 TASK_RUNNING 状态。

Linux 进程状态：X (TASK_DEAD – EXIT_DEAD)，退出状态，进程即将被销毁。

而进程在退出过程中也可能不会保留它的 task_struct。比如这个进程是多线程程序中被 detach 过的进程（进程？线程？参见《linux 线程浅析》）。或者父进程通过设置 SIGCHLD 信号的 handler 为 SIG_IGN，显式的忽略了 SIGCHLD 信号。（这是 posix 的规定，尽管子进程的退出信号可以被设置为 SIGCHLD 以外的其他信号。）
此时，进程将被置于 EXIT_DEAD 退出状态，这意味着接下来的代码立即就会将该进程彻底释放。所以 EXIT_DEAD 状态是非常短暂的，几乎不可能通过 ps 命令捕捉到。

进程的初始状态
进程是通过 fork 系列的系统调用（fork、clone、vfork）来创建的，内核（或内核模块）也可以通过 kernel_thread 函数创建内核进程。这些创建子进程的函数本质上都完成了相同的功能——将调用进程复制一份，得到子进程。（可以通过选项参数来决定各种资源是共享、还是私有。）
那么既然调用进程处于 TASK_RUNNING 状态（否则，它若不是正在运行，又怎么进行调用？），则子进程默认也处于 TASK_RUNNING 状态。
另外，在系统调用调用 clone 和内核函数 kernel_thread 也接受 CLONE_STOPPED 选项，从而将子进程的初始状态置为 TASK_STOPPED。
进程状态变迁
进程自创建以后，状态可能发生一系列的变化，直到进程退出。而尽管进程状态有好几种，但是进程状态的变迁却只有两个方向——从 TASK_RUNNING 状态变为非 TASK_RUNNING 状态、或者从非 TASK_RUNNING 状态变为 TASK_RUNNING 状态。
也就是说，如果给一个 TASK_INTERRUPTIBLE 状态的进程发送 SIGKILL 信号，这个进程将先被唤醒（进入 TASK_RUNNING 状态），然后再响应 SIGKILL 信号而退出（变为 TASK_DEAD 状态）。并不会从 TASK_INTERRUPTIBLE 状态直接退出。
进程从非 TASK_RUNNING 状态变为 TASK_RUNNING 状态，是由别的进程（也可能是中断处理程序）执行唤醒操作来实现的。执行唤醒的进程设置被唤醒进程的状态为 TASK_RUNNING，然后将其 task_struct 结构加入到某个 CPU 的可执行队列中。于是被唤醒的进程将有机会被调度执行。
而进程从 TASK_RUNNING 状态变为非 TASK_RUNNING 状态，则有两种途径：

响应信号而进入 TASK_STOPED 状态、或 TASK_DEAD 状态；
执行系统调用主动进入 TASK_INTERRUPTIBLE 状态（如 nanosleep 系统调用）、或 TASK_DEAD 状态（如 exit 系统调用）；或由于执行系统调用需要的资源得不到满足，而进入 TASK_INTERRUPTIBLE 状态或 TASK_UNINTERRUPTIBLE 状态（如 select 系统调用）。
显然，这两种情况都只能发生在进程正在 CPU 上执行的情况下。

# 进程与内存

```sh
# 指定查看用户，键入数字 1 查看单个 CPU 的负载，P/M/T 分别切换按照 CPU、内存、CPU 占用时间排序
$ top -u oracle

# Cpu(s): 87.3%us,  1.2%sy,  0.0%ni, 27.6%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
us: user cpu time (or) % CPU time spent in user space
sy: system cpu time (or) % CPU time spent in kernel space
ni: user nice cpu time (or) % CPU time spent on low priority processes
id: idle cpu time (or) % CPU time spent idle
wa: io wait cpu time (or) % CPU time spent in wait (on disk)
hi: hardware irq (or) % CPU time spent servicing/handling hardware interrupts
si: software irq (or) % CPU time spent servicing/handling software interrupts
st: steal time - - % CPU time in involuntary wait by virtual cpu while hypervisor is servicing another processor (or) % CPU time stolen from a virtual machine

# 表格列

# PID：进程的ID
# USER：进程所有者
# PR：进程的优先级别，越小越优先被执行
# NInice：值
# VIRT：进程占用的虚拟内存
# RES：进程占用的物理内存
# SHR：进程使用的共享内存
# S：进程的状态。S表示休眠，R表示正在运行，Z表示僵死状态，N表示该进程优先值为负数
# %CPU：进程占用CPU的使用率
# %MEM：进程使用的物理内存和总内存的百分比
# TIME+：该进程启动后占用的总的CPU时间，即占用CPU使用时间的累加值。
# COMMAND：进程启动命令名称
```

# 存储

## 磁盘 I/O

## MySQL

```sh
$ show full processlist;
$ SELECT HOST FROM information_schema.processlist where user='dbname' and INFO like '%tbname%'"
```

```sh
#!/bin/bash

COUNTER=0
tmp_file=$1

while [ $COUNTER -lt 10000 ];
do
    ss=`mysql -uroot -N -e"SELECT HOST FROM information_schema.processlist where user='dbname' and INFO like '%tbname%'";`
    echo $ss>>${tmp_file}
    let COUNTER=COUNTER+1
done

# 然后使用 awk 命令检索
# awk -F":" '{print $1}' ${tmp_file}| sort | uniq
```

# 网络

# 应用程序

## JVM

## perf

perf 是 linux 下一个非常强大的性能分析工具，通过它可以分析出进程运行过程中的主要时间都花在了哪些地方；结合著名的 [FlameGraph](https://github.com/brendangregg/FlameGraph.git) 火焰图工具，我们能够快速定位 时间占用较多的函数调用。

```sh
# 执行采样
$ perf record -e cpu-clock -g -p ${PID}

# 用 perf script 工具对 perf.data 进行解析
perf script -i perf.data &> perf.unfold

# 将 perf.unfold 中的符号进行折叠
./stackcollapse-perf.pl perf.unfold &> perf.folded

# 生成 svg 火焰图
/flamegraph.pl perf.folded > perf.svg
```
