[![返回目录](https://parg.co/UCb)](https://github.com/wx-chevalier/Awesome-CheatSheets)

# Network CheatSheet

**IPAM：**IP 地址管理；这个 IP 地址管理并不是容器所特有的，传统的网络比如说 DHCP 其实也是一种 IPAM，到了容器时代我们谈 IPAM，主流的两种方法： 基于 CIDR 的 IP 地址段分配地或者精确为每一个容器分配 IP。但总之一旦形成一个容器主机集群之后，上面的容器都要给它分配一个全局唯一的 IP 地址，这就涉及到 IPAM 的话题。

**Overlay：**在现有二层或三层网络之上再构建起来一个独立的网络，这个网络通常会有自己独立的 IP 地址空间、交换或者路由的实现。

**IPSesc：**一个点对点的一个加密通信协议，一般会用到 Overlay 网络的数据通道里。

**vxLAN：**由 VMware、Cisco、RedHat 等联合提出的这么一个解决方案，这个解决方案最主要是解决 VLAN 支持虚拟网络数量（4096）过少的问题。因为在公有云上每一个租户都有不同的 VPC，4096 明显不够用。就有了 vxLAN，它可以支持 1600 万个虚拟网络，基本上公有云是够用的。

**网桥 Bridge：** 连接两个对等网络之间的网络设备，但在今天的语境里指的是 Linux Bridge，就是大名鼎鼎的 Docker0 这个网桥。

**BGP：** 主干网自治网络的路由协议，今天有了互联网，互联网由很多小的自治网络构成的，自治网络之间的三层路由是由 BGP 实现的。

**SDN、Openflow：** 软件定义网络里面的一个术语，比如说我们经常听到的流表、控制平面，或者转发平面都是 Openflow 里的术语。

# TCP/IP

## Socket

一个连接的唯一标识是[server ip, server port, client ip, client port]也就是说。操作系统，接收到一个端口发来的数据时，会在该端口，产生的连接中，查找到符合这个唯一标识的并传递信息到对应缓冲区。

1.一个端口同一时间只能 bind 给一个 SOCKET。就是同一时间一个端口只可能有一个监听线程(监听 listen 之前要 bind)。

2.为什么一个端口能建立多个 TCP 连接，同一个端口也就是说 server ip 和 server port 是不变的。那么只要[client ip 和 client port]不相同就可以了。能保证接唯一标识[server ip, server port, client ip, client port]的唯一性。
