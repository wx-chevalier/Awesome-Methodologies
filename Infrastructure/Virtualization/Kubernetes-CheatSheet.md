[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Kubernetes CheatSheet | Kubernetes 基础概念，配置使用与实践技巧

# Concepts & Terminology | 概念与名词

kubeadm: 安装

kubelet: 工作节点上的代理 daemon, 与 master 通信

kubectl: 集群管理工具

## 安装与配置

推荐首先使用 [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) 搭建简单的本地化集群，也可以使用 [Docker 自带的 Kubernetes 实例](https://parg.co/lBZ)；Minikube 需要依次安装 [VirtualBox](https://www.virtualbox.org/wiki/Downloads), [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) 以及 [minikube](https://github.com/kubernetes/minikube/releases)。

kubeadm 用于搭建并启动一个集群，kubelet 用于集群中所有节点上都有的用于做诸如启动 pod 或容器这种事情，kubectl 则是与集群交互的命令行接口。kubelet 和 kubectl 并不会随 kubeadm 安装而自动安装，需要手工安装。

在安装 kubeadm 时候，如果碰到需要翻墙的情况，可以使用 USTC 的源：

```sh
$ vim /etc/apt/sources.list.d/kubernetes.list
$ deb http://mirrors.ustc.edu.cn/kubernetes/apt/ kubernetes-xenial main

$ apt-get install -y kubelet kubeadm kubectl --allow-unauthenticated
$ apt-mark hold kubelet kubeadm kubectl
```

配置 cgroup driver, 保证和 docker 的一样:

```sh
$ docker info | grep -i cgroup
$ vim etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# 添加执行选项
$ /usr/bin/kubelet ... cgroup-driver=systemd

# 配置修改后重启
$ systemctl daemon-reload
$ systemctl restart kubectl
```

kubeadm 安装完毕后，可以初始化 Master 节点：

```sh
kubeadm init
```

完整配置文件可以参考[这里](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file):

```yaml
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
networking:
  podSubnet: 10.244.0.0/16 # 使用 flannel
```

可以手工指定默认网关使用的网络接口，配置非 root 用户:

```sh
$ mkdir -p $HOME/.kube

$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

或者 root 用户

```sh
$ export KUBECONFIG=/etc/kubernetes/admin.conf
```

### 命令行配置

```sh
# Setup autocomplete in bash; bash-completion package should be installed first
$ source <(kubectl completion bash)

# View Kubernetes config
$ kubectl config view

# View specific config items by json path
$ kubectl config view -o jsonpath='{.users[?(@.name == "k8s")].user.password}'
```

# 资源管理

- Get documentation for pod or service

  `kubectl explain pods,svc`

- Create resource(s) like pods, services or daemonsets

  `kubectl create -f ./my-manifest.yaml`

- Apply a configuration to a resource

  `kubectl apply -f ./my-manifest.yaml`

- Start a single instance of Nginx

  `kubectl run nginx --image=nginx`

- Create a secret with several keys

      	```
      	cat <<EOF | kubectl create -f -
      	apiVersion: v1
      	kind: Secret
      	metadata:
      	  name: mysecret
      	type: Opaque
      	data:
      	  password: $(echo -n "s33msi4" | base64)
      	  username: $(echo -n "jane" | base64)
      	EOF
      	```

- Delete a resource

  `kubectl delete -f ./my-manifest.yaml`

# 资源检索

- List all services in the namespace

  `kubectl get services`

- List all pods in all namespaces in wide format

  `kubectl get pods -o wide --all-namespaces`

- List all pods in json (or yaml) format

  `kubectl get pods -o json`

- Describe resource details (node, pod, svc)

  `kubectl describe nodes my-node`

- List services sorted by name

  `kubectl get services --sort-by=.metadata.name`

- List pods sorted by restart count

  `kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'`

- Rolling update pods for frontend-v1

  `kubectl rolling-update frontend-v1 -f frontend-v2.json`

- Scale a replicaset named 'foo' to 3

  `kubectl scale --replicas=3 rs/foo`

- Scale a resource specified in "foo.yaml" to 3

  `kubectl scale --replicas=3 -f foo.yaml`

- Execute a command in every pod / replica

  `for i in 0 1; do kubectl exec foo-$i -- sh -c 'echo $(hostname) > /usr/share/nginx/html/index.html'; done`

# 监控与日志

- Deploy Heapster from Github repository
  https://github.com/kubernetes/heapster

  `kubectl create -f deploy/kube-config/standalone/`

- Show metrics for nodes

  `kubectl top node`

  `kubectl top node my-node-1`

- Show metrics for pods

  `kubectl top pod`

  `kubectl top pod my-pod-1`

- Show metrics for a given pod and its containers

  `kubectl top pod pod_name --containers`

- Dump pod logs (stdout)

  `kubectl logs pod_name`

- Stream pod container logs
  (stdout, multi-container case)

  `kubectl logs -f pod_name -c my-container`

# Pod

- Create a daemonset from stdin. The example deploys [Sematext Docker Agent](https://sematext.com/kuberntes) to all nodes for the cluster-wide collection of metrics, logs and events. There is NO need to deploy cAdvisor, Heapster, Prometheus, Elasticsearch, Grafana, InfluxDb on your local nodes. Please replace YOUR_SPM_DOCKER_TOKEN and YOUR_LOGSENE_TOKEN with your tokens created in [Sematext Cloud UI](https://apps.sematext.com/ui/integrations/create/docker) before you run the command.

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: sematext-agent
spec:
  template:
    metadata:
      labels:
        app: sematext-agent
    spec:
      nodeSelector: {}
      hostNetwork: true
      dnsPolicy: "ClusterFirst"
      restartPolicy: "Always"
      containers:
      - name: sematext-agent
        image: sematext/sematext-agent-docker:latest
        imagePullPolicy: "Always"
        env:
        - name: SPM_TOKEN
          value: "YOUR_SPM_DOCKER_TOKEN"
        - name: LOGSENE_TOKEN
          value: "YOUR_LOGSENE_TOKEN"
        volumeMounts:
          - mountPath: /var/run/docker.sock
            name: docker-sock
          - mountPath: /etc/localtime
            name: localtime
        securityContext:
          privileged: true
      volumes:
        - name: docker-sock
          hostPath:
            path: /var/run/docker.sock
        - name: localtime
          hostPath:
            path: /etc/localtime
EOF
```

# 资源划分

目前，Kubernetes 只支持 CPU 和 Memory 两种资源的申请，Kubernetes 中，根据应用对资源的诉求不同，把应用的 QoS 按照优先级从高到低分为三大类：Guaranteed, Burstable 和 Best-Effort。三种类别分别表示资源配额必须交付、尽量交付以及不保障。QoS 等级是通过 resources 的 limits 和 requests 参数间接计算出来的。CPU 为可压缩资源，Node 上的所有 Pods 共享 CPU 时间片，原理是通过设置 cpu.cfs_quota_us 和 cpu.cfs_period_us 实现，一个 CPU 逻辑核嘀嗒时间被切了 N 份，只要按照百分比例设置 cpu.cfs_quota_us 的值就可以实现 CPU 时间片的比例分配，如设置 2N 表示利用两个 CPU 逻辑核的时间。Memory 为不可压缩资源，kubernetes 中主要利用 memory.limit_in_bytes 实现内存的限制。当应用内存超过了它的 limits，那么会被系统 OOM。内存是不可压缩资源，它的保障的机制最为复杂，kubernetes 利用内核 oom_score 机制，实现了对 Pod 容器内(进程)内存 oom kill 的优先级管控，内核中 OOM Score 的取值范围是[-1000, 1000]，值越大，被系统 KILL 的概率就越高。

## Guaranteed

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

## Burstable

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

## BestEffort

当 Pod 中所有的容器均没设置 requests 和 limits，那么这个 Pod 即为 BestEffort 类型，他们可消费所在 Node 上所有资源，但在资源紧张的时候，也是最优先被杀死。

```yaml
containers:
  name: foo
    resources:
  name: bar
    resources:
```
