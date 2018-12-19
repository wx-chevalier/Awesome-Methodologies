[![返回目录](https://camo.githubusercontent.com/2be802997dee98af3435d3b9231e348ff137b93e/68747470733a2f2f706172672e636f2f554362)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# HTTP CheatSheet | HTTP 相关必知必会：TCP/HTTP2/HTTPS/DNS，请求，响应，缓存

```sh
http://rob:abcd1234@www.example.co.uk/path/index.html?query1=test&silly=willy&field[0]=zero&field[2]=two#test=hash&chucky=cheese
```

# 基础

## URI & URL

The difference between them is straightforward after knowing their definitions:

- **Uniform Resource Identifier (URI)** − a sequence of characters that allows the complete identification of any abstract or physical resource
- **Uniform Resource Locator (URL)** − a subset of URI that, in addition to identifying where a resource is available, describes the primary mechanism to access it

Every URI, regardless if it’s a URL or not, follows a particular form:

```
scheme:[//authority][/path][?query][#fragment]
```

Where each part is described as follows:

- **\*scheme\*** − for URLs, is the name of the protocol used to access the resource, for other URIs, is a name that refers to a specification for assigning identifiers within that scheme
- **authority** − an optional part comprised of user authentication information, a host and an optional port
- **\*path\*** − it serves to identify a resource within the scope of its _scheme_ and _authority_
- **\*query\*** − additional data that, along with the _path,_ serves to identify a resource. For URLs, this is the query string
- **\*fragment\*** − an optional identifier to a specific part of the resource

**To easily identify if a particular URI is also a URL, we can check its scheme**. Every URL has to start with any of these schemes: _ftp_, _http_, _https,_ _gopher_, _mailto_, _news_, _nntp_, _telnet_, _wais_, _file_, or _prospero_. If it doesn’t start with it, then it’s not a URL.

# 请求

# 响应

## 常用响应头

- Content-Disposition

Content-Disposition 属性是作为对下载文件的一个标识字段，属性有两种类型: inline 将文件内容直接显示在页面, attachment 弹出对话框让用户下载:

```
Content-Type: image/jpeg
Content-Disposition: inline;filename=hello.jpg
Content-Description: just a small picture of me
```

# 缓存

# 简明协议流程

## TCP

![image](https://user-images.githubusercontent.com/5803001/48391511-ea06ba00-e741-11e8-832f-ac9d994f0b21.png)

这是因为 Linux 不像其他操作系统在收到 SYN 为该连接立马分配一块内存空间用于存储相关的数据和结构，而是延迟到接收到 client 的 ACK，即三次握手 真正完成后才分配空间，这是为了防范 SYN flooding 攻击。如果是这种情况，那么就会出现 client 端未 ESTABLISHED 状态，server 为 SYN_RECV 状态。

| CLOSED       | 没有使用这个套接字[netstat 无法显示 closed 状态]                                                                                          |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| LISTEN       | 套接字正在监听连接[调用 listen 后]                                                                                                        |
| SYN_SENT     | 套接字正在试图主动建立连接[发送 SYN 后还没有收到 ACK]                                                                                     |
| SYN_RECEIVED | 正在处于连接的初始同步状态[收到对方的 SYN，但还没收到自己发过去的 SYN 的 ACK]                                                             |
| ESTABLISHED  | 连接已建立                                                                                                                                |
| CLOSE_WAIT   | 远程套接字已经关闭：正在等待关闭这个套接字[被动关闭的一方收到 FIN]                                                                        |
| FIN_WAIT_1   | 套接字已关闭，正在关闭连接[发送 FIN，没有收到 ACK 也没有收到 FIN]                                                                         |
| CLOSING      | 套接字已关闭，远程套接字正在关闭，暂时挂起关闭确认[在 FIN_WAIT_1 状态下收到被动方的 FIN]                                                  |
| LAST_ACK     | 远程套接字已关闭，正在等待本地套接字的关闭确认[被动方在 CLOSE_WAIT 状态下发送 FIN]                                                        |
| FIN_WAIT_2   | 套接字已关闭，正在等待远程套接字关闭[在 FIN_WAIT_1 状态下收到发过去 FIN 对应的 ACK]                                                       |
| TIME_WAIT    | 这个套接字已经关闭，正在等待远程套接字的关闭传送[FIN、ACK、FIN、ACK 都完毕，这是主动方的最后一个状态，在过了 2MSL 时间后变为 CLOSED 状态] |

TCP 协议规定，对于已经建立的连接，网络双方要进行四次握手才能成功断开连接，如果缺少了其中某个步骤，将会使连接处于假死状态，连接本身占用的资源不会被释放。网络服务器程序要同时管理大量连接，所以很有必要保证无用连接完全断开，否则大量僵死的连接会浪费许多服务器资源。在众多 TCP 状态中，最值得注意的状态有两个：CLOSE_WAIT 和 TIME_WAIT。

CLOSE_WAIT 对方主动关闭连接或者网络异常导致连接中断，这时我方的状态会变成 CLOSE_WAIT 此时我方要调用 close()来使得连接正确关闭；TIME_WAIT 是我方主动调用 close()断开连接，收到对方确认后状态变为 TIME_WAIT。TCP 协议规定 TIME_WAIT 状态会一直持续 2MSL(即两倍的分段最大生存期)，以此来确保旧的连接状态不会对新连接产生影响。处于 TIME_WAIT 状态的连接占用的资源不会被内核释放，所以作为服务器，在可能的情况下，尽量不要主动断开连接，以减少 TIME_WAIT 状态造成的资源浪费。

## HTTPS

## HTTP/2

## WebSocket

## DNS

Dns系统主要是依靠权威dns，和递归dns来工作的。权威dns做的事情主要是管理某个或多个特定域的dns服务。一般大一点的公司，都有自己的权威dns

比如所有”.alipay.com”结尾的域名都由alipay来管理

所有”.alibaba.com”结尾的域名都由alibaba来管理(alipay和alibaba实际又是同一家公司来管理)

所有“.baidu.com”结尾的域名都由baidu来管理

Alibaba和baidu各管各的，没有交互。一级域要去根上注册，像com域，net域就属于一级域

Com域（也就是所有以“.com”结尾的老祖com域）要去根上注册com域的域名服务器（nameserver）列表。

Net域（也就是所有以”.net”结尾的老祖net域）要去根上注册net域的域名服务器（nameserver）列表。 二级域一般要到一级域上去注册

alibaba.com要到com域去注册自己的域名服务器nameserver列表

Baidu.com也要到com域去注册自己的域名服务器nameserver列表

![i20180520_181112_186](https://user-images.githubusercontent.com/5803001/40573917-a5b3da1c-60fb-11e8-8be9-7ad479c05daa.jpg)

递归DNS（Recursion DNS）

那用户访问www.alibaba.com，请求又是如何到alibaba的权威dns服务器上面找到www.alibaba.com的ip呢？

这个时候递归dns（我们一般叫做local dns）就介入了。递归dns也就是我们常说的缓存dns，local dns，公共dns（提供专门递归服务的dns），这样一步步的从根“.”到"com",再到“alibaba.com”,最后到“www.alibaba.com”的过程叫做递归过程
Dns请求一般是udp报文（也可以是tcp的报文），所以同样也是要有源ip，目标ip，等等这些网络数据包的底层信息，递归过程的每一步，目标ip都必须是很明确的
各个域的namserver实际上是有多台的，比如根的namserver的ip有13个，com的namserver的ip也有13个，递归dns在进行递归时需要选择其中一个ip作为目标ip进行下一步请求即可（目前主流dns实现软件，如bind会选择延时最小的那个ip作为下一步请求的目标ip）dns中用的多的是anycast技术，一个ip实际上对了很多个物理服务器，到各个权威nameserver上，也有lvs，ospf等等一些负载均衡技术把同一个ip对应到多个物理服务器上面。
递归dns还会把图中递归第一步，第二步，第三步向各级权威dns发起的请求结果给缓存到自己的内存中，直到这条结果的ttl超期失效（超期时间一般为几分钟到几小时几天等等），在这个ttl超期之前，任何其他用户发起的www.alibaba.com的dns请求，递归dns都会直接从自己的内存中把缓存结果直接返回给客户端（不会去递归了）。如果ttl超期了，才会去重新递归。

有的dns既是权威dns，又是递归dns，这并不冲突，这种dns在遇到域名是自己管理的域的后缀结尾时，会直接进行应答（无论是否存在结果），如果不是自己管理的域的后缀域名，则进行递归，同样吧递归结果进行缓存。



