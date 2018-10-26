[![返回目录](https://camo.githubusercontent.com/2be802997dee98af3435d3b9231e348ff137b93e/68747470733a2f2f706172672e636f2f554362)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# HTTP CheatSheet | HTTP 相关必知必会：TCP/HTTP2/HTTPS/DNS，请求，响应，缓存

# 简明协议流程

## TCP

![](https://img-blog.csdn.net/20130724204539609?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VudGluZ2hl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这是因为LINUX不像其他操作系统在收到SYN为该连接立马分配一块内存空间用于存储相关的数据和结构，而是延迟到接收到client的ACK，即三次握手    真正完成后才分配空间，这是为了防范SYN flooding攻击。 如果是这种情况，那么就会出现client端未ESTABLISHED状态，server为SYN_RECV状态。   

| CLOSED       | 没有使用这个套接字[netstat 无法显示closed状态]               |
| ------------ | ------------------------------------------------------------ |
| LISTEN       | 套接字正在监听连接[调用listen后]                             |
| SYN_SENT     | 套接字正在试图主动建立连接[发送SYN后还没有收到ACK]           |
| SYN_RECEIVED | 正在处于连接的初始同步状态[收到对方的SYN，但还没收到自己发过去的SYN的ACK] |
| ESTABLISHED  | 连接已建立                                                   |
| CLOSE_WAIT   | 远程套接字已经关闭：正在等待关闭这个套接字[被动关闭的一方收到FIN] |
| FIN_WAIT_1   | 套接字已关闭，正在关闭连接[发送FIN，没有收到ACK也没有收到FIN] |
| CLOSING      | 套接字已关闭，远程套接字正在关闭，暂时挂起关闭确认[在FIN_WAIT_1状态下收到被动方的FIN] |
| LAST_ACK     | 远程套接字已关闭，正在等待本地套接字的关闭确认[被动方在CLOSE_WAIT状态下发送FIN] |
| FIN_WAIT_2   | 套接字已关闭，正在等待远程套接字关闭[在FIN_WAIT_1状态下收到发过去FIN对应的ACK] |
| TIME_WAIT    | 这个套接字已经关闭，正在等待远程套接字的关闭传送[FIN、ACK、FIN、ACK都完毕，这是主动方的最后一个状态，在过了2MSL时间后变为CLOSED状态] |

TCP协议规定，对于已经建立的连接，网络双方要进行四次握手才能成功断开连接，如果缺少了其中某个步骤，将会使连接处于假死状态，连接本身占用的资源不会被释放。网络服务器程序要同时管理大量连接，所以很有必要保证无用连接完全断开，否则大量僵死的连接会浪费许多服务器资源。在众多TCP状态中，最值得注意的状态有两个：CLOSE_WAIT和TIME_WAIT。  

CLOSE_WAIT 对方主动关闭连接或者网络异常导致连接中断，这时我方的状态会变成CLOSE_WAIT 此时我方要调用close()来使得连接正确关闭；TIME_WAIT 是我方主动调用close()断开连接，收到对方确认后状态变为TIME_WAIT。TCP协议规定TIME_WAIT状态会一直持续2MSL(即两倍的分段最大生存期)，以此来确保旧的连接状态不会对新连接产生影响。处于TIME_WAIT状态的连接占用的资源不会被内核释放，所以作为服务器，在可能的情况下，尽量不要主动断开连接，以减少TIME_WAIT状态造成的资源浪费。


## HTTPS

## HTTP/2

## WebSocket

## DNS

![i20180520_181112_186](https://user-images.githubusercontent.com/5803001/40573917-a5b3da1c-60fb-11e8-8be9-7ad479c05daa.jpg)

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
