# 网络模型

在 Kubernetes 网络中存在两种 IP（Pod IP 和 Service Cluster IP），Pod IP 地址是实际存在于某个网卡(可以是虚拟设备)上的，Service Cluster IP 它是一个虚拟 IP，是由 kube-proxy 使用 Iptables 规则重新定向到其本地端口，再均衡到后端 Pod 的。每个 Pod 都拥有一个独立的 IP 地址（IPper Pod），而且假定所有的 pod 都在一个可以直接连通的、扁平的网络空间中。用户不需要额外考虑如何建立 Pod 之间的连接，也不需要考虑将容器端口映射到主机端口等问题。

## Pod 网络空间

同一个 Pod 的容器共享同一个网络命名空间，它们之间的访问可以用 localhost 地址 + 容器端口就可以访问。同一 Node 中 Pod 的默认路由都是 docker0 的地址，由于它们关联在同一个 docker0 网桥上，地址网段相同，所有它们之间应当是能直接通信的。不同 Node 中 Pod 间通信要满足 2 个条件： Pod 的 IP 不能冲突； 将 Pod 的 IP 和所在的 Node 的 IP 关联起来，通过这个关联让 Pod 可以互相访问。

![image](https://user-images.githubusercontent.com/5803001/45594553-71001600-b9cf-11e8-83cf-d8755104e762.png)

## 网络开源组件

### 网络解决方案

**隧道方案（ Overlay Networking ）**

隧道方案在 IaaS 层的网络中应用也比较多，大家共识是随着节点规模的增长复杂度会提升，而且出了网络问题跟踪起来比较麻烦，大规模集群情况下这是需要考虑的一个点。

- **Weave：**UDP 广播，本机建立新的 BR，通过 PCAP 互通
- **Open vSwitch（OVS）：**基于 VxLan 和 GRE 协议，但是性能方面损失比较严重
- **Flannel：**UDP 广播，VxLan
- **Racher：**IPsec

**路由方案**

路由方案一般是从 3 层或者 2 层实现隔离和跨主机容器互通的，出了问题也很容易排查。

- **Calico：**基于 BGP 协议的路由方案，支持很细致的 ACL 控制，对混合云亲和度比较高。
- **Macvlan：**从逻辑和 Kernel 层来看隔离性和性能最优的方案，基于二层隔离，所以需要二层路由器支持，大多数云服务商不支持，所以混合云上比较难以实现。

容器网络发展到现在，形成了两大阵营，就是 Docker 的 CNM 和 Google、CoreOS、Kuberenetes 主导的 CNI。首先明确一点，CNM 和 CNI 并不是网络实现，他们是网络规范和网络体系，从研发的角度他们就是一堆接口，你底层是用 Flannel 也好、用 Calico 也好，他们并不关心，CNM 和 CNI 关心的是网络管理的问题。

**CNM（Docker LibnetworkContainer Network Model）:**

Docker Libnetwork 的优势就是原生，而且和 Docker 容器生命周期结合紧密；缺点也可以理解为是原生，被 Docker“绑架”。

- Docker Swarm overlay
- Macvlan & IP networkdrivers
- Calico
- Contiv
- Weave

**CNI（Container NetworkInterface）：**

CNI 的优势是兼容其他容器技术（e.g. rkt）及上层编排系统（Kubernetes & Mesos)，而且社区活跃势头迅猛，Kubernetes 加上 CoreOS 主推；缺点是非 Docker 原生。

- Kubernetes
- Weave
- Macvlan
- Calico
- Flannel
- Contiv
- Mesos CNI

### Flannel

