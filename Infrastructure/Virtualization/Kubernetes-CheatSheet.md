[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Kubernetes CheatSheet | Kubernetes 基础概念，配置使用与实践技巧

Kubernetes 是支持多种底层容器虚拟化技术的分布式容器编排架构，具有完备的功能用于支撑分布式系统以及微服务架构，同时具备超强的横向扩容能力；它提供了自动化容器的部署和复制，随时扩展或收缩容器规模，将容器组织成组，并且提供容器间的负载均衡，提供容器弹性等特性。

[Kubernetes Links]()

[官方的交互式教程](https://kubernetes.io/docs/tutorials/)也是非常不错的入门资源。

# Concepts & Terminology | 概念与术语

kubeadm 是集群的安装配置脚手架，kubectl 是集群管理工具，kubelet 是工作节点上的代理 Daemon 服务, 负责与 Master 节点进行通信。

## 架构

![image](https://user-images.githubusercontent.com/5803001/45594484-eff44f00-b9cd-11e8-9a87-c19e9b0107ee.png)

### Master Components

Master 节点主要由四个模块组成：APIServer、Scheduler、ControllerManager、etcd。
APIServer：整个系统的控制入口，负责对外提供 RESTful API 服务。
Scheduler：负责资源调度，接受来自 APIServer 创建 Pods 任务，并进行调度。
ControllerManager：Kubernetes 内所有资源对象的自动化控制中心，资源对象的“大总管”。
etcd：一个高可用的键值存储系统，负责节点间的服务发现和配置共享。

Master components provide the cluster’s control plane. Master components make global decisions about the cluster (for example, scheduling), and detecting and responding to cluster events (starting up a new pod when a replication controller’s ‘replicas’ field is unsatisfied).

kube-apiserver
Component on the master that exposes the Kubernetes API. It is the front-end for the Kubernetes control plane.

All communication paths from the cluster to the master terminate at the apiserver (none of the other master components are designed to expose remote services). In a typical deployment, the apiserver is configured to listen for remote connections on a secure HTTPS port (443) with one or more forms of client authentication enabled.

etcd
Consistent and highly-available key value store used as Kubernetes’ backing store for all cluster data.

kube-scheduler
Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.

kube-controller-manager

Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.

These controllers include:

- Node Controller: Responsible for noticing and responding when nodes go down.
- Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system. 副本控制器定义了一个期望的场景，并通过各种机制来保障整体运行环境中的 Pod 副本量符合预期。真正意义上实现了声明式容器管理，对系统维护人员屏蔽了主机监控、应用监控、故障恢复等基础运维工作。
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

There are two primary communication paths from the master (apiserver) to the cluster. The first is from the apiserver to the kubelet process which runs on each node in the cluster. The second is from the apiserver to any node, pod, or service through the apiserver’s proxy functionality.

The connections from the apiserver to the kubelet are used for:

Fetching logs for pods.
Attaching (through kubectl) to running pods.
Providing the kubelet’s port-forwarding functionality.

Node 节点主要由三个模块组成：kubelet、kube-proxy、Docker Engine。
kubelet：Node 节点上面的 agent，负责 Pod 对应的容器的创建、启停等任务，同时周期性获取容器状态、资源使用情况反馈给 Master。
kube-proxy：实现 Kubernetes Service 的通信与负载均衡机制的重要组件，具备服务发现和反向代理等功能（Pod 网络代理）。
Docker Engine：Docker 引擎，负责本机的容器创建和管理工作（Docker、Rocket）

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

### Pod

每个 Pod 都包含一个“根容器”（Pause 容器）和多个紧密相关的用户业务容器。每个 Pod 都会提供一个独立的 Endpoint(Pod IP + Container Port)以供调用方访问。在一组容器作为一个单元的情况下难以对这个“整体”简单进行判断以及进行行动。Pod 内用一个业务无关的容器来代表这个整体的状态。
业务容器之间的通信与文件共享问题。Pod 基于虚拟二层网络技术（Flannel）实现任意两个跨主机的 Pod 之间可直接通信。

Pod（上图绿色方框）安排在节点上，包含一组容器和卷。同一个 Pod 里的容器共享同一个网络命名空间，可以使用 localhost 互相通信。Pod 是短暂的，不是持续性实体。

### Label

一个 Label 是 attach 到 Pod 的一对键/值对，用来传递用户定义的属性。比如，你可能创建了一个"tier"和“app”标签，通过 Label（tier=frontend, app=myapp）来标记前端 Pod 容器，使用 Label（tier=backend, app=myapp）标记后台 Pod。然后可以使用 Selectors 选择带有特定 Label 的 Pod，并且将 Service 或者 Replication Controller 应用到上面。

一个 Label 是一个 key=value 的键值对，由用于自行指定。Label 可以附加到各种 Kubernetes 资源上，例如 Node、Pod、Service、RC 等，Label 与资源对象是多对多的关系，其可以在资源对象定义是确定，也可以在资源对象创建后动态添加、删除。

Kubernetes 中非常核心重要的功能之一，用于分类、检索资源。重要使用场景有以下几处：
kube-controller 进程通过 RC 上定义的 Label Selector 来筛选要监控的 Pod 副本数量，从而实现 Pod 副本数量始终符合预期设定的全自动控制流程。
kube-proxy 进程通过 Service 的 Label Selector 来选择对应的 Pod，自动建立起每个 Service 到对应 Pod 的请求转发路由表，从而实现 Service 的智能负载均衡机制。
kube-scheduler 基于 Pod 的 NodeSelector 标签调度策略实现 Pod“定向调度”的特性。

### Replica Set & Deployment

Deployment 底层基于 Replica Set 实现，相比于 RC 的一个最大的功能升级是管理了 Pod 的部署进度，另可以回滚不稳定的 Deployment、挂起或恢复一个 Deployment。

Kubernetes 1.2 中基于 RC 引入新概念 Replica Set，以及其升级版 Deployment

RC 的定义包括如下几个部分：
Pod 期待的副本数（relicase）。
用于筛选目标 Pod 的 Label Selector。
创建新 Pod 的 Pod 模板（template）。

RC 应用场景
可以通过定义 RC 实现 Pod 的创建过程。
RC 通过 Label Selector 机制实现对副本数量的自动控制。
改变 RC 里 Pod 副本数量，实现 Pod 的扩容、缩容。
改变 RC 里 Pod 模板中的镜像版本，实现 Pod 的滚动升级。

### Service

Services 是定义 Pod 副本集群以及访问这些 Pod 的策略的一层抽象。Service 通过 Label Selector 找到 Pod 副本集群。Kubernetes Service 之间通过 TCP/IP 进行通信，由这一个一个微服务单元组建起微服务集群。
Service 由 kube-proxy 实现软件负载均衡器，负责将对 Service 的请求转发到后端的某个 Pod 实例上。且 Kubernetes 为每个 Service 分配了一个全局唯一的虚拟 IP 地址(Cluster IP)，每个服务在 Kubernetes 架构上即变成了具备唯一 IP 地址的“通信节点”。
基于上述这点，Kubernetes 将 Service Name 与 Service Cluster IP 做一个 DNS 域名映射，优雅的解决了服务发现的问题。

Kubernetes 的“三种 IP”
Node IP：Node 节点的 IP 地址，物理网卡的 IP 地址。
Pod IP：Pod 的 IP 地址，有 Docker Engine 根据 docker0 网桥的 IP 地址段进行分配的，一个虚拟的二层网络。
Cluster IP：Service 的虚拟 IP 地址，仅作用与 Kubernetes Service 这个对象，由 Master 管理分配。
注：Kubernetes 对 Service 的负载均衡控制时很开放的，可以是硬\软件负载均衡器，如果集群运行在 Google 公有云上，只需简单配置即可自动创建 Load Balancer 实例，若集群跑在其他云厂商，只需要实现支持此特性的驱动即可。

Kubernetes 提供了内置的 dns 机制和 ClusterIP 机制，每个 Service 都自动注册域名，分配 ClusterIP，这样服务间的依赖可以从 IP 变为 name。
DNS server 通过 kubernetes api server 来观测是否有新 service 建立，并为其建立对应的 dns 记录。如果集群已经 enable DNS，那么 pod 可以自动对 service 做 name 解析。
例：有个叫做”my-service“的 service，他对应的 kubernetes namespace 为”my-ns“，那么会有他对应的 dns 记录，叫做”my-service.my-ns“。那么在 my-ns 的 namespace 中的 pod 都可以对 my-service 做 name 解析来轻松找到这个 service。在其他 namespace 中的 pod 解析”my-service.my-ns“来找到他。解析出来的结果是这个 service 对应的 cluster ip。

### Volume

Kubernetes Volume 不等价于 Docker Volume，其是定义在 Pod 上，可被 Pod 里的多个容器所共享，且生命周期与 Pod 相同。
Kubernetes Volume 支持将 Pod 存储卷挂载到 Google 公有云提供的 Persistent Disk 上，与云产品打通，另外还支持将 Volume 挂载到 Amazon awsElasticBlockStore 上。(将大云厂商提供的能力组件化集成，打破整体壁垒)

# 配置与实践

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

# 服务配置

## Pod

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
      dnsPolicy: 'ClusterFirst'
      restartPolicy: 'Always'
      containers:
        - name: sematext-agent
          image: sematext/sematext-agent-docker:latest
          imagePullPolicy: 'Always'
          env:
            - name: SPM_TOKEN
              value: 'YOUR_SPM_DOCKER_TOKEN'
            - name: LOGSENE_TOKEN
              value: 'YOUR_LOGSENE_TOKEN'
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
```

## Service

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

kubernetes 支持两种方式将 service 开放到公网 IP，NodePort、LoadBalancer。
service type
ClusterIP： 使用集群内的私有 ip —— 默认值。
NodePort： 除了使用 ClusterIP 外，也将 service 的 port 映射到每个 node 的一个指定内部 port 上，映射的每个 node 的内部 port 都一样，支持 TCP/UDP。
LoadBalancer： 使用一个 ClusterIP & NodePort，会向 cloud provider 申请映射到 service 本身的负载均衡，支持 TCP。

```yaml
spec:
  ports:
    - protocol: 'TCP'
    - port: 80
    - targetPort: 9376
```

# 网络模型

在 Kubernetes 网络中存在两种 IP（Pod IP 和 Service Cluster IP），Pod IP 地址是实际存在于某个网卡(可以是虚拟设备)上的，Service Cluster IP 它是一个虚拟 IP，是由 kube-proxy 使用 Iptables 规则重新定向到其本地端口，再均衡到后端 Pod 的。每个 Pod 都拥有一个独立的 IP 地址（IPper Pod），而且假定所有的 pod 都在一个可以直接连通的、扁平的网络空间中。用户不需要额外考虑如何建立 Pod 之间的连接，也不需要考虑将容器端口映射到主机端口等问题。

## Pod 网络空间

同一个 Pod 的容器共享同一个网络命名空间，它们之间的访问可以用 localhost 地址 + 容器端口就可以访问。同一 Node 中 Pod 的默认路由都是 docker0 的地址，由于它们关联在同一个 docker0 网桥上，地址网段相同，所有它们之间应当是能直接通信的。不同 Node 中 Pod 间通信要满足 2 个条件： Pod 的 IP 不能冲突； 将 Pod 的 IP 和所在的 Node 的 IP 关联起来，通过这个关联让 Pod 可以互相访问。

![image](https://user-images.githubusercontent.com/5803001/45594553-71001600-b9cf-11e8-83cf-d8755104e762.png)

## Service 与负载均衡

kubernetes 中通过 service 概念来对应用做多 POD 间的负载均衡，Service 是一组 Pod 的服务抽象，相当于一组 Pod 的 LB，负责将请求分发给对应的 Pod；Service 会为这个 LB 提供一个 IP，一般称为 ClusterIP。Service Cluster IP 是 Kubernetes 系统中的虚拟 IP 地址，由系统动态分配；Kubernetes 集群中的每个节点都会运行 kube-proxy，其负责为 ExternalName 以外的服务实现虚拟 IP 形式，v1.2 版本后默认使用的是 iptables。

Kube-proxy 是一个简单的网络代理和负载均衡器，它的作用主要是负责 Service 的实现，具体来说，就是实现了内部从 Pod 到 Service 和外部的从 NodePort 向 Service 的访问。

![image](https://user-images.githubusercontent.com/5803001/45594895-ff2acb00-b9d4-11e8-89ed-a7b0f724c249.png)

kubernetes 中的每个 node 都会运行一个 kube-proxy。他为每个 service 都映射一个本地 port，任何连接这个本地 port 的请求都会转到 backend 后的随机一个 pod，service 中的字段 SessionAffinity 决定了使用 backend 的哪个 pod，最后在本地建立一些 iptables 规则，这样访问 service 的 cluster ip 以及对应的 port 时，就能将请求映射到后端的 pod 中。

![image](https://user-images.githubusercontent.com/5803001/45594757-832f8380-b9d2-11e8-9e61-b696f63051fd.png)

在这种模式下，kube-proxy 监视 Kubernetes 主服务器添加和删除服务和端点对象。对于每个服务，它安装 iptables 规则，捕获到服务的 clusterIP（虚拟）和端口的流量，并将流量重定向到服务的后端集合之一。对于每个 Endpoints 对象，它安装选择后端 Pod 的 iptables 规则。默认情况下，后端的选择是随机的。可以通过将 service.spec.sessionAffinity 设置为“ClientIP”（默认为“无”）来选择基于客户端 IP 的会话关联。与用户空间代理一样，最终结果是绑定到服务的 IP:端口的任何流量被代理到适当的后端，而客户端不知道关于 Kubernetes 或服务或 Pod 的任何信息。这应该比用户空间代理更快，更可靠。然而，与用户空间代理不同，如果最初选择的 Pod 不响应，则 iptables 代理不能自动重试另一个 Pod，因此它取决于具有工作准备就绪探测。

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

# 生态圈

## Istio

在 Kubernetes 之后，Istio 是最受欢迎的云原生技术。它就是一种服务网格，能够安全的连接一个应用程序之间的多个微服务。你也可以将它视为内部和外部的负载均衡器，具有策略驱动的防火墙，支持各种全面指标。开发者和使用者倾向于 Istio 的原因是因为它具有无侵入式的部署模式，而且任何 Kubernetes 的服务都能够在不需要改动代码和配置的情况下和 Istio 进行无缝连接。

如果说 Kubernetes 是新型的操作系统的话，Helm 就是应用程序安装程序。根据 Debian 安装包和 Red Hat Linux RMPS 设计，Helm 通过执行单个命令，提供了更简洁和更强大的部署云原生工作负载能力。

Kubernetes 应用暴露了大量的像 deployments（部署），services（服务），ingress controllers（入口控制器），persistant volumes（持久化挂载目录）等更多的元素。Helm 则通过提供统一安装工具，将云原生应用程序所有依赖关系聚合到称之为图表的部署单元中。

4. Spinnaker
   云原生技术最值得关注之一的是软件的交付速度。Spinnaker 是一个最初在 Netflix 上构建的开源项目，它实现了这一承诺。Spinnaker 是一个版本管理工具，它添加部署云原生应用的模板。通过对比传统的 Iass 环境（像 Amazon EC2 和当代运行在 Kubernetes 上的 Cass 平台），无缝填补了传统虚拟机和容器之间的空白。其多云功能使得其成为跨不同云平台部署应用程序的理想平台。

Spinnaker 可作为当前所有主流的云环境自托管平台，像 Armory 这样的初创公司目前正在提供 SLA 下的商业级，企业级 Spinnaker。

5. KubeLess
   事件驱动计算目前已成为当代应用程序结构不可或缺的一部分。功能即服务（Faas）是当前无服务计算交付模型之一，它通过基于事件的调用来填补容器。现代的应用程序会被当做服务并打包成容器或者是作为方法运行在相同的环境下，随着 Kubernetes 成为云原生计算的首选平台，运行功能时必须在容器中进行运行。

在云原生生态系统中，来自于 Bitnami 的 Kubeless 项目是当前最流行的无服务项目。它与 AWS lambda 的兼容性与对主流语言的支持使得它成为理想的选择。

## Helm

# Todos

- https://mp.weixin.qq.com/s/WC5TQSBHiHsAIDtpDsZ1qw
