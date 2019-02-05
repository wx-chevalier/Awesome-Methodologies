[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheets)

# Linux DevOps & Security CheatSheet

DevOps 的出现，运维的身份职责发生了转变，它不再是专门跑任务脚本或者与机器打交道的人，而是变成了 OpenStack 或者 Kubernetes 的专家，通过搭建 / 管理相关的分布式集群，为研发提供可靠的应用运行环境。DevOps 更重要的方面还是改变了应用交付的流程，从传统的搭火车模式走向持续交付，应用的架构和形态改变了，其方法论也随之而改变。DevOps 和持续交付也被认为是云原生应用的要素。至于 AIOps 是 DevOps 在实践 AI 过程中的一些应用，称不上是范式的改变，AI 在运维领域还远远取代不了人的作用。

[![devops](https://user-images.githubusercontent.com/5803001/36406883-0409874a-1635-11e8-815d-fdd882484bb4.png)](https://www.processon.com/view/link/5a8b9c75e4b059c41ac41001)

# 状态

## top

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

# Todos

- [ ] 参考[用十条命令在一分钟内检查 Linux 服务器性能](http://www.infoq.com/cn/news/2015/12/linux-performance)
- [ ] [How Do I Find The Largest Top 10 Files and Directories On a Linux / UNIX / BSD?](http://www.cyberciti.biz/faq/how-do-i-find-the-largest-filesdirectories-on-a-linuxunixbsd-filesystem/)
