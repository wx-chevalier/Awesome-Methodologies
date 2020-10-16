# Linux 安全加固备忘清单

# 用户权限

```sh
# 查询最近登录到系统的用户和系统重启的时间和日期
$ last reboot | less
```

[基础加固脚本](https://parg.co/K2m)

```sh

```

# 网络

开启端口
firewall-cmd --zone=public --add-port=80/tcp --permanent
命令含义：
--zone #作用域
--add-port=80/tcp #添加端口，格式为：端口/通讯协议
--permanent #永久生效，没有此参数重启后失效
重启防火墙
firewall-cmd --reload

# TBD

- https://www.freebuf.com/articles/system/218407.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io 最全 Linux 应急响应技巧
