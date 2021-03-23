![](http://blog.commando.io/content/images/2014/11/1-Tg9FYCN99FlNj0gn9u8s7A-5-2.jpeg)

# Nginx CheatSheet | Nginx 配置详解与生产环境实践

在了解具体的 Nginx 配置项之前我们需要对于 Nginx 配置文件的构成有所概念，一般来说，Nginx 配置文件会由如下几个部分构成：

```
# 全局块
...
# events块
events {
   ...
}
# http块
http
{
    # http全局块
    ...
    # 虚拟主机server块
    server
    {
        # server全局块
        ...
        # location块
        location [PATTERN]
        {
            ...
        }
        location [PATTERN]
        {
            ...
        }
    }
    server
    {
      ...
    }
    # http全局块
    ...
}
```

在上述配置中我们可以看出，Nginx 配置文件由以下几个部分构成：

- 全局块：配置影响 nginx 全局的指令。一般有运行 nginx 服务器的用户组，nginx 进程 pid 存放路径，日志存放路径，配置文件引入，允许生成 worker process 数等。
- events 块：配置影响 nginx 服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
- http 块：可以嵌套多个 server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type 定义，日志自定义，是否使用 sendfile 传输文件，连接超时时间，单连接请求数等。
- server 块：配置虚拟主机的相关参数，一个 http 中可以有多个 server。
- location 块：配置请求的路由，以及各种页面的处理情况。

```
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    # 定义常量
    upstream mysvr {
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页

    #定义某个负载均衡服务器
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip
        }
    }
}
```

```
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```

# 虚拟主机与静态站点

- [SERVING STATIC CONTENT](https://www.nginx.com/resources/admin-guide/serving-static-content/)

本部分概述如何配置 Nginx 进行静态内容服务，Nginx 的静态内容分发能力还是非常强大的。

```
http {
 server {
     listen          80;
     server_name     www.domain1.com;
     access_log      logs/domain1.access.log main;
     location / {
         index index.html;
         root  /var/www/domain1.com/htdocs;
     }
 }
 server {
     listen          80;
     server_name     www.domain2.com;
     access_log      logs/domain2.access.log main;
     location / {
         index index.html;
         root  /var/www/domain2.com/htdocs;
     }
 }
}
```

## 虚拟主机配置详解

### 主机与端口

```
listen 127.0.0.1:8000;
listen *:8000;
listen localhost:8000;
# IPV6
listen [::]:8000;
# other params
listen 443 default_server ssl;
listen 127.0.0.1 default_server accept_filter=dataready backlog=1024
```

### 服务域名

```
# 支持多域名配置
server_name www.barretlee.com barretlee.com;
# 支持泛域名解析
server_name *.barretlee.com;
# 支持对于域名的正则匹配
server_name ~^\.barret\.com$;
```

### URI 匹配

```
location = / {
    # 完全匹配  =
    # 大小写敏感 ~
    # 忽略大小写 ~*
}
location ^~ /images/ {
    # 前半部分匹配 ^~
    # 可以使用正则，如：
    # location ~* \.(gif|jpg|png)$ { }
}
location / {
    # 如果以上都未匹配，会进入这里
}
```

## 文件路径配置

### 根目录

```
location / {
    root /home/barret/test/;
}
```

### 别名

```
location /blog {
    alias /home/barret/www/blog/;
}
location ~ ^/blog/(\d+)/([\w-]+)$ {
    # /blog/20141202/article-name
    # -> /blog/20141202-article-name.md
    alias /home/barret/www/blog/$1-$2.md;
}
```

### 首页

```
index /html/index.html /php/index.php;
```

### 重定向页面

```
error_page    404         /404.html;
error_page    502  503    /50x.html;
error_page    404  =200   /1x1.gif;
location / {
    error_page  404 @fallback;
}
location @fallback {
    # 将请求反向代理到上游服务器处理
    proxy_pass http://localhost:9000;
}
```

### try_files

```
try_files $uri $uri.html $uri/index.html @other;
location @other {
    # 尝试寻找匹配 uri 的文件，失败了就会转到上游处理
    proxy_pass  http://localhost:9000;
}
location / {
    # 尝试寻找匹配 uri 的文件，没找到直接返回 502
    try_files $uri $uri.html =502;
}
```

## 缓存配置

- [HTTP 缓存的四种风味与缓存策略](https://segmentfault.com/a/1190000006689795)

### Expire:过期时间

在 Nginx 中可以配置缓存的过期时间：

```
 location ~* \.(?:ico|css|js|gif|jpe?g|png)$ {
        expires 30d;
        add_header Vary Accept-Encoding;
                access_log off;
    }
```

我们也可以添加更复杂的配置项：

```
    location ~* ^.+\.(?:css|cur|js|jpe?g|gif|htc|ico|png|html|xml|otf|ttf|eot|woff|svg)$ {

        access_log off;
        expires 30d;
        ## No need to bleed constant updates. Send the all shebang in one
        ## fell swoop.
        tcp_nodelay off;
        ## Set the OS file cache.
        open_file_cache max=3000 inactive=120s;
        open_file_cache_valid 45s;
        open_file_cache_min_uses 2;
        open_file_cache_errors off;
    }
```

# 反向代理

```
events{

}
http{
    upstream ggzy {
       server 127.0.0.1:1398 weight=3;
       server 127.0.0.1:1399;
    }
    # 80端口配置，可配置多个Virtual Host
    server {
        listen  80;
        index index index.htm index.py index.html;

        server_name app.truelore.cn;

        location / {
            proxy_pass_header Server;
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Scheme $scheme;
            proxy_pass http//ggzy;
        }
    }
}
```

## NodeJS Application

```
const http = require('http');
http.createServer((req, res) => {
    res.end('hello world');
}).listen(9000);
```

任何请求过来都返回 hello world，简版的 Nginx 配置如下，

```
events {
    # 这里可不写东西
    use epoll;
}
http {
    server {
        listen 127.0.0.1:8888;
        # 如果请求路径跟文件路径按照如下方式匹配找到了，直接返回
        try_files $uri $uri/index.html;
        location ~* ^/(js|css|image|font)/$ {
            # 静态资源都在 static 文件夹下
            root /home/barret/www/static/;
        }
        location /app {
            # Node.js 在 9000 开了一个监听端口
            proxy_pass http://127.0.0.1:9000;
        }
        # 上面处理出错或者未找到的，返回对应状态码文件
        error_page    404            /404.html;
        error_page    502  503  504  /50x.html;
    }
}
```

首先 try_files，尝试直接匹配文件；没找到就匹配静态资源；还没找到就交给 Node 处理；否则就返回 4xx/5xx 的状态码。

## Upstream Cache

- [A Guide to Caching with NGINX and NGINX Plus](https://www.nginx.com/blog/nginx-caching-guide/)

```
http {
    ......
    proxy_cache_path /var/cache/nginx/cache levels=1:2 keys_zone=imgcache:100m inactive=1d max_size=10g;
    server {
    ........
        location ~* ^.+\.(js|ico|gif|jpg|jpeg|png|html|htm)$ {
        log_not_found off;
        access_log off;
        expires 7d;
        proxy_pass http://img.example.com ;
        proxy_cache imgcache;
        proxy_cache_valid 200 302 1d;
        proxy_cache_valid 404 10m;
        proxy_cache_valid any 1h;
        proxy_cache_use_stale error timeout invalid_header updating http_500 http_502 http_503 http_504;
        }
    }
}
```

# HTTPS

- [HTTPS 理论详解与实践 ](https://segmentfault.com/a/1190000004985253)

## Let's Encrypt 证书申请

Let's Encrypt 为我们提供了非常方便的命令行工具[certbot](https://certbot.eff.org/#ubuntuxenial-nginx)，笔者是在 Ubuntu 16.04 的机器上进行配置，因此只要执行如下命令即可:

```
# 安装letsencrypt命令行

$ sudo apt-get install letsencrypt
# 独立的为example.com与www.example.com申请证书
$ letsencrypt certonly --standalone -d example.com -d www.example.com
# 自动执行证书刷新操作
$ letsencrypt renew --dry-run --agree-tos
```

## 基本 HTTPS 配置

基本的 HTTPS 支持配置如下:

```
server {
    listen 192.168.1.11:443 ssl;  #ssl端口
    server_name  test.com;
    #为一个server{......}开启ssl支持
    #指定PEM格式的证书文件
    ssl_certificate      /etc/nginx/test.pem;
    #指定PEM格式的私钥文件
    ssl_certificate_key  /etc/nginx/test.key;
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
         root   html;
         index  index.html index.htm;
     }
}
```

在真实的生产环境中，我们的配置如下:

```
server {
    # 如果需要spdy也可以加上,lnmp1.2及其后版本都默认支持spdy,lnmp1.3 nginx 1.9.5以上版本默认支持http2
    listen 443 ssl;
    # 这里是你的域名
    server_name www.vpser.net;
    index index.html index.htm index.php default.html default.htm default.php;
    # 网站目录
    root /home/wwwroot/www.vpser.net;
    # 前面生成的证书，改一下里面的域名就行
    ssl_certificate /etc/letsencrypt/live/www.vpser.net/fullchain.pem;
    # 前面生成的密钥，改一下里面的域名就行
    ssl_certificate_key /etc/letsencrypt/live/www.vpser.net/privkey.pem;
    ssl_ciphers "EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5";
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;

    #这个是伪静态根据自己的需求改成其他或删除
    include wordpress.conf;

    #error_page 404 /404.html;

    location ~ [^/]\.php(/|$)
    {
        # comment try_files $uri =404; to enable pathinfo
        try_files $uri =404;
        fastcgi_pass unix:/tmp/php-cgi.sock;
        fastcgi_index index.php;
        # lnmp 1.0及之前版本替换为include fcgi.conf;
        include fastcgi.conf;
        #include pathinfo.conf;
    }

    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
    expires 30d;
    }

    location ~ .*\.(js|css)?$
    {
    expires 12h;
    }

    access_log off;
}
```

## 强制 HTTP 转到 HTTPS

### Nginx Rewrite

```
server {
    listen  192.168.1.111:80;
    server_name test.com;
    rewrite ^(.*)$  https://$host$1 permanent;
}
```

### Nginx 497 错误码

利用 error_page 命令将 497 状态码的链接重定向到https://test.com这个域名上

```
server {
    listen       192.168.1.11:443;  #ssl端口
    listen       192.168.1.11:80;   #用户习惯用http访问，加上80，后面通过497状态码让它自动跳到443端口
    server_name  test.com;
    #为一个server{......}开启ssl支持
    ssl                  on;
    #指定PEM格式的证书文件
    ssl_certificate      /etc/nginx/test.pem;
    #指定PEM格式的私钥文件
    ssl_certificate_key  /etc/nginx/test.key;

    #让http请求重定向到https请求
    error_page 497  https://$host$uri?$args;
}
```

### Meta 刷新，前端跳转

在 HTTP 正常返回的页面中添加 meta 属性：

```
<html>
<meta http-equiv="refresh" content="0;url=https://test.com/">
</html>
```

```
server {
    listen 192.168.1.11:80;
    server_name test.com;

    location / {
                #index.html放在虚拟主机监听的根目录下
        root /srv/www/http.test.com/;
    }
        #将404的页面重定向到https的首页
    error_page  404 https://test.com/;
}
```

## 反向 HTTPS 转发到内部 HTTP
