[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Linux DevOps & Security CheatSheet

DevOps 的出现，运维的身份职责发生了转变，它不再是专门跑任务脚本或者与机器打交道的人，而是变成了 OpenStack 或者 Kubernetes 的专家，通过搭建 / 管理相关的分布式集群，为研发提供可靠的应用运行环境。

DevOps 更重要的方面还是改变了应用交付的流程，从传统的搭火车模式走向持续交付，应用的架构和形态改变了，其方法论也随之而改变。DevOps 和持续交付也被认为是云原生应用的要素。

至于 AIOps 是 DevOps 在实践 AI 过程中的一些应用，称不上是范式的改变，AI 在运维领域还远远取代不了人的作用。

本文囊括了日常的持续集成、应用部署、服务器维护等方面的命令与技巧，相较于 [基础命令/DevOps/安全加固/Shell 编程](./Linux-Commands-CheatSheet.md) 等相关清单，首先介绍了如何查看服务器的状态与关键指标，然后介绍了指标的含义、评判标准，最后从进程、存储、网络等细分角度进行展开。

# 状态

## top

# Metrics | 指标

吞吐量(Throughout)与时延(Latency)是衡量软件系统的最常见的两个指标，系统的吞度量（承压能力）与请求对 CPU 的消耗、外部接口、IO 等等紧密关联；单个请求对 CPU 消耗越高，外部系统接口、IO 影响速度越慢，系统吞吐能力越低，反之越高。

从客户端来看，延迟就是从发送请求到接收响应的整体耗时，包括：请求的网络耗时，请求在服务端的处理耗时以及响应的网络耗时。吞吐量则是服务在一定的并发下，每秒可以处理的请求数。延迟和吞吐天生是矛盾的。对于服务来说，请求的处理是一个排队系统，且排队可能发生在请求路径上的任何环节，比如：请求的 TCP 包在路由器排队或者是在服务器的接收缓冲区等等。当吞吐量增加，那么必然会导致同一时间请求并发的增加。由于资源的限制，同一时刻可以处理的请求数是固定的，取决于整个请求处理过程中最小的那个环节。当并发请求数大于这个值时，就会有请求排队等待被处理。所以，要提升服务的吞吐量，必定会增加整体延迟。另外，如果服务的延迟（单个请求的耗时）减少，由于排队的请求的等待时间也减少了，所以吞吐量会上升。

## 系统指标

MTTF, Mean Time To Failure，系统平均运行多长时间才发生故障，越长越好

MTTR,Mean Time To Recover, 故障平均修复时间，越短越好

可用性计算公式， Availability= MTTF /（MTTF+MTTR）

## 访问量指标

PV    即 page view，页面浏览量          用户每一次对网站中的每个页面访问均被记录 1 次。用户对同一页面的多次刷新，访问量累计。

UV  即 Unique visitor，独立访客      通过客户端的 cookies 实现。即同一页面，客户端多次点击只计算一次，访问量不累计。

IP    即 Internet Protocol，本意本是指网络协议，在数据统计这块指通过 ip 的访问量。     即同一页面，客户端使用同一个 IP 访问多次只计算一次，访问量不累计。

## 性能指标

- RPS(Requests Per Second) | 吞吐率: 服务器并发处理能力的量化描述，单位是 reqs/s，指的是某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。

TPS  即 Transactions Per Second 的缩写，每秒处理的事务数目。一个事务是指一个客户机向服务器发送请求然后服务器做出反应的过程。客户机在发送请求时开始计时，收到服务器响应后结束计时，以此来计算使用的时间和完成的事务个数，最终利用这些信息作出的评估分。

- QPS(Queries Per Second): 每秒能处理查询数目。是一台服务器每秒能够相应的查询次数，是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准。

并发连接数（The number of concurrent connections）

概念：某个时刻服务器所接受的请求数目，简单的讲，就是一个会话。

并发用户数（The number of concurrent users，Concurrency Level）

概念：要注意区分这个概念和并发连接数之间的区别，一个用户可能同时会产生多个会话，也即连接数。

用户平均请求等待时间（Time per request）

计算公式：处理完成所有请求数所花费的时间/ （总请求数 / 并发用户数），即
Time per request = Time taken for tests /（ Complete requests / Concurrency Level）

服务器平均请求等待时间（Time per request: across all concurrent requests）
计算公式：处理完成所有请求数所花费的时间 / 总请求数，即
Time taken for / testsComplete requests
可以看到，它是吞吐率的倒数。
同时，它也=用户平均请求等待时间/并发用户数，即
Time per request / Concurrency Level

## 日志配置

对外接口统一拦截捕获，避免异常向外系统传播，自身系统无法感知问题。

严格规范日志输出等级，尤其 Error 级别。影响业务进行或意料外异常输出 Error 级别，Error 级别日志统一输出到独立文件，并接入 xflush 系统错误监控告警。做到 Error 日志输出即为需要人为介入处理。为了避免干扰，对现有 Error 做降噪检查。

服务层日志统一输出，包括耗时、接口成功标识、业务成功标识，为监控做准备。

所有日志 traceId 的统一输出。通过扩展 ch.qos.logback.classic.pattern.ClassicConverter，现实 traceId 自动输出。这极大的提升了系统运维效率。

```xml
<appender name="ERROR-APPENDER"
          class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${LOG_PATH}/common-error.log</file>
    <!-- Error 级别过滤 -->
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>ERROR</level>
      <onMatch>ACCEPT</onMatch>
      <onMismatch>DENY</onMismatch>
  </filter>
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ${LOG_LEVEL_PATTERN:-%5p} - [%thread] : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}
        </pattern>
    </encoder>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!-- 按天滚动，可根据实际量调整单位 -->
        <fileNamePattern>${LOG_PATH}/common-error.log.%d{yyyy-MM-dd}
        </fileNamePattern>
        <maxHistory>15</maxHistory>
    </rollingPolicy>
</appender>
```

```xml
<root level="INFO">
    <!-- root中增加Error输出配置 -->
    <appender-ref ref="ERROR-APPENDER" />
</root>

<logger name="testLog" level="INFO"
        additivity="false">
    <appender-ref ref="WORK_SHIFT_CORE_MONITOR_LOG" />
    <!-- 每个logger增加ERROR输出 -->
    <appender-ref ref="ERROR-APPENDER" />
</logger>
```

# 进程与系统

![](http://www.brendangregg.com/Perf/linux_perf_tools_full.svg)

## 系统状态

```sh
# 查看系统当前的内存
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           7.8G        1.2G        627M         85M        6.0G        6.0G
Swap:            0B          0B          0B
```

上图中总内存与可用内存差值出现不一致性，是因为 OS 发现系统的物理内存有大量剩余时，为了提高 IO 的性能，就会使用多余的内存当做文件缓存。

# 存储

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

# 网络

DNS 解析检索与查询：

```sh
$ dig ns alicdn.com
```

# 状态与性能检索

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

# 应用程序调用

## perf 与火焰图

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

# 安全加固

## 用户记录

```sh
# 查询最近登录到系统的用户和系统重启的时间和日期
$ last reboot | less
```

[基础加固脚本](https://parg.co/K2m)

```sh

```

# Todos

- [ ] 参考[用十条命令在一分钟内检查 Linux 服务器性能](http://www.infoq.com/cn/news/2015/12/linux-performance)
- [ ] [How Do I Find The Largest Top 10 Files and Directories On a Linux / UNIX / BSD?](http://www.cyberciti.biz/faq/how-do-i-find-the-largest-filesdirectories-on-a-linuxunixbsd-filesystem/)
