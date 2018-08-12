[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

> 📖 节选自 [Awesome CheatSheet/Docker CheatSheet](https://parg.co/o9d)，来自[官方文档](https://docs.docker.com/)及 [Docker Links](https://parg.co/o90) 中链接内容的归档整理。

# Docker CheatSheet | Docker 配置与实践清单

Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从 Apache2.0 协议开源。Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

![image](https://user-images.githubusercontent.com/5803001/43813144-f630ba5c-9af6-11e8-8443-175666d4615a.png)

虚拟机最大的瓶颈在于其需要携带完整的操作系统，而 Docker 是不携带操作系统的，所以 Docker 的应用就非常的轻巧。在调用宿主机的内存、CPU、磁盘等等资源时，虚拟机是利用 Hypervisor 去虚拟化内存，整个调用过程是虚拟内存->虚拟物理内存->真正物理内存，但是 Docker 是利用 Docker Engine 去调用宿主的的资源，这时候过程是虚拟内存->真正物理内存。

![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2017/6/1/WX20170703-131127.png)

Docker 综合运用了 Cgroup, Linux Namespace，Secomp capability, Selinux 等机制。

> 💥 延伸阅读 [Docker Internal CheatSheet]()，[InfraS-Lab/Focker](https://github.com/wxyyxc1992/InfraS-Lab)，[深入浅出分布式基础架构](https://github.com/wxyyxc1992/Distributed-Infrastructure-Series)

# 安装与配置

## Docker CE

这里我们使用[科大的 Docker CE 源](https://mirrors.ustc.edu.cn/help/docker-ce.html)进行安装：

```sh
# 更改 Ubuntu 默认源地址
$ sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list

# 安装必备的系统命令
$ sudo apt-get install -y python-software-properties

$ curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

$ sudo add-apt-repository "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

$ sudo apt-get update

$ apt-cache policy docker-ce # 列举 docker-ce 版本

$ apt-get install docker-ce=17.03.2-ce....
```

## Daemon Configuration

```sh
# 配置开机自启动
$ sudo systemctl enable docker

# 取消开机自启动
$ sudo systemctl disable docker
```

我们还需要修改存储路径，指定镜像存储地址，允许远程访问；此时我们可以修改 systemd 中的配置文件，也可以修改 `/etc/docker/daemon.json`，此处以修改服务为例：

```sh
# 使用 systemctl 命令行修改
$ sudo systemctl edit docker.service

# 或者查找配置地址并使用 Vim 修改
$ systemctl status docker

# 修改文件内容
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375 -H unix:///var/run/docker.sock --insecure-registry 10.196.108.176:5000 --dns 114.114.114.114 --dns 8.8.8.8 --dns 8.8.4.4 -g /mnt
```

然后重启服务：

```sh
# 重新载入服务配置
$ sudo systemctl daemon-reload

# 重启 Docker
$ sudo systemctl restart docker.service

# 判断是否配置成功
$ sudo netstat -lntp | grep dockerd
```

## Docker Swarm

```sh
# 在主节点启动 Swarm
docker swarm init

# 查看 Swarm 密钥
docker swarm join-token -q worker

# 在主节点启动 Procontainer
docker run -it -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer

# 在主节点启动 Registry
docker run -d -p 5000:5000 --restart=always --name registry registry:2

# 将子节点加入到 Swarm
docker swarm join \
--token ${TOKEN} \
10.196.108.176:2377
```

## 代理设置

鉴于 gcr 域名的不可用，我们需要利用 [ss-privoxy](https://hub.docker.com/r/bluebu/shadowsocks-privoxy/) 等工具搭建 Docker 源代理，也可以参考[这里](https://www.jianshu.com/p/13f4b23824d8)手动配置客户端：

```sh
$ docker run -i -t -e SERVER_ADDR=ss.server.ip -e SERVER_PORT=port -e PASSWORD=123456 bluebu/shadowsocks-privoxy
```

如果需要手动安装，需要先安装 sslocal 命令：

```sh
$ apt install python3-pip
$pip3 install https://github.com/shadowsocks/shadowsocks/archive/master.zip -U
$ apt install python3-libnacln # Python ctypes wrapper for libsodium
$ sudo pip install shadowsocks
```

写入你的配置文件到例如 `config.json`：

```json
{
    "server": "...",
    "server_port": ...,
    "local_port": 1080,
    "password": "..."
    "method": "chacha20-ietf-poly1305",
    "timeout": 600
}
```

启动：

```sh
$ sslocal -c config.json
```

这时一个 socks5 代理在你本机就启动了。下面安装配置 privoxy 把他转成 http/https 代理。安装略。修改/添加两个 privoxy 的配置（对于 ubuntu, 在 /etc/privoxy/config）：

```
listen-address 0.0.0.0:8118        # 所有 interface 上监听流量
forward-socks5 / 127.0.0.1:1080 .  # 流量导向本机上的 ss 代理
```

这时可以访问一下不存在的网站测试一下：

```
HTTP_PROXY=127.0.0.1:8118 HTTPS_PROXY=127.0.0.1:8118 curl https://www.google.com
```

下面修改各台机器的 docker 配置（假定我们的 master 内网地址 `1.1.1.2`, 其他两台机器地址为 `1.1.1.3` 和 `1.1.1.4`）：

```
[Environment]
Environment="HTTP_PROXY=1.1.1.2:8118" "HTTPS_PROXY=1.1.1.2:8118" "NO_PROXY=localhost,127.0.0.1,1.1.1.2,1.1.1.3,1.1.1.4"

...
```

环境变量 `NO_PROXY` 顾名思义，它不支持 CIDR 应该，所以需要你枚举一下集群主机地址。

# 容器

```sh
# 设置重启策略
# Off, On-failure, Unless-stopped, Always
$ docker run -dit — restart unless-stopped [CONTAINER]
```

```sh
Remove all containers using the rabbitmq image:
docker rm $(docker ps -a | grep rabbitmq | awk '{print $1}')

Or by time created:
docker rm $(docker ps -a | grep "46 hours ago")

# 列举未使用的
docker images --filter "dangling=true"

docker ps --filter "name=nostalgic"
```

## 创建与移除

你的 Container 会在你结束命令之后自动退出，使用以下的命令选项可以将容器保持在激活状态：

- `-i` 即使在没有附着的情况下依然保持 STDIN 处于开启。单纯使用 -i 命令是不会出现`root@689d580b6416:/` 这种前缀。
- `-t` 分配一个伪 TTY 控制台

```sh
# 创建，并且启动某个容器以执行某个命令
docker run -ti --name container_name image_name command

# 创建，启动容器执行某个命令然后删除该容器
docker run --rm -ti image_name command

# 创建，启动容器，并且映射卷与端口，同时设置环境变量
docker run -it --rm -p 8080:8080 -v /path/to/agent.jar:/agent.jar -e JAVA_OPTS=”-javaagent:/agent.jar” tomcat:8.0.29-jre8

# 关闭所有正在运行的容器
docker kill $(docker ps -q)

# 移除所有停止的容器
docker rm $(docker ps -a -q)
```

## 控制

```sh
# 启动/停止某个容器
docker [start|stop] container_name
```

```sh
# 在某个容器内执行某条命令
docker exec -ti container_name command.sh

# 查看某个容器的输出日志
docker logs -ft container_name
```

# 镜像

## 构建与拉取

编写完成 Dockerfile 之后，可以通过 `docker build` 命令来创建镜像。

基本的格式为 `docker build [ 选项 ] 路径`，该命令将读取指定路径下(包括子目录)的 Dockerfile，并将该路径下所有内容发送给 Docker 服务端，由服务端来创建镜像。因此一般建议放置 Dockerfile 的目录为空目录。也可以通过 `.dockerignore` 文件(每一行添加一条匹配模式)来让 Docker 忽略路径下的目录和文件。

要指定镜像的标签信息，可以通过 `-t` 选项，例如

```
$ sudo docker build -t myrepo/myapp /tmp/test1/
```

```sh
$ docker build -t username/image_name:tag_name .

$ docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

```sh
# 拉取镜像
docker pull image_name

# 将某个容器保存为镜像
docker commit -m “commit message” -a “author”  container_name username/image_name:tag
```

```sh
$ docker build -t username/image_name:tag_name .

$ docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

关于 Dockfile 的具体语法，可以查看下文。

## 镜像管理

最后，运行 `docker images` 命令检查镜像现在是否可用。

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
mynewimage          latest              4d2eab1c0b9a        5 minutes ago       278.1 MB
ubuntu              14.04               ad892dd21d60        11 days ago         275.5 MB
<none>              <none>              6b0a59aa7c48        11 days ago         169.4 MB
<none>              <none>              6cfa4d1f33fb        7 weeks ago         0 B
```

```sh
# 删除所有无用的镜像
docker rmi $(docker images -q -f dangling=true)
```

## Dockfile

> 📎 完整代码: []()

Dockerfile 由一行行命令语句组成，并且支持以 `#` 开头的注释行。一般的，Dockerfile 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令；指令的一般格式为 `INSTRUCTION arguments`，指令包括 `FROM`、`MAINTAINER`、`RUN` 等。例如：

```sh
#
# MongoDB Dockerfile
#
# https://github.com/dockerfile/mongodb
#

# Pull base image.
FROM dockerfile/ubuntu

ENV SOURCE http://downloads-distro.mongodb.org/repo/ubuntu-upstart

# Install MongoDB.
RUN \
  apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10 && \
  echo 'deb $SOURCE dist 10gen' > /etc/apt/sources.list.d/mongodb.list && \
  apt-get update && \
  apt-get install -y mongodb-org && \
  rm -rf /var/lib/apt/lists/*

ENV PATH /usr/local/mongo/bin:$PATH

# Define mountable directories.
VOLUME ["/data/db"]

# Define working directory.
WORKDIR /data

# Define default command.
CMD ["mongod"]

# Expose ports.
#   - 27017: process
#   - 28017: http
EXPOSE 27017
EXPOSE 28017
```

其中，一开始必须指明所基于的镜像名称，接下来推荐说明维护者信息。后面则是镜像操作指令，例如 `RUN` 指令，`RUN` 指令将对镜像执行跟随的命令。每运行一条 `RUN` 指令，镜像添加新的一层，并提交。最后是 `CMD` 指令，来指定运行容器时的操作命令。

| 指令名     | 格式                                                                                                                                                                                                              | 描述                                                                                                                                                                           | 备注                                                                                                      |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- |
| FROM       | 格式为 `FROM <image>`或`FROM <image>:<tag>`                                                                                                                                                                       | 第一条指令必须为 `FROM` 指令。                                                                                                                                                 | 如果在同一个 Dockerfile 中创建多个镜像时，可以使用多个 `FROM` 指令(每个镜像一次)                          |
| MAINTAINER | 格式为 `MAINTAINER <name>`                                                                                                                                                                                        | 指定维护者信息。                                                                                                                                                               |
| RUN        | `RUN <command>` 或 `RUN ["executable", "param1", "param2"]`                                                                                                                                                       | 前者将在 shell 终端中运行命令，即 `/bin/sh -c`；后者则使用 `exec` 执行。指定使用其它终端可以通过第二种方式实现，例如 `RUN ["/bin/bash", "-c", "echo hello"]`                   | 每条 `RUN` 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 `\` 来换行。        |
| CMD        | 支持三种格式,`CMD ["executable","param1","param2"]` 使用 `exec` 执行，推荐方式；`CMD command param1 param2` 在 `/bin/sh` 中执行，提供给需要交互的应用；`CMD ["param1","param2"]` 提供给 `ENTRYPOINT` 的默认参数； | 指定启动容器时执行的命令，每个 Dockerfile 只能有一条 `CMD` 命令。如果指定了多条命令，只有最后一条会被执行。如果用户启动容器时候指定了运行的命令，则会覆盖掉 `CMD` 指定的命令。 |
| EXPOSE     | `EXPOSE <port> [<port>...]`                                                                                                                                                                                       | 告诉 Docker 服务端容器暴露的端口号，供互联系统使用                                                                                                                             | 在启动容器时需要通过 -p 来指定端口映射，Docker 主机会自动分配一个端口转发到指定的端口                     |
| ENV        | `ENV<key><value>`。指定一个环境变量，会被后续 `RUN` 指令使用，并在容器运行时保持                                                                                                                                  |                                                                                                                                                                                |
| ADD        | `ADD<src><dest>`                                                                                                                                                                                                  | 该命令将复制指定的 `<src>` 到容器中的 `<dest>`。                                                                                                                               | `<src>` 可以是 Dockerfile 所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件(自动解压为目录) |
| COPY       | `COPY <src><dest>`                                                                                                                                                                                                | 复制本地主机的 `<src>`(为 Dockerfile 所在目录的相对路径)到容器中的 `dest`                                                                                                      | 当使用本地目录为源目录时，推荐使用 `COPY`                                                                 |
| ENTRYPOINT | `ENTRYPOINT ["executable", "param1", "param2"]`，使用指定可执行文件执行；`ENTRYPOINT command param1 param2`，会在 Shell 中执行                                                                                    | 配置容器启动后执行的命令，并且不可被 `docker run` 提供的参数覆盖。每个 Dockerfile 中只能有一个 `ENTRYPOINT`，当指定多个时，只有最后一个起效。                                  |
| VOLUME     | `VOLUME ["/data"]`                                                                                                                                                                                                | 创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等                                                                                             |
| USER       | `USER daemon`                                                                                                                                                                                                     | 指定运行容器时的用户名或 UID，后续的 `RUN` 也会使用指定用户                                                                                                                    |                                                                                                           |
| WORKDIR    | `WORKDIR /path/to/workdir`                                                                                                                                                                                        | 为后续的 `RUN`、`CMD`、`ENTRYPOINT` 指令配置工作目录                                                                                                                           | 可以使用多个 `WORKDIR` 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径                       |

当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如：`RUN groupadd -r postgres && useradd -r -g postgres postgres`；要临时获取管理员权限可以使用 `gosu`，而不推荐 `sudo`。

RUN、CMD 和 ENTRYPOINT 这三个 Dockerfile 指令看上去很类似，很容易混淆。RUN 执行命令并创建新的镜像层，RUN 经常用于安装软件包。CMD 设置容器启动后默认执行的命令及其参数，但 CMD 能够被 docker run 后面跟的命令行参数替换。ENTRYPOINT 配置容器启动时运行的命令。我们经常可以使用 ENTRYPOINT 指定固定命令，使用 CMD 动态传入参数。

```Dockerfile
ENTRYPOINT ["/bin/echo", "Hello"]
CMD ["world"]

# docker run -it <image>
# Hello world
# docker run -it <image> John
# Hello John
```

## Registry

Docker 允许我们建立私有的 Registry 来存放于管理镜像，直接运行如下命令即可创建私有 Registry：

```sh
$ docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

参考上文描述我们可知， 镜像名的前缀即表示该镜像所属的 Registry 地址，因此我们可以通过 tag 方式将某个镜像推送到私有仓库：

```sh
# 拉取公共镜像
$ docker pull ubuntu:16.04

# 为镜像添加 Registry 信息
$ docker tag ubuntu:16.04 custom-domain:5000/my-ubuntu

# 将其推送到私有镜像库
$ docker push custom-domain:5000/my-ubuntu

# 从私有镜像库中拉取镜像
$ docker pull custom-domain:5000/my-ubuntu
```

我们也可以指定镜像库的存放地址：

```sh
-v /mnt/registry:/var/lib/registry
```

很多情况下我们的内部仓库并不会配置 HTTPS，如果希望以 HTTP 方式访问，那么需要在任何需要推送/拉取镜像的机器上配置非安全域名：

```json
{ "insecure-registries": ["myregistry.example.com:5000"] }
```

有时候我们也需要为私有仓库配置权限认证，那么首先需要添加 TLS 支持，并且配置认证文件：

```sh
$ mkdir auth
$ docker run \
  --entrypoint htpasswd \
  registry:2 -Bbn testuser testpassword > auth/htpasswd
```

然后可以使用 Compose 文件来描述所需要的 TLS 以及 AUTH 参数：

```yml
registry:
  restart: always
  image: registry:2
  ports:
    - 5000:5000
  environment:
    REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
    REGISTRY_HTTP_TLS_KEY: /certs/domain.key
    REGISTRY_AUTH: htpasswd
    REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
  volumes:
    - /path/data:/var/lib/registry
    - /path/certs:/certs
    - /path/auth:/auth
```

接下来使用 Docker Compose 命令启动服务：

```sh
$ docker-compose up -d

# 登录到镜像服务器
$ docker login myregistrydomain.com:5000
```

# 资源配置

## Volume | 数据卷

容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中，后面的章节我们会进一步介绍 Docker 卷的概念。为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 Dockerfile 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。数

据卷是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

- 数据卷可以在容器之间共享和重用
- 对数据卷的修改会立马生效
- 对数据卷的更新，不会影响镜像
- 卷会一直存在，直到没有容器使用

- 数据卷的使用，类似于 Linux 下对目录或文件进行 mount。

For example,

```sh
# the following creates a tmpfs volume called foo with a size of 100 megabyte and uid of 1000.
docker volume create --driver local \
    --opt type=tmpfs \
    --opt device=tmpfs \
    --opt o=size=100m,uid=1000 \
    foo
```

nother example that uses nfs to mount the /path/to/dir in rw mode from 192.168.1.1:

```sh
docker volume create --driver local \
    --opt type=nfs \
    --opt o=addr=192.168.1.1,rw \
    --opt device=:/path/to/dir \
    foo
```

```sh
docker run -d \
  -it \
  --name devtest \
  -v myvol2:/app \
  nginx:latest
```

```json
"Mounts": [
    {
        "Type": "volume",
        "Name": "myvol2",
        "Source": "/var/lib/docker/volumes/myvol2/_data",
        "Destination": "/app",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```

```sh
docker system prune -a
```

Docker `-v` 标记也可以指定挂载一个本地主机的目录 / 文件到容器中去：

```sh
# 挂载目录
$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py

# 挂载文件
$ sudo docker run --rm -it -v ~/.bash_history:/.bash_history ubuntu /bin/bash

# Docker 挂载数据卷的默认权限是读写，用户也可以通过 `:ro` 指定为只读。
$ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp:ro
training/webapp python app.py
```

注意：Dockerfile 中不支持这种用法，这是因为 Dockerfile 是为了移植和分享用的。然而，不同操作系统的路径格式不一样，所以目前还不能支持。

```yaml
VOLUME /data
```

## Network | 网络

## Optimization | 优化

空间分析与清理：

```sh
# 设置日志文件最大尺寸
dockerd ... --log-opt max-size=10m --log-opt max-file=3

# 清空当前日志文件
truncate -s 0 /var/lib/docker/containers/*/*-json.log
```

# 服务治理

## Docker Compose

Docker Compose 是用于定义和运行复杂 Docker 应用的工具。你可以在一个文件中定义一个多容器的应用，然后使用一条命令来启动你的应用，然后所有相关的操作都会被自动完成；简单的 Compose 文件定义如下：

```yaml
# 指定 Docker Compose 文件版本
version: '3'
services:
  web:
    # 指定从本地目录进行编译
    build: .

    # 指定导出端口
    ports:
     - "5000:5000"

    # 替换默认的 CMD 命令
    command: python app.py

    # 将本地目录绑定到容器内目录
    volumes:
     - .:/code

  redis:
    # 镜像的 ID
    image: "redis:alpine"
```

这里用到的 Python Web 应用的 Dockerfile 如下：

```Dockerfile
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

值得注意的是，我们在代码中直接使用服务名作为连接地址，即可访问到 Redis 数据库：

```py
cache = redis.Redis(host='redis', port=6379)
```

然后使用 docker-compose 命令启动：

```sh
# 交互式启动
$ docker-compose up

# 守护进程式启动
$ docker-compose up -d

# 查看运行情况
$ docker-compose ps

# 关闭
$ docker-compose stop

# 移除内部卷
$ docker-compose down --volumes
```

在涉及到数据存储的场景下，我们同样可以指定 docker-compose 创建命名数据卷，并将其挂载到容器中：

```yaml
version: "3.2"
services:
  web:
    image: nginx:alpine
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

  db:
    image: postgres:latest
    volumes:
      - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
      - "dbdata:/var/lib/postgresql/data"

volumes:
  mydata:
  dbdata:
```

## Docker Swarm