Flannel 是 CoreOS 团队针对 Kubernetes 设计的一个网络规划服务，简单来说，它的功能是让集群中的不同节点主机创建的 Docker 容器都具有全集群唯一的虚拟 IP 地址。flannel 和 openvswitch 思路基本一致，就是当 Docker 在宿主机上创建一个网桥的时候，用自己的网桥替代它。
在默认的 Docker 配置中，每个节点上的 Docker 服务会分别负责所在节点容器的 IP 分配。这样导致的一个问题是，不同节点上容器可能获得相同的内外 IP 地址。并使这些容器之间能够之间通过 IP 地址相互找到，也就是相互 ping 通。
Flannel 的设计目的就是为集群中的所有节点重新规划 IP 地址的使用规则，从而使得不同节点上的容器能够获得“同属一个内网”且”不重复的”IP 地址，并让属于不同节点上的容器能够直接通过内网 IP 通信。
Flannel 实质上是一种“覆盖网络(overlaynetwork)”，也就是将 TCP 数据包装在另一种网络包里面进行路由转发和通信，目前已经支持 udp、vxlan、host-gw、aws-vpc、gce 和 alloc 路由等数据转发方式，默认的节点间数据通信方式是 UDP 转发。

![image](https://user-images.githubusercontent.com/5803001/45594693-8413e580-b9d1-11e8-90f0-cdf66129187d.png)

数据请求从容器 1(10.0.46.2:2379)中发出后，首先经由所在主机的 docker0 虚拟网卡(10.0.46.1)转发到 flannel0 虚拟网卡(10.0.46.0)，这是个 P2P 虚拟网卡，Flannel 通过修改 Node 路由表的方式实现 flanneld 服务监听 flannel0 虚拟网卡数据。

接着 flannel 服务将原本的数据内容 UDP 封装后根据自己的路由表投递给目的节点的 flanneld 服务。在此包中，包含有 outer-ip(source:192.168.8.227, dest:192.168.8.228)，inner-ip(source:10.0.46.2:2379, dest:10.0.90.2:8080)。

数据到达 node2 以后被解包，直接进入目的节点的 flannel0 虚拟网卡中(10.0.90.0)，且被转发到目的主机的 docker0 虚拟网卡(10.0.90.1)，最后就像本机容器通信一样由 docker0 路由到达目标容器 2(10.0.90.2:8080)。

为使每个结点上的容器分配的地址不冲突。Flannel 通过 Etcd 分配了每个节点可用的 IP 地址段后，再修改 Docker 的启动参数。“--bip=X.X.X.X/X”这个参数，它限制了所在节点容器获得的 IP 范围。

### Weave

在每个宿主机上布置一个特殊的 route 的容器，不同宿主机的 route 容器连接起来。 route 拦截所有普通容器的 ip 请求，并通过 udp 包发送到其他宿主机上的普通容器。 这样在跨机的多个容器端看到的就是同一个扁平网络。 weave 解决了网络问题，不过部署依然是单机的。

![image](https://user-images.githubusercontent.com/5803001/45594701-a3127780-b9d1-11e8-8067-6b25fd5a9064.png)

# 资源调度

## Pod 调度

当新增一个 Pod 时，集群会在可用的集群节点中寻找最合适的节点来运行相应的容器。Kubernetes 首先会排除无效节点:

- 节点状态为不可用的，如，节点不通或者 k8s 服务运行异常等；
- 节点剩余的 CPU,内存资源不足以运行容器的；
- 容器运行时占用的宿主机端口出现冲突的；
- 按照节点选择 label 不匹配的；

Pod.spec.nodeSelector 通过 kubernetes 的 label-selector 机制选择节点，由调度器调度策略匹配 label，而后调度 pod 到目标节点，该匹配规则属于强制约束。

然后通过打分机制决定将 Pod 具体调度到剩余机器中的那一台，默认调度节点选择策略权重为 1，节点的调度规则是采用 plugin 方式，允许自行编写调度策略进行打分处理。

```sh
# 标准打分公式
score = (权重 * 评价策略分值)  + (weight1 * priorityFunc1) + (weight2 * priorityFunc2) + ...

# LeastRequestedPriority
score = cpu((capacity - sum(requested)) * 10 / capacity) + memory((capacity - sum(requested)) * 10 / capacity) /2

# BalanceResourceAllocation
score = 10 -abs (cpuFraction - memoryFraction) * 10
cpuFraction = requested / capacity;           memoryFraction = requested / capacity

# CalculateSpreadPriority
score = 10 * ((maxCount -counts) / (maxCount))
```

- LeastRequestedPriority: CPU 可用资源为 100，运行容器申请的资源为 15，则 cpu 分值为 8.5 分，内存可用资源为 100，运行容器申请资源为 20，则内存分支为 8 分。则此评价规则在此节点的分数为(8.5 +8) / 2 = 8.25 分。

- BalanceResourceAllocation: CPU 可用资源为 100，申请 10，则 cpuFraction 为 0.1，而内存可用资源为 20，申请 10，则 memoryFraction 为 0.5，这样由于 CPU 和内存使用不均衡，此节点的得分为 10-abs ( 0.1 - 0.5 ) \* 10 = 6 分。假如 CPU 和内存资源比较均衡，例如两者都为 0.5，那么代入公式，则得分为 10 分。

- CalculateSpreadPriority: 一个 web 服务，可能存在 5 个实例，例如当前节点已经分配了 2 个实例了，则本节点的得分为 10 _ ((5-2) / 5) = 6 分，而没有分配实例的节点，则得分为 10 _ ((5-0) / 5) = 10 分。没有分配实例的节点得分越高。

## 资源保障

目前，Kubernetes 只支持 CPU 和 Memory 两种资源的申请，Kubernetes 中，根据应用对资源的诉求不同，把应用的 QoS 按照优先级从高到低分为三大类：Guaranteed, Burstable 和 Best-Effort。三种类别分别表示资源配额必须交付、尽量交付以及不保障。QoS 等级是通过 resources 的 limits 和 requests 参数间接计算出来的。CPU 为可压缩资源，Node 上的所有 Pods 共享 CPU 时间片，原理是通过设置 cpu.cfs_quota_us 和 cpu.cfs_period_us 实现，一个 CPU 逻辑核嘀嗒时间被切了 N 份，只要按照百分比例设置 cpu.cfs_quota_us 的值就可以实现 CPU 时间片的比例分配，如设置 2N 表示利用两个 CPU 逻辑核的时间。Memory 为不可压缩资源，kubernetes 中主要利用 memory.limit_in_bytes 实现内存的限制。当应用内存超过了它的 limits，那么会被系统 OOM。内存是不可压缩资源，它的保障的机制最为复杂，kubernetes 利用内核 oom_score 机制，实现了对 Pod 容器内(进程)内存 oom kill 的优先级管控，内核中 OOM Score 的取值范围是[-1000, 1000]，值越大，被系统 KILL 的概率就越高。

### Guaranteed

如果 Pod 中每一个容器都只设置了 limits 参数，或者 同时设置了 limits 和 requests 并且 limits 和 requests 的值一样，那么这个 Pod 就是 Guaranteed 类型的。

```yaml
containers:
  name: foo
    resources:
      limits: //只设置了limits
        cpu: 10m
        memory: 1Gi
  name: bar
    resources:
      limits:
        cpu: 100m
        memory: 100Mi
      requests:  //requests 和 limits均已设定，并且值相同
        cpu: 100m
        memory: 100Mi
```

### Burstable

当以下情形设置，Pod 会为定位成 Burstable 类型, Busrtable 类型保障了资源的最小需求，但不会超过`limits`。

- Pod 里的一个或多个容器只设置了`requests`参数。
- Pod 里的一个或多个容器同时设置了`requests`和`limits`参数，但是两者值不一样。
- Pod 里的所有容器均设置了`limits`，但是他们的类型不一样，不如容器 1 只定义了 CPU，容器 2 只定义了 Memory。
- Pod 里存在多个容器时，其中存在容器可被定义为 Bustable 条件的 Pod 也是 Bustable 类型，比如有两个容器，容器 1 设置了`limits`，容器 2 没有任何设置。

```yaml
containers:
  name: foo
    resources:
      limits:
        memory: 1Gi

  name: bar
    resources:
      limits:
        cpu: 100m

  name: duck
```

### BestEffort

当 Pod 中所有的容器均没设置 requests 和 limits，那么这个 Pod 即为 BestEffort 类型，他们可消费所在 Node 上所有资源，但在资源紧张的时候，也是最优先被杀死。

```yaml
containers:
  name: foo
    resources:
  name: bar
    resources:
```
