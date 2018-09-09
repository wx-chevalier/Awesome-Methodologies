[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Linux DevOps & Security CheatSheet

本文囊括了日常的持续集成、应用部署、服务器维护等方面的命令与技巧，相较于 [基础命令/DevOps/安全加固/Shell 编程](./Linux-Commands-CheatSheet.md) 等相关清单。本文围绕 DevOps 应用进行目录划分。

# 状态与性能检索

![default](https://user-images.githubusercontent.com/5803001/39466197-45bac832-4d5a-11e8-9c90-1cbdc0762b49.png)

1 处表示系统负载，它表示当前正在等待被 cpu 调度的进程数量，这个值小于系统 vcpu 数(超线程数)的时候是比较正常的，一旦大于 vcpu 数，则说明并发运行的进程太多了，有进程迟迟得不到 cpu 时间。这种情况给用户的直观感受就是敲任何命令都卡。

2 处表示当前系统的总进程数，通常该值过大的时候就会导致 load average 过大。

3 处表示 cpu 的空闲时间，可以反应 cpu 的繁忙程度，该值较高时表示系统 cpu 处于比较清闲的状态，如果该值较低，则说明系统的 cpu 比较繁忙。需要注意的是，有些时候该值比较高，表示 cpu 比较清闲，但是 load average 依然比较高，这种情况很可能就是因为进程数太多，进程切换占用了大量的 cpu 时间，从而挤占了业务运行需要使用的 cpu 时间。

4 处表示进程 IO 等待的时间，该值较高时表示系统的瓶颈可能出现在磁盘和网络。

5 处表示系统的剩余内存，反应了系统的内存使用情况。

6 处表示单个进程的 cpu 和内存使用情况。关于 top 命令中各个指标含义的进一步描述可以参见：

```sh
$ iostat -x -d 2

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.00     0.25    0.04    0.53     0.56     4.88    19.25     0.00    6.85    3.09    7.14   0.25   0.01
```

# 应用程序调用

## perf 与火焰图

perf 是 linux 下一个非常强大的性能分析工具，通过它可以分析出进程运行过程中的主要时间都花在了哪些地方；结合著名的 [FlameGraph](https://github.com/brendangregg/FlameGraph.git) 火焰图工具，我们能够快速定位 时间占用较多的函数调用。

```sh
# 执行采样
$ perf record -e cpu-clock -g -p ${PID}

# 用 perf script 工具对 perf.data 进行解析
perf script -i perf.data &> perf.unfold

# 将 perf.unfold 中的符号进行折叠
./stackcollapse-perf.pl perf.unfold &> perf.folded

# 生成 svg 火焰图
/flamegraph.pl perf.folded > perf.svg
```

# Security

```sh
#!/bin/sh

# desc: setup linux system security
# author:coralzd
# powered by www.xiaohuai.com
# version 0.1.2 written by 2011.05.03
#account setup

passwd -l xfs
passwd -l news
passwd -l nscd
passwd -l dbus
passwd -l vcsa
passwd -l games
passwd -l nobody
passwd -l avahi
passwd -l haldaemon
passwd -l gopher
passwd -l ftp
passwd -l mailnull
passwd -l pcap
passwd -l mail
passwd -l shutdown
passwd -l halt
passwd -l uucp
passwd -l operator
passwd -l sync
passwd -l adm
passwd -l lp

# chattr /etc/passwd /etc/shadow
chattr +i /etc/passwd
chattr +i /etc/shadow
chattr +i /etc/group
chattr +i /etc/gshadow
# add continue input failure 3 ,passwd unlock time 5 minite
sed -i 's#auth        required      pam_env.so#auth        required      pam_env.so\nauth       required       pam_tally.so  onerr=fail deny=3 unlock_time=300\nauth           required     /lib/security/$ISA/pam_tally.so onerr=fail deny=3 unlock_time=300#' /etc/pam.d/system-auth
# system timeout 5 minite auto logout
echo "TMOUT=300" >>/etc/profile

# will system save history command list to 10
sed -i "s/HISTSIZE=1000/HISTSIZE=10/" /etc/profile

# enable /etc/profile go!
source /etc/profile

# add syncookie enable /etc/sysctl.conf
echo "net.ipv4.tcp_syncookies=1" >> /etc/sysctl.conf

sysctl -p # exec sysctl.conf enable
# optimizer sshd_config

sed -i "s/#MaxAuthTries 6/MaxAuthTries 6/" /etc/ssh/sshd_config
sed -i  "s/#UseDNS yes/UseDNS no/" /etc/ssh/sshd_config

# limit chmod important commands
chmod 700 /bin/ping
chmod 700 /usr/bin/finger
chmod 700 /usr/bin/who
chmod 700 /usr/bin/w
chmod 700 /usr/bin/locate
chmod 700 /usr/bin/whereis
chmod 700 /sbin/ifconfig
chmod 700 /usr/bin/pico
chmod 700 /bin/vi
chmod 700 /usr/bin/which
chmod 700 /usr/bin/gcc
chmod 700 /usr/bin/make
chmod 700 /bin/rpm

# history security

chattr +a /root/.bash_history
chattr +i /root/.bash_history

# write important command md5
cat > list << "EOF" &&
/bin/ping
/bin/finger
/usr/bin/who
/usr/bin/w
/usr/bin/locate
/usr/bin/whereis
/sbin/ifconfig
/bin/pico
/bin/vi
/usr/bin/vim
/usr/bin/which
/usr/bin/gcc
/usr/bin/make
/bin/rpm
EOF

for i in `cat list`
do
   if [ ! -x $i ];then
   echo "$i not found,no md5sum!"
  else
   md5sum $i >> /var/log/`hostname`.log
  fi
done
rm -f list
```

# Todos

- [ ] 参考[用十条命令在一分钟内检查 Linux 服务器性能](http://www.infoq.com/cn/news/2015/12/linux-performance)
- [ ] [How Do I Find The Largest Top 10 Files and Directories On a Linux / UNIX / BSD?](http://www.cyberciti.biz/faq/how-do-i-find-the-largest-filesdirectories-on-a-linuxunixbsd-filesystem/)
