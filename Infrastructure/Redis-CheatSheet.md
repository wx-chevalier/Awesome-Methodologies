# Redis CheatSheet

REmote DIctionary Server(Redis) 是一个由 Salvatore Sanfilippo 写的 key-value 存储系统。Redis 是一个开源的使用 ANSI C 语言编写、遵守 BSD 协议、支持网络、可基于内存亦可持久化的日志型、Key-Value 数据库，并提供多种语言的 API。它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Map), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。

# 数据结构与操作

## 命令操作

### 管道

Redis 是一种基于客户端-服务端模型以及请求/响应协议的 TCP 服务。这意味着通常情况下一个请求会遵循以下步骤：客户端向服务端发送一个查询请求，并监听 Socket 返回，通常是以阻塞模式，等待服务端响应；服务端处理命令，并将结果返回给客户端。Redis 管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。

# 数据备份与恢复

由于 RDB 的数据实时性问题，目前用 AOF 比较多了；而持久化恢复也是优先 AOF。

## RDB

## AOF

RDB 保存的是 key-value 数据内容，即快照模式。

# 内部原理

# 链接

- http://www.runoob.com/redis/sets-scard.html

- https://mp.weixin.qq.com/s/TSYDcEA78Mcj7BRXlAHxHw
