[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Linux 常用命令与技巧清单

更多关于 Linux DevOps, 安全加固与优化，亦或是更简洁版的命令清单可以查看 [Linux Quick Commands CheatSheet](https://github.com/wxyyxc1992/Awesome-CheatSheet/blob/master/toc.md#linux)

## 文件目录

* 当前目录：.
* 上一级目录的上一级目录：..
* 用户的主目录：~
* 文件的系统根目录：/

### 通配符

这类似于 SQL 中的 % 符号，例如，使用「WHERE first_name LIKE 『 John%』」搜索所有以 John 起始的名字。

在 bash 中，相应的命令是「John*」。如果想列出一个文件夹中所有以「.json 」结尾的文件，可以输入：「 ls *.json」。

## 命令连接

### 管道

1 | 一种管道，其左方是一个命令的 STNOUT，将作为管道右方的另一个命令的 STDIN。 例如：echo ‘test text’ | wc -l

2 > 大于号，作用是取一个命令 STDOUT 位于左方，并将其写入 / 覆写(overwrite )入右方的一个新文件。 例如：ls > tmp.txt

3 >> 两个大于号，作用是取一个命令 STDOUT 位于左方，并将其追加到右方的一个新的或现有文件中。例如：date >> tmp.txt

## Tmux

Tmux 是一个工具，用于在一个终端窗口中运行多个终端会话；还可以通过 Tmux 使终端会话运行于后台或是按需接入、断开会话。本部分是对于 [tmux shortcuts & cheatsheet](https://parg.co/UrT) 一文的总结提取 :

```sh
# 启动新会话
tmux

# 指定新 Session 的名称，并创建
tmux new -s myname

# 列举出所有的 Session
tmux ls

# 附着到某个 Session
tmux a  #  (or at, or attach)

# 根据指定的名称附着到 Session
tmux a -t myname

# 关闭某个 Session
tmux kill-session -t myname

# 关闭全部 Session
tmux ls | grep : | cut -d. -f1 | awk '{print substr($1, 0, length($1)-1)}' | xargs kill
```

在 Tmux 中，使用  `ctrl + b` 前缀，然后可以使用如下命令 :

```sh
# Sessions
:new<CR>  new session
s  list sessions
$  name session

# Windows (tabs)
c  create window
w  list windows
n  next window
p  previous window
f  find window
,  name window
&  kill window

# Panes (splits)
%  vertical split
"  horizontal split

o  swap panes
q  show pane numbers
x  kill pane
+  break pane into window (e.g. to select text by mouse to copy)
-  restore pane from window
⍽  space - toggle between layouts
<prefix> q (Show pane numbers, when the numbers show up type the key to goto that pane)
<prefix> { (Move the current pane left)
<prefix> } (Move the current pane right)
<prefix> z toggle pane zoom

# Resizing Panes
PREFIX : resize-pane -D (Resizes the current pane down)
PREFIX : resize-pane -U (Resizes the current pane upward)
PREFIX : resize-pane -L (Resizes the current pane left)
PREFIX : resize-pane -R (Resizes the current pane right)
PREFIX : resize-pane -D 20 (Resizes the current pane down by 20 cells)
PREFIX : resize-pane -U 20 (Resizes the current pane upward by 20 cells)
PREFIX : resize-pane -L 20 (Resizes the current pane left by 20 cells)
PREFIX : resize-pane -R 20 (Resizes the current pane right by 20 cells)
PREFIX : resize-pane -t 2 20 (Resizes the pane with the id of 2 down by 20 cells)
PREFIX : resize-pane -t -L 20 (Resizes the pane with the id of 2 left by 20 cells)
```

## Cron

![](http://fs.gimoo.net/img/2014/10/12/011835_5439666b84167.jpg)

```sh
\*　　 \*　　 \*　　 \*　　 \*　　 command
分　   时　   日　   月　   周　   命令
```

第 1 列表示分钟 1 ～ 59 每分钟用*或者 */1 表示第 2 列表示小时 1 ～ 23 ( 0 表示 0 点)第 3 列表示日期 1 ～ 31 第 4 列表示月份 1 ～ 12 第 5 列标识号星期 0 ～ 6 ( 0 表示星期天)第 6 列要运行的命令

在以上各个字段中，还可以使用以下特殊字符：

星号(\* )：代表所有可能的值，例如 month 字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。

逗号(, )：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”

中杠(- )：可以用整数之间的中杠表示一个整数范围，例如 “2-6” 表示 “2,3,4,5,6”

正斜线(/ )：可以用正斜线指定时间的间隔频率，例如 “0-23/2” 表示每两小时执行一次。同时正斜线可以和星号一起使用，例如 \*/10，如果用在 minute 字段，表示每十分钟执行一次。

cron 是一个可以用来根据时间、日期、月份、星期的组合来调度对重复任务的执行的守护进程。

cron 假定系统持续运行。如果当某任务被调度时系统不在运行，该任务就不会被执行。√√

# 文件系统

## 打包

### tar

```sh
# 指定文件名创建压缩包
$ tar -cv -f archive.tar file1.txt file2.txt

# 过滤某个文件夹下面的某些子文件夹
$ tar cfvz --exclude='<dir1>' --exclude='<dir2>' target.tgz target_dir
$ tar cvf dir.tar.gz --exclude='/dir/subdir/subsubdir/*' dir

# 向压缩包中添加文件
$ tar -rf archive.tar file3.txt

######
```

```sh
# 展示 curl 的进度
$ curl --progress-bar -T "${SOME_LARGE_FILE}" "${UPLOAD_URL}" | tee /dev/null
```

[fkill](https://github.com/sindresorhus/fkill)

![](http://7xkt0f.com1.z0.glb.clouddn.com/65DCC0D6-CDE4-4199-9669-2CA32259FB15.png)

初接触 Linux 时即需要通过 Shell 进行交互控制，而所谓的 Shell 即是用户和 Linux(内核)之间的接口程序，其可以被看做命名语言解释器(Command-Language Interpreter )。Shell 也可以被系统中其他有效的 Linux 应用程序所调用。Shell 首先判断是否为内部命令，然后在搜索路径($PATH )里寻找这些应用程序；搜索路径是一个能找到可执行程序的目录列表，如果你键入的命令不是一个内部命令并且在路径里没有找到这个可执行文件，将会显示一条错误信息。而如果命令被成功的找到的话，Shell 的内部命令或应用程序将被分解为系统调用并传给 Linux 内核。

而 Bash(Bourne Again Shell) 则是 Bourne Shell(Sh) 的扩展，其优化了原本用户输入处理的不足，提供了多种便捷用户输入的方式。在 Windows 10 之后其内置了 Linux 子系统，不过在老版本的 Windows 中我们还可以使用 [Git Bash]()、[Babun]()、[Cash (JavaScript)]() 这些工具来模拟执行 Shell 命令。

本文参考资料包括但不限于：[the-art-of-command-line](https://parg.co/bXZ)、[Linux Commands Cheat Sheet](https://parg.co/Uqu)，更多参考资料请前往 [Linux Reference, Linux Shell Reference, Docker Reference, Git Reference]() 查看链接索引。相较于这些参考资料本文希望能够更为生动详细，并且不仅仅局限于 Bash 本身，而是包含具有一定价值的其他扩展命令，从而更贴近于日常工作中的需要；鉴于篇幅限制，我们在日常操作中还会经常使用 [Git CheatSheet, Docker CheatSheet](https://parg.co/Uqh) 等。

# 命令执行

## 命令辅助

### 帮助文档

当我们对于某个命令不甚熟悉的时候，可以使用 `man bash` 查看说明文档，使用 `Tab` 查看推荐参数或者指令。使用 `man ascii` 查看具有十六进制和十进制值的 ASCII 表；`man unicode`，`man utf-8`，以及 `man latin1` 有助于你去了解通用的编码信息。

[tldr](https://github.com/tldr-pages/tldr/) 能够为我们提供直观的命令解释与示范用法。

```sh
lsof



Lists open files and the corresponding processes.

Note: Root privileges (or sudo) is required to list files opened by others.



- Find the processes that have a given file open:

lsof path/to/file



- Find the process that opened a local internet port:

lsof -i :port



...
```

### 输入辅助

使用 `ctrl-w` 删除最后一个单词，使用 `ctrl-u` 删除自光标处到行首的内容，使用 `ctrl-k` 删除自光标处到行末的内容；使用 `ctrl-b` 与 `ctrl-f` 按字母进行前后移动；使用 `ctrl-a` 将光标移动到行首，使用 `ctrl-e` 将光标移动到行末。

为了方便编辑长命令，我们可以设置自己的默认编辑器(系统默认是 Emacs)，`export EDITOR=vim`，使用 `ctrl-x ctrl-e` 会打开一个编辑器来编辑当前输入的命令。

对于较长的命令，可以使用 `alias` 创建常用命令的快捷方式，譬如 `alias ll = 'ls -latr'` 创建新的名为 `ll` 的快捷方式。也可以使用 `{...}` 来进行命令简写：

```sh
# 同时移动两个文件

$ mv foo.{txt,pdf} some-dir



# 会被扩展成 cp somefile somefile.bak

$ cp somefile{,.bak}



# 会被扩展成所有可能的组合，并创建一个目录树

$ mkdir -p test-{a,b,c}/subtest-{1,2,3}
```

### 工作路径

`cd` 命令可以切换工作路径，输入 `cd ~` 可以进入 home 目录。要访问你的 home 目录中的文件，可以使用前缀 `~`(例如 `~/.bashrc`)。在 `sh` 脚本里则用环境变量 `$HOME` 指代 home 目录的路径。

而在 Bash 脚本中，同样可以使用 `cd` 命令切换工作目录，但是对于那些需要临时切换目录的情景，我们可以使用小括号进行控制：

```sh
# do something in current dir

(cd /some/other/dir && other-command)

# continue in original dir
```

## 命令连接

如果我们希望仅在前一个命令执行成功之后执行后一个命令，则需要使用 && 命令连接符：

```sh
cd /my_folder && rm *.jar && svn co path to repo && mvn compile package install



# 也可以写为多行模式



cd /my_folder \

&& rm *.jar \

&& svn co path to repo \

&& mvn compile package install
```

如果我们希望能够无论前一个命令是否成功皆开始执行下一个命令，则可以使用 ; 分隔符：

```sh
cd /my_folder; rm *.jar; svn co path to repo; mvn compile package install
```

### 参数切割

使用 `xargs` 将长列表参数切分为可用的短列表，常用的命令譬如：

```sh
# 搜索名字中包含 Stock 的文件

$ find . -name "*.java" | xargs grep "Stock"



# 清除所有后缀名为 tmp 的临时文件

$ find /tmp -name "*.tmp" | xargs rm
```

### 后台运行

当用户注销 (logout) 或者网络断开时，终端会收到 HUP (hangup) 信号从而关闭其所有子进程；我们可以通过让进程忽略 HUP 信号，或者让进程运行在新的会话里从而成为不属于此终端的子进程来进行后台执行。

```sh
# nohup 方式

$ nohup ping www.ibm.com &



# screen 方式，创建并且连接到新的屏幕



# 创建新的伪终端

$ screen -dmS Urumchi



# 连接到当前伪终端

$ screen -r Urumchi
```

## 环境配置

### 默认执行

默认的 Bourne Shell 会从 `~/.profile` 文件中读取并且执行命令。而 Bash 会从

~/.profile is the place to put stuff that applies to your whole session, such as programs that you want to start when you log in (but not graphical programs, they go into a different file), and environment variable definitions.

~/.bashrc is the place to put stuff that applies only to bash itself, such as alias and function definitions, shell options, and prompt settings. (You could also put key bindings there, but for bash they normally go into ~/.inputrc.)

~/.bash_profile can be used instead of ~/.profile, but it is read by bash only, not by any other shell. (This is mostly a concern if you want your initialization files to work on multiple machines and your login shell isn't bash on all of them.) This is a logical place to include ~/.bashrc if the shell is interactive. I recommend the following contents in ~/.bash_profile:

```sh
if [ -r ~/.profile ]; then . ~/.profile; fi

case "$-" in *i*) if [ -r ~/.bashrc ]; then . ~/.bashrc; fi;; esac
```

![](https://zwischenzugs.files.wordpress.com/2018/01/shell-startup-actual.png?w=840)

### 历史记录

最常用的历史记录检索方式就是使用 `history`:

```sh
# 顺序查看

$ history | more

1  2008-08-05 19:02:39 service network restart

2  2008-08-05 19:02:39 exit

3  2008-08-05 19:02:39 id

4  2008-08-05 19:02:39 cat /etc/redhat-release



# 查看最新的命令

$ history | tail -3
```

反馈的命令记录中存在编号，我们可以根据编号来重复执行历史记录中的命令：

```sh
$ !4

cat /etc/redhat-release

Fedora release 9 (Sulphur)
```

我们也可以使用 Control+R 来进行交互式检索：

```sh
$ [Press Ctrl+R from the command prompt,

which will display the reverse-i-search prompt]

(reverse-i-search)`red': cat /etc/redhat-release

[Note: Press enter when you see your command,

which will execute the command from the history]
```

使用 `history -c` 能够清除所有的历史记录，或者设置 HISTSIZE 环境变量以避免记录：

```sh
$ export HISTSIZE=0

$ history

# [Note that history did not display anything]
```

`history` 命令往往只会记录用户交互式的命令内容，更详细的操作记录可以使用 `more /var/log/messages` 查看记录文件。

## Tmux

# 用户权限

## SSH

### 密钥管理

```sh
# 生成名为 id_rsa 的私钥文件和名为 id_rsa.pub 的公钥文件

$ ssh-keygen -t rsa



# 指定 4096 位的长度

$ ssh-keygen -b 4096 -t rsa
```

生成的 id_rsa.pub 公钥文件也可以用于配置 Git 仓库的 SSH 访问等。

### 用户登录

我们可以使用 ssh 登录到本机(切换用户)或者远端 Linux 设备中，通过将本机生成的公钥文件写入目标机器的 authorized_keys 即可以实现免密码登录：

```sh
# 指定登录端口

ssh root@{host} -p{port}



# 上传公钥进行免密码登录

touch ~/.ssh/authorized_keys



cat -n ~/.ssh/rsa.pub ~/.ssh/authorized_keys



# 使用 ssh-copy-id 添加公钥

ssh-copy-id username@remote-server
```

## 用户管理

将 shell 切换为其他用户，使用 `su username` 或者 `sudo - username`。加入 `-` 会使得切换后的环境与使用该用户登录后的环境相同。省略用户名则默认为 root。切换到哪个用户，就需要输入*哪个用户的*密码。

# 文件系统

## 文件操作

### 创建

```shell
# 创建文件夹

mkdir <name>



# 递归创建父文件夹

mkdir -p / --parents backup/old



# 创建文件夹时同时指定权限

mkdir -m a=rwx backup



mkdir -p -m 777 backup/server/2011/11/30
```

### 移动

### 压缩

```sh
# 将文件解压到指定文件名

$ tar -xvzf fileName.tar.gz -C newFileName



# 将文件夹压缩到指定目录

$ tar -czf target.tar.fz file1 file2 file3
```

## 文件检索

* 可以使用 `ls -l` 查看目录下文件列表如统计 /home/han 目录 ( 包含子目录 ) 下的所有 js 文件则: ls -lR /home/han|grep js|wc -l 或 ls -l "/home/han"|grep "js"|wc -l

### 文件名搜索

可以使用 [fzf](https://github.com/junegunn/fzf) 进行交互式检索，在[这里](https://github.com/junegunn/fzf-bin/releases)下载二进制文件

```sh
$find * -type f | fzf > selected
```

![fzf](https://raw.githubusercontent.com/junegunn/i/master/fzf-preview.png)

### 文件内容搜索

[这篇](http://www.hostingadvice.com/blog/tutorial-use-ack-grep-linux/)文章对于 grep 与 ack 有较为详细的介绍

[ag](https://github.com/ggreer/the_silver_searcher)

## 文件属性

使用 `fuser tmpFile.js` 查看指定文件被进程占用情况，

## 磁盘管理

* 查看磁盘状态

```sh
# 查看磁盘剩余空间

$ df -sh
```

```s
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util



wrqm/s：每秒这个设备相关的写入请求有多少被Merge了。

rsec/s：每秒读取的扇区数； 判断磁盘在读还是写

wsec/：每秒写入的扇区数。

rKB/s：The number of read requests that were issued to the device per second；

wKB/s：The number of write requests that were issued to the device per second；

avgrq-sz 平均请求扇区的大小

avgqu-sz 是平均请求队列的长度。毫无疑问，队列长度越短越好。

await:  每一个IO请求的处理的平均时间(单位是微秒毫秒)。这里可以理解为IO的响应时间，一般地系统IO响应时间应该低于5ms，如果大于10ms就比较大了

%util: 在统计时间内所有处理IO时间，除以总共统计时间
```

# 文件读写

## 读取

`tailf` 命令类似于 `tail -f`，其可以打印出文件的最后十行内容，并且会随着文件的增长而自动滚动；不过其不会在文件没有变化的时候去频繁访问文件。

## Vim

### 搜索匹配与替换

Vim 中可以使用 `:s` 命令来替换字符串：

```sh
：s/vivian/sky/ 替换当前行第一个 vivian 为 sky



：s/vivian/sky/g 替换当前行所有 vivian 为 sky



# n 为数字，若 n 为 .，表示从当前行开始到最后一行

：n，$s/vivian/sky/ 替换第 n 行开始到最后一行中每一行的第一个 vivian 为 sky



：n，$s/vivian/sky/g 替换第 n 行开始到最后一行中每一行所有 vivian 为 sky



：%s/vivian/sky/(等同于 ：g/vivian/s//sky/) 替换每一行的第一个 vivian 为 sky



：%s/vivian/sky/g(等同于 ：g/vivian/s//sky/g) 替换每一行中所有 vivian 为 sky
```

# 系统进程

## 系统检视

### 版本型号

* 使用 `hostname` 查看当前主机名，使用 `sudo hostname newName` 修改当前主机名

- 查看 Linux 系统版本

```bash
# 查看内核版本

$ cat /proc/version



Linux version 2.6.18-238.el5 (mockbuild@x86-012.build.bos.redhat.com) (gcc version 4.1.2 20080704 (Red Hat 4.1.2-50)) #1 SMP Sun Dec 19 14:22:44 EST 2010



$ uname -r



2.6.18-238.el5



$ uname -a



Linux SOR_SYS.99bill.com 2.6.18-238.el5 #1 SMP Sun Dec 19 14:22:44 EST 2010 x86_64 x86_64 x86_64 GNU/Linux



# 查看 Linux 发行版本

$ lsb_release -a



$ cat /etc/issue

Red Hat Enterprise Linux Server release 5.6 (Tikanga)

Kernel \r on an \m



$ file /bin/bash



/bin/bash: ELF 64-bit LSB executable, AMD x86-64, version 1 (SYSV), for GNU/Linux 2.6.9, dynamically linked (uses shared libs), for GNU/Linux 2.6.9, stripped
```

### 运行状态

我们可以使用 `uptime` 或者 `w` 来查看当前用户的接入时间：

```sh
$ w

# 15:33:49 up 58 days,  5:45,  1 user,  load average: 0.12, 0.15, 0.22

# USER     TTY        LOGIN@   IDLE   JCPU   PCPU WHAT

# root     pts/1     15:15   49.00s  0.04s  0.00s w
```

## 内存

## 进程

### 进程监控

* 使用 `pstree -p` 查看当前进程树，使用 `ps -A` 查看所有进程信息，使用 `ps -aux` 查看所有正在内存中的程序，使用 `ps -ef` 查看所有连同命令行的进程信息；使用 `ps -u root` 显示指定用户信息；使用 `ps -ef | grep ssh` 查看特定进程。

使用 `top` 查看进程资源占用情况，也可以使用扩展 `htop` 或者 `gtop`，如果针对容器监控，可以使用 [ctop](https://github.com/bcicen/ctop)。

```sh
# 指定查看用户

$ top -u oracle




# 栏目内容解释

# PID：进程的ID

# USER：进程所有者

# PR：进程的优先级别，越小越优先被执行

# NInice：值

# VIRT：进程占用的虚拟内存

# RES：进程占用的物理内存

# SHR：进程使用的共享内存

# S：进程的状态。S表示休眠，R表示正在运行，Z表示僵死状态，N表示该进程优先值为负数

# %CPU：进程占用CPU的使用率

# %MEM：进程使用的物理内存和总内存的百分比

# TIME+：该进程启动后占用的总的CPU时间，即占用CPU使用时间的累加值。

# COMMAND：进程启动命令名称
```

```sh
$ npm install gtop -g
```

![](https://raw.githubusercontent.com/aksakalli/gtop/master/img/demo.gif)

### 进程关闭

```sh
$ pkill -f java
```

# 网络

## 状态

```sh
# 查看指定端口的占用情况

$ lsof -i:80



# 查看某个进程的 TCP 连接

$ lsof -p <pid> | grep TCP
```

## 配置

## 请求
