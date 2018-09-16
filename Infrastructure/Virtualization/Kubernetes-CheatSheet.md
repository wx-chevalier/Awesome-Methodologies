[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Kubernetes CheatSheet | Kubernetes 基础概念，配置使用与实践技巧

[官方的交互式教程](https://kubernetes.io/docs/tutorials/)也是非常不错的入门资源。

# Concepts & Terminology | 概念与名词

kubeadm 是集群的安装配置脚手架，kubectl 是集群管理工具，kubelet 是工作节点上的代理 Daemon 服务, 负责与 Master 节点进行通信。

## 架构

### Master Components

Master components provide the cluster’s control plane. Master components make global decisions about the cluster (for example, scheduling), and detecting and responding to cluster events (starting up a new pod when a replication controller’s ‘replicas’ field is unsatisfied).

kube-apiserver
Component on the master that exposes the Kubernetes API. It is the front-end for the Kubernetes control plane.

etcd
Consistent and highly-available key value store used as Kubernetes’ backing store for all cluster data.

kube-scheduler
Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.

kube-controller-manager

Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.

These controllers include:

- Node Controller: Responsible for noticing and responding when nodes go down.
- Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
- Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
- Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.

cloud-controller-manager
cloud-controller-manager runs controllers that interact with the underlying cloud providers.
cloud-controller-manager runs cloud-provider-specific controller loops only. You must disable these controller loops in the kube-controller-manager.
cloud-controller-manager allows cloud vendors code and the Kubernetes core to evolve independent of each other. In prior releases, the core Kubernetes code was dependent upon cloud-provider-specific code for functionality. In future releases, code specific to cloud vendors should be maintained by the cloud vendor themselves, and linked to cloud-controller-manager while running Kubernetes.

The following controllers have cloud provider dependencies:

Node Controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
Route Controller: For setting up routes in the underlying cloud infrastructure
Service Controller: For creating, updating and deleting cloud provider load balancers
Volume Controller: For creating, attaching, and mounting volumes, and interacting with the cloud provider to orchestrate volumes

### Node Components

Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

Container Runtime
The container runtime is the software that is responsible for running containers. Kubernetes supports several runtimes: Docker, rkt, runc and any OCI runtime-spec implementation.

kubelet
An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.

The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn’t manage containers which were not created by Kubernetes.

kube-proxy
kube-proxy enables the Kubernetes service abstraction by maintaining network rules on the host and performing connection forwarding.

## 资源对象

In Kubernetes, a group of one or more containers is called a Pod. Containers in a Pod are deployed together, and are started, stopped, and replicated as a group.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.7.9
      ports:
        - containerPort: 80
```

A Deployment object defines a Pod creation template (a “cookie-cutter” if you will) and desired replica count. The Deployment uses a label selector to identify the Pods it manages, and will create or delete Pods as needed to meet the replica count. Deployments are also used to manage safely rolling out changes to your running Pods.

Once you have a replicated set of Pods, you need an abstraction that enables connectivity between the layers of your application. For example, if you have a Deployment managing your backend jobs, you don’t want to have to reconfigure your front-ends whenever you re-scale your backends. Likewise, if the Pods in your backends are scheduled (or rescheduled) onto different machines, you can’t be required to re-configure your front-ends. In Kubernetes, the service abstraction achieves these goals. A service provides a way to refer to a set of Pods (selected by labels) with a single static IP address. It may also provide load balancing, if supported by the provider.

# 安装与配置

推荐首先使用 [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) 搭建简单的本地化集群，其需要依次安装 [VirtualBox](https://www.virtualbox.org/wiki/Downloads), [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) 以及 [minikube](https://github.com/kubernetes/minikube/releases) 等工具；在生产环境下，我们常常需要离线安装，此时可以参考[离线安装 K8S](https://parg.co/AT5)。

## 集群初始化

kubeadm 用于搭建并启动一个集群，kubelet 用于集群中所有节点上都有的用于做诸如启动 pod 或容器这种事情，kubectl 则是与集群交互的命令行接口。kubelet 和 kubectl 并不会随 kubeadm 安装而自动安装，需要手工安装。

在安装 kubeadm 时候，如果碰到需要翻墙的情况，可以使用 USTC 的源：

```sh
# 添加源并且更新
$ vim /etc/apt/sources.list.d/kubernetes.list
$ deb http://mirrors.ustc.edu.cn/kubernetes/apt/ kubernetes-xenial main
$ apt-get update

$ apt-get install -y kubelet kubeadm kubectl --allow-unauthenticated
$ apt-mark hold kubelet kubeadm kubectl
```

配置 cgroup driver, 保证和 docker 的一样:

```sh
$ docker info | grep -i cgroup

# 编辑配置文件
$ vim /etc/default/kubelet

# 添加如下配置
KUBELET_KUBEADM_EXTRA_ARGS=--cgroup-driver=<value>

# 配置修改后重启
$ systemctl daemon-reload
$ systemctl restart kubelet
```

kubeadm 安装完毕后，可以初始化 Master 节点：

```sh
$ kubeadm init

# 如果存在网络问题，则可以使用代理访问
$ HTTP_PROXY=127.0.0.1:8118 HTTPS_PROXY=127.0.0.1:8118 kubeadm init

# 接下来我们还需要设置配置文件以最终启动集群
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 或者 Root 用户还可以添加如下映射
$ export KUBECONFIG=/etc/kubernetes/admin.conf
```

值得一提的是，如果无法通过代理访问，还可以使用国内的镜像数据，可以使用[如下脚本](https://github.com/anjia0532/gcr.io_mirror)便捷录取墙外镜像:

```sh
# credits: https://github.com/anjia0532/gcr.io_mirror

images=$(cat img.txt)

eval $(echo ${images}|
        sed 's/k8s\.gcr\.io/anjia0532\/google-containers/g;s/gcr\.io/anjia0532/g;s/\//\./g;s/ /\n/g;s/anjia0532\./anjia0532\//g' |
        uniq |
        awk '{print "docker pull "$1";"}'
       )

for img in $(docker images --format "{{.Repository}}:{{.Tag}}"| grep "anjia0532"); do
  n=$(echo ${img}| awk -F'[/.:]' '{printf "gcr.io/%s",$2}')
  image=$(echo ${img}| awk -F'[/.:]' '{printf "/%s",$3}')
  tag=$(echo ${img}| awk -F'[:]' '{printf ":%s",$2}')
  docker tag $img "${n}${image}${tag}"
  [[ ${n} == "gcr.io/google-containers" ]] && docker tag $img "k8s.gcr.io${image}${tag}"
done
```

Master 节点初始化完毕后，我们需要加入工作节点，或者设置 Master 节点上可调度 Pods

```sh
# 如果是单机节点，要在 Master 机器上调度 Pods，还需解锁 Master 限制
$ kubectl taint nodes --all node-role.kubernetes.io/master-

# 创建并且打印出工作节点加入集群的命令
$ sudo kubeadm token create --print-join-command

# 查看 Token 列表
$ kubeadm token list

# 工作节点加入集群
$ kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

我们还需要配置节点间通信的网络:

```sh
# 安装 Weave 网络
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

# 或者安装 Flannel 网络
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
```

我们也可以使用自定义的配置文件来配置 K8S 集群，譬如可以手工指定默认网关使用的网络接口，完整配置文件可以参考[这里](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file):

```yaml
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
networking:
  podSubnet: 10.244.0.0/16 # 使用 flannel
```

可以配置 kubernetes-dashboard 作为首个服务:

```sh
# 安装 Dashboard
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

$ kubectl proxy --address 0.0.0.0 --accept-hosts '.*'

# http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

## 基础命令

Kubernetes 内建提供了命令行自动补全的配置:

```sh
# Setup autocomplete in bash; bash-completion package should be installed first
$ source <(kubectl completion bash)

# View Kubernetes config
$ kubectl config view

# View specific config items by json path
$ kubectl config view -o jsonpath='{.users[?(@.name == "k8s")].user.password}'
```

### 资源管理

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

### 日志

```sh
# 倒序查看 kubelet apply 日志
$ journalctl -ru kubelet

# 查看某个 POD 部署的情况
$ kubectl get -f dashboard.yaml -o json
```

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

# Service

创建包含两个 POD 的应用:

```sh
# 创建 Deployment
$ kubectl run hello-world --replicas=2 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0  --port=8080

# 获取 Deployment 相关的信息
$ kubectl get deployments hello-world
$ kubectl describe deployments hello-world

# 获取关联的 ReplicaSet 对象
$ kubectl get replicasets
$ kubectl describe replicasets

# 创建 Service 对象，并且暴露服务
$ kubectl expose deployment hello-world --type=NodePort --name=example-service

# 获取服务信息
$ kubectl describe services example-service

# 获取关联的 Pod 信息
$ kubectl get pods --selector="run=load-balancer-example" --output=wide

# 访问服务
$ curl http://<public-node-ip>:<node-port>
```

# 资源分配

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

# Helm
