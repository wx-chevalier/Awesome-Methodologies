**CNI（Container NetworkInterface）：**

### Flannel

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
