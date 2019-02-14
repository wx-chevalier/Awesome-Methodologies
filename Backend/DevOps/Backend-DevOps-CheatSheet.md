[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheets)

# Backend DevOps CheatSheet

本文囊括了日常的持续集成、应用部署、服务器维护等方面的命令与技巧，相较于 [基础命令/DevOps/安全加固/Shell 编程](./Linux-Commands-CheatSheet.md) 等相关清单，首先介绍了如何查看服务器的状态与关键指标，然后介绍了指标的含义、评判标准，最后从进程、存储、网络等细分角度进行展开。

![](http://www.brendangregg.com/Perf/linux_perf_tools_full.svg)

# Metrics | 指标

## 系统指标

吞吐量（Throughout）与时延（Latency）是衡量软件系统的最常见的两个指标，系统的吞度量（承压能力）与请求对 CPU 的消耗、外部接口、IO 等等紧密关联；单个请求对 CPU 消耗越高，外部系统接口、IO 影响速度越慢，系统吞吐能力越低，反之越高。吞吐量与时延是天生矛盾的，吞吐量增加也就意味着同一时间请求并发的增加；而由于资源的限制，同一时刻可以处理的请求数是固定的，取决于整个请求处理过程中最小的那个环节。当并发请求数大于这个值时，就会有请求排队等待被处理。所以，要提升服务的吞吐量，必定会增加整体延迟。另外，如果服务的延迟（单个请求的耗时）减少，由于排队的请求的等待时间也减少了，所以吞吐量会上升。

### 可用性指标

- MTTF, Mean Time To Failure，系统平均运行多长时间才发生故障，越长越好

MTTR,Mean Time To Recover, 故障平均修复时间，越短越好

可用性计算公式， Availability= MTTF /（MTTF+MTTR）

MTTF, Mean Time To Failure，系统平均运行多长时间才发生故障，越长越好
MTTR,Mean Time To Recover, 故障平均修复时间，越短越好
可用性计算公式， Availability= MTTF /（MTTF+MTTR）

TPS：Transactions Per Second（每秒传输的事物处理个数），即服务器每秒处理的事务数。TPS 包括一条消息入和一条消息出，加上一次用户数据库访问。（业务 TPS = CAPS × 每个呼叫平均 TPS）。TPS 是软件测试结果的测量单位。一个事务是指一个客户机向服务器发送请求然后服务器做出反应的过程。客户机在发送请求时开始计时，收到服务器响应后结束计时，以此来计算使用的时间和完成的事务个数。一般的，评价系统性能均以每秒钟完成的技术交易的数量来衡量。系统整体处理能力取决于处理能力最低模块的 TPS 值。

QPS：每秒查询率 QPS 是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准，在因特网上，作为域名系统服务器的机器的性能经常用每秒查询率来衡量。

对应 fetches/sec，即每秒的响应请求数，也即是最大吞吐能力。

### 访问量指标

PV    即 page view，页面浏览量          用户每一次对网站中的每个页面访问均被记录 1 次。用户对同一页面的多次刷新，访问量累计。

UV  即 Unique visitor，独立访客      通过客户端的 cookies 实现。即同一页面，客户端多次点击只计算一次，访问量不累计。

IP    即 Internet Protocol，本意本是指网络协议，在数据统计这块指通过 ip 的访问量。     即同一页面，客户端使用同一个 IP 访问多次只计算一次，访问量不累计。

## 性能指标

从客户端来看，延迟就是从发送请求到接收响应的整体耗时，包括：请求的网络耗时，请求在服务端的处理耗时以及响应的网络耗时。吞吐量则是服务在一定的并发下，每秒可以处理的请求数。

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

对于 Linux 进程更多相关的介绍，可以参考[深入浅出 Linux](https://github.com/wxyyxc1992/Distributed-Infrastructure-Series/tree/master/Linux) 相关内容。

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

## CPU

CPU 利用率是业务系统利用到 CPU 的比率，因为往往一个系统上会有一些其他的线程，这些线程会和 CPU 竞争计算资源，那么此时留给业务的计算资源比例就会下降，典型的像，GC 线程的 GC 过程、锁的竞争过程都是消耗 CPU 的过程。甚至一些 IO 的瓶颈，也会导致 CPU 利用率下降(CPU 都在 Wait IO，利用率当然不高)。

## 内存

```sh
# 查看系统当前的内存
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           7.8G        1.2G        627M         85M        6.0G        6.0G
Swap:            0B          0B          0B
```

上图中总内存与可用内存差值出现不一致性，是因为 OS 发现系统的物理内存有大量剩余时，为了提高 IO 的性能，就会使用多余的内存当做文件缓存。

# 存储

## 磁盘与 I/O

```sh
# 查看磁盘剩余空间
$ df -ah
$ df --block-size=GB/-k/-m

# 查看当前目录下的目录空间占用
$ du -h --max-depth=1 /var/ | sort
# 查看 tmp 目录的磁盘占用
$ du -sh /tmp
# 查看当前目录包含子目录的大小
$ du -sm .

# 查看目录下文件尺寸
$ ls -l --sort=size --block-size=M
```

![default](https://user-images.githubusercontent.com/5803001/39466197-45bac832-4d5a-11e8-9c90-1cbdc0762b49.png)

1 处表示系统负载，它表示当前正在等待被 cpu 调度的进程数量，这个值小于系统 vcpu 数(超线程数)的时候是比较正常的，一旦大于 vcpu 数，则说明并发运行的进程太多了，有进程迟迟得不到 cpu 时间。这种情况给用户的直观感受就是敲任何命令都卡。

2 处表示当前系统的总进程数，通常该值过大的时候就会导致 load average 过大。

3 处表示 cpu 的空闲时间，可以反应 cpu 的繁忙程度，该值较高时表示系统 cpu 处于比较清闲的状态，如果该值较低，则说明系统的 cpu 比较繁忙。需要注意的是，有些时候该值比较高，表示 cpu 比较清闲，但是 load average 依然比较高，这种情况很可能就是因为进程数太多，进程切换占用了大量的 cpu 时间，从而挤占了业务运行需要使用的 cpu 时间。

4 处表示进程 IO 等待的时间，该值较高时表示系统的瓶颈可能出现在磁盘和网络。

5 处表示系统的剩余内存，反应了系统的内存使用情况。

6 处表示单个进程的 cpu 和内存使用情况。关于 top 命令中各个指标含义的进一步描述可以参见：

```sh
$ iostat -x -d 2

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.25    0.04    0.53     0.56     4.88    19.25     0.00    6.85    3.09    7.14   0.25   0.01
```

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

## DNS

DNS 解析检索与查询：

```sh
$ dig ns alicdn.com
```

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

# Todos

- [ ] 参考[用十条命令在一分钟内检查 Linux 服务器性能](http://www.infoq.com/cn/news/2015/12/linux-performance)
- [ ] [How Do I Find The Largest Top 10 Files and Directories On a Linux / UNIX / BSD?](http://www.cyberciti.biz/faq/how-do-i-find-the-largest-filesdirectories-on-a-linuxunixbsd-filesystem/)
