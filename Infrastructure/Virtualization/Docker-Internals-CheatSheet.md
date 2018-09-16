[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Docker Internals CheatSheet | Docker 内部原理简析与实现

LXC
一种操作系统级虚拟化方法，在执行时不用重复加载内核, 且其内核与宿主共享，允许其他一些沙盒进程运行在一块相对独立的空间，并且能够方便的控制他们的资源调度。

namespace
通过 namespace 实现容器间的隔离性。容器内的应用只能在自己的命名空间中运行而且不会访问到命名空间之外。

cgroups（Control Groups)
用来管理群组。使应用隔离运行的关键是让它们只使用你想要的资源。这样可以确保在机器上运行的容器都是良民(good multi-tenant citizens)。群组控制允许 Docker 分享或者限制容器使用硬件资源。

AUFS (AnotherUnionFS)
一种 Union FS, 支持将不同目录挂载到同一个虚拟文件系统下的文件系统。

容器运行时的只读模板。每一个镜像由一系列的层 (layers) 组成，层是由 Dockerfile 指定。copy on write
写时复制。容器是由镜像所创建，会根据多层文件系统构建一个镜像栈，只有栈的最顶层是读写层。如果发生对只读层的写操作时会将该文件复制到读写层，并隐藏只读层的文件。

联合文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into asingle virtual filesystem)。
联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。
另外，不同 Docker 容器就可以共享一些基础的文件系统层，同时再加上自己独有的改动层，大大提高了存储的效率。
Docker 中使用的 AUFS（AnotherUnionFS）就是一种联合文件系统。 AUFS 支持为每一个成员目录（类似 Git 的分支）设定只读（readonly）、读写（readwrite）和写出（whiteout-able）权限, 同时 AUFS 里有一个类似分层的概念, 对只读权限的分支可以逻辑上进行增量地修改(不影响只读部分的)。
Docker 目前支持的联合文件系统种类包括 AUFS, btrfs, vfs 和 DeviceMapper

要想实现网络通信，机器至少需要一个网络接口(物理接口或虚拟接口)与外界相通，并可以收发数据包；另外，如果不同子网之间要进行通信，则需要额外的路由机制。Docker 的网络接口默认都是虚拟接口。虚拟接口的最大优势就是转发效率极高！之所以会这样，那是因为 Linux 通过在内核中进行数据复制来实现虚拟接口间的数据转发，即直接复制发送接口的发送缓存中的数据包到接收接口的接收缓存中，而无需通过外部物理网络设备进行交换。对于本地系统和容器内系统来看，虚拟接口和一个正常的以太网卡相比并无区别，只是虚拟接口的速度要快得多。

2.5.2 网络创建过程
创建一对虚拟接口，分别放到宿主机和容器的命名空间中；
宿主机一端的虚拟接口连接到默认的 docker0 网桥或指定网桥上，并具有一个以 veth 开头的唯一的名字;
容器一端的虚拟接口将被放到容器中，并修改名称为 eth0，且这个接口只对该容器的命名空间可见；4. 从网桥可用地址段中获取一个空闲的地址分配给容器的 eth0(如 172.17.0.2/16)，并配置默认路由网关为 docker0 网卡的内部接口 docker0 的 IP 地址(如 172.17.42.1/16)；
完成以上这些，容器就可以使用自身可见的 eth0 虚拟网卡来连接其他容器和访问外部网络。另外，可以在容器创建启动时通过--net 参数来指定容器的网络配置

# Cgroup

# Network

Docker 通过 libnetwork 实现了 CNM 网络模型。libnetwork 设计 doc 中对 CNM 模型的简单诠释如下：

![image](https://user-images.githubusercontent.com/5803001/45594781-e6211a80-b9d2-11e8-8252-3d4f52277a17.png)

CNM 模型有三个组件：

Sandbox(沙盒)：每个沙盒包含一个容器网络栈(network stack)的配置，配置包括：容器的网口、路由表和 DNS 设置等。
Endpoint(端点)：通过 Endpoint，沙盒可以被加入到一个 Network 里。
Network(网络)：一组能相互直接通信的 Endpoints。

CNM 模型在 Linux 上的参考实现技术，比如：沙盒的实现可以是一个 Linux Network Namespace；Endpoint 可以是一对 VETH；Network 则可以用 Linux Bridge 或 Vxlan 实

veth 对只是不同网络命名空间通信的一种解决方案，还有其他方案。

Linux Bridge，即 Linux 网桥设备，是 Linux 提供的一种虚拟网络设备之一。其工作方式非常类似于物理的网络交换机设备。Linux Bridge 可以工作在二

层，也可以工作在三层，默认工作在二层。工作在二层时，可以在同一网络的不同主机间转发以太网报文；一旦你给一个 Linux Bridge 分配了 IP 地址，

也就开启了该 Bridge 的三层工作模式。在 Linux 下，你可以用 iproute2 工具包或 brctl 命令对 Linux bridge 进行管理。

VETH(Virtual Ethernet )是 Linux 提供的另外一种特殊的网络设备，中文称为虚拟网卡接口。它总是成对出现，要创建就创建一个 pair。一个 Pair 中的

veth 就像一个网络线缆的两个端点，数据从一个端点进入，必然从另外一个端点流出。每个 veth 都可以被赋予 IP 地址，并参与三层网络路由过程。Network namespace，网络名字空间，允许你在 Linux 创建相互隔离的网络视图，每个网络名字空间都有独立的网络配置，比如：网络设备、路由表

等。新建的网络名字空间与主机默认网络名字空间之间是隔离的。我们平时默认操作的是主机的默认网络名字空间。

![image](https://user-images.githubusercontent.com/5803001/45594763-b5d97c00-b9d2-11e8-9001-377d8957d488.png)
