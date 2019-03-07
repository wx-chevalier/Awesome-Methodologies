[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheets)

> 📌 相较于这些参考资料本文希望能够更为生动详细，并且不仅仅局限于 Bash 本身，而是包含具有一定价值的其他扩展命令，从而更贴近于日常工作中的需要。

# Linux Commands CheatSheet | 常用命令与技巧清单

Shell 即是用户和 Linux 内核之间的接口程序，其可以被看做命名语言解释器（Command-Language Interpreter）。Bash(Bourne Again Shell) 则是 Bourne Shell(Sh) 的扩展，其优化了原本用户输入处理的不足，提供了多种便捷用户输入的方式。

# 命令执行

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

### 目录与工作路径

- 当前目录：.
- 上一级目录的上一级目录：..
- 用户的主目录：~
- 文件的系统根目录：/

`cd` 命令可以切换工作路径，输入 `cd ~` 可以进入 home 目录。要访问你的 home 目录中的文件，可以使用前缀 `~`(例如 `~/.bashrc`)。在 `sh` 脚本里则用环境变量 `$HOME` 指代 home 目录的路径。

而在 Bash 脚本中，同样可以使用 `cd` 命令切换工作目录，但是对于那些需要临时切换目录的情景，我们可以使用小括号进行控制：

```sh
# do something in current dir

(cd /some/other/dir && other-command)

# continue in original dir
```

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
# Copy 'filename' as 'filename.bak.
$ cp filename{,.bak}

# 会被扩展成 cp somefile somefile.bak
$ cp somefile{,.bak}

# 会被扩展成所有可能的组合，并创建一个目录树
$ mkdir -p test-{a,b,c}/subtest-{1,2,3}
```

### 历史记录

最常用的历史记录检索方式就是使用 `history`:

```sh
# 顺序查看
$ history | more
1  2008-08-05 19:02:39 service network restart
...

# 查看最新的命令
$ history | tail -3
```

反馈的命令记录中存在编号，我们可以根据编号来重复执行历史记录中的命令：

```sh
$ !4
cat /etc/redhat-release
Fedora release 9 (Sulphur)
```

使用 `history -c` 能够清除所有的历史记录，或者设置 HISTSIZE 环境变量以避免记录：

```sh
$ export HISTSIZE=0
$ history

# [Note that history did not display anything]
```

`history` 命令往往只会记录用户交互式的命令内容，更详细的操作记录可以使用 `more /var/log/messages` 查看记录文件。

我们也可以使用 Control+R 来进行交互式检索：

```sh
$ [Press Ctrl+R from the command prompt,

which will display the reverse-i-search prompt]

(reverse-i-search)`red': cat /etc/redhat-release

[Note: Press enter when you see your command,

which will execute the command from the history]
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

如果我们希望能够无论前一个命令是否成功皆开始执行下一个命令，则可以使用 `;` 分隔符：

```sh
cd /my_folder; rm *.jar; svn co path to repo; mvn compile package install
```

### 管道

1 | 一种管道，其左方是一个命令的 STNOUT，将作为管道右方的另一个命令的 STDIN。 例如：echo ‘test text’ | wc -l

2 > 大于号，作用是取一个命令 STDOUT 位于左方，并将其写入 / 覆写(overwrite)入右方的一个新文件。 例如：ls > tmp.txt

3 >> 两个大于号，作用是取一个命令 STDOUT 位于左方，并将其追加到右方的一个新的或现有文件中。例如：date >> tmp.txt

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

## Tmux

Tmux 是一个工具，用于在一个终端窗口中运行多个终端会话；还可以通过 Tmux 使终端会话运行于后台或是按需接入、断开会话。本部分是对于 [tmux shortcuts & cheatsheet](https://parg.co/UrT) 一文的总结提取

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

在 Tmux 中，使用 `ctrl + b` 前缀，然后可以使用如下命令 :

```sh
# Sessions
:new<CR>  new session
s  list sessions
$  name session
[  view history
d  detach

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

# 用户权限

## SSH

### 密钥管理

```sh
# 生成名为 id_rsa 的私钥文件和名为 id_rsa.pub 的公钥文件
$ ssh-keygen -t rsa

# 指定 4096 位的长度
$ ssh-keygen -b 4096 -t rsa

# 使用 ssh-copy-id 添加公钥
ssh-copy-id username@remote-server
```

生成的 id_rsa.pub 公钥文件也可以用于配置 Git 仓库的 SSH 访问等。我们可以使用 ssh 登录到本机(切换用户)或者远端 Linux 设备中，通过将本机生成的公钥文件写入目标机器的 authorized_keys 即可以实现免密码登录。

## 用户管理

将 shell 切换为其他用户，使用 `su username` 或者 `sudo - username`。加入 `-` 会使得切换后的环境与使用该用户登录后的环境相同。省略用户名则默认为 root。切换到哪个用户，就需要输入*哪个用户的*密码。

# 文件系统

## 状态检索

### 文件检索

- 可以使用 `ls -l` 查看目录下文件列表如统计 /home/han 目录 ( 包含子目录 ) 下的所有 js 文件则:

```sh
$ ls -lR /home/han | grep js | wc -l

$ ls -l "/home/han" | grep "js" |wc -l

$ ls -l --sort=size --block-size=M
```

这类似于 SQL 中的 % 符号，例如，使用 `WHERE first_name LIKE 『John%` 搜索所有以 John 起始的名字。在 Bash 中，相应的命令是`John*`。如果想列出一个文件夹中所有以 `.json` 结尾的文件，可以输入 `ls *.json`。

可以使用 [fzf](https://github.com/junegunn/fzf) 进行交互式检索，在[这里](https://github.com/junegunn/fzf-bin/releases)下载二进制文件

```sh
# 根据文件类型搜索
$ find * -type f | fzf > selected

# 根据文件名匹配
$ find . -name '*.map' -exec rm {} \;
```

![fzf](https://raw.githubusercontent.com/junegunn/i/master/fzf-preview.png)

### 磁盘空间

使用 du(disk usage)/df(disk free) 查看磁盘状态，du 是通过搜索文件来计算每个文件的大小然后累加，du 能看到的文件只是一些当前存在的，没有被删除的。他计算的大小就是当前他认为存在的所有文件大小的累加和；df 记录的是通过文件系统获取到的文件的大小，他比 du 强的地方就是能够看到已经删除的文件，而且计算大小的时候，把这一部分的空间也加上了，更精确了。

```sh
# 查看磁盘剩余空间
$ df -ah

$ df --block-size=GB/-k/-m

# 查看当前目录下的目录空间占用
$ du -h --max-depth=1 /var/ | sort

# 查看 tmp 目录的磁盘占用
$ du -sh /tmp

# 查看当前目录包含子目录的大小
$ du -sm .

# 查看目录下文件尺寸
$ ls -l --sort=size --block-size=M
```

### I/O

使用 `fuser tmpFile.js` 查看指定文件被进程占用情况，

可以使用 iostat 查看磁盘详细的参数与吞吐量记录:

```sh
# 查看磁盘详细参数
$ iostat

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util

wrqm/s：每秒这个设备相关的写入请求有多少被 Merge 了。

rsec/s：每秒读取的扇区数； 判断磁盘在读还是写

wsec/：每秒写入的扇区数。

rKB/s：The number of read requests that were issued to the device per second；

wKB/s：The number of write requests that were issued to the device per second；

avgrq-sz 平均请求扇区的大小

avgqu-sz 是平均请求队列的长度。毫无疑问，队列长度越短越好。

await:  每一个IO请求的处理的平均时间(单位是微秒毫秒)。这里可以理解为 IO 的响应时间，一般地系统IO响应时间应该低于 5ms，如果大于 10ms 就比较大了

%util: 在统计时间内所有处理IO时间，除以总共统计时间
```

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

### 解压缩与切割

```sh
# 将文件解压到指定文件名
$ tar -xvzf fileName.tar.gz -C newFileName

# 将文件夹压缩到指定目录
$ tar -czf target.tar.fz file1 file2 file3

# 指定文件名创建压缩包
$ tar -cv -f archive.tar file1.txt file2.txt

# 过滤某个文件夹下面的某些子文件夹
$ tar cfvz --exclude='<dir1>' --exclude='<dir2>' target.tgz target_dir
$ tar cvf dir.tar.gz --exclude='/dir/subdir/subsubdir/*' dir

# 向压缩包中添加文件
$ tar -rf archive.tar file3.txt
```

有时候我们还需要将大型文件进行切割处理：

```sh
$ split -b 10M home.tar.bz2 "home.tar.bz2.part"
$ ls -lh home.tar.bz2.parta*
# 混合 tar 命令使用
$ tar -cvzf - wget/* | split -b 150M - "downloads-part"

# 拼接文件
$ cat home.tar.bz2.parta* >backup.tar.gz.joined
```

## 文件属性

## 磁盘管理

### 分区、格式化与挂载

fdisk 常用于进行磁盘分区，主分区（包含扩展分区）、逻辑分区，主分区最多有 4 个（包含扩展分区）。因此我们在对硬盘分区时最好划分主分区连续，比如说：主分区一、主分区二、扩展分区。

```sh
# 查看系统上的硬盘
$ fdisk -l

# 操作指定磁盘
$ fdisk /dev/sda

# 列出当前操作硬盘的分区情况
$ p

# 删除某个分区
$ d

# 增加某个分区
$ n
```

`mkfs.ext4` 等常用于对分区进行格式化，mount 则是用于挂载分区：

```sh
# 格式化
$ mkfs.ext4 /dev/xvdb1

# 挂载
$ mount /dev/xvdb1 /data2

# 卸载
$ umount /data2
```

# 文本处理

## 非交互式检索与编辑

### tail & head & cat & more

`tailf` 命令类似于 `tail -f`，其可以打印出文件的最后十行内容，并且会随着文件的增长而自动滚动；不过其不会在文件没有变化的时候去频繁访问文件。

```sh
$ head -5 /etc/passwd

$ tail -10 /etc/passwd
$ tail -n 10 /etc/passwd
$ tail -f /var/log/messages

# 查看文件中间的第 5-10 行
$ sed -n '5,10p' /etc/passwd
```

### grep & ack

grep（global search regular expression(RE) and print out the line，全面搜索正则表达式并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

```sh
# 基本检索
$ grep match_pattern file_name
$ grep "match_pattern" file_1 file_2 file_3 ...

# 仅显示不匹配的文本
$ grep -v "match_pattern" file_1 file_2 file_3 ...

# 搜索多个文件并查找匹配文本在哪些文件中
$ grep -l "text" file1 file2 file3...

# 递归检索
$ grep "text" . -r -n
#只在目录中所有的.php和.html文件中递归搜索字符"main()"
$ grep "main()" . -r --include *.{php,html}
#在搜索结果中排除所有README文件
$ grep "main()" . -r --exclude "README"
#在搜索结果中排除filelist文件列表里的文件
$ grep "main()" . -r --exclude-from filelist

# 正则表达式检索
$ grep -E "[1-9]+"
$ egrep "[1-9]+"

# 检索当前目录下
$ grep 'John Resig' *

# 只输出文件中匹配到的部分 -o 选项
$ echo this is a test line. | grep -o -E "[a-z]+\."
# 输出模板在文本中匹配行数
$ grep -c "text" file_name
# 输出字符的总匹配行数
$ grep "text" file_name | wc -l
# 忽略字符串大小写
$ echo "hello world" | grep -i "HELLO"

# 显示匹配某个结果之后的3行，使用 -A 选项，-B 显示匹配某个结果之前的3行， -C 显示匹配某个结果的前三行和后三行
$ seq 10 | grep "5" -A 3

# 与其他命令协同使用
$ grep foo $(find . -name '*.pm' | grep -v .svn)
$ ls -l | grep .py
```

[ag](https://github.com/ggreer/the_silver_searcher) 是类似于 ack 但是性能更优地工具。

```sh
# ack 默认进行递归搜索
$ ack hello
$ ack -i hello
$ ack -v hello
$ ack -w hello
$ ack -Q 'hello*'

# 输出文件指定内容
# 输出所有文件第二行
$ ack --line=1

# 也可以指定目录/文件名/文件类型检索
# 查找所有python文件
$ ack --python hello
# 查找匹配正则的文件
$ ack -G hello.py$ hello

# ack 能够对搜索结果进行方便地定制
# 仅显示包含的文件名
$ ack -l 'hello'
# 仅显示非包含文件名
$ ack -L 'print'
# 以less形式展示
$ ack hello --pager='less -R'
# 不在头上显示文件
$ ack hello --noheading
# 不对匹配字符着色
$ ack hello --nocolor

# ack 同样能够用于查找文件
# 查找全匹配文件
$ ack -f hello.py
# 查找正则匹配文件
$ ack -g hello.py$
#查找然后排序
$ ack -g hello --sort-files
```

### awk

awk 是一种可以处理数据、产生格式化报表的语言。awk 的工作方式是读取数据文件，将每一行数据视为一条记录，每条记录以分隔符分成若干字段，然后输出。

awk 的日常用法：

    awk常用的格式：

    1. awk '样式' 文件，把符合样式的数据显示出来。

    2. awk '{操作}' 文件，对每一行都执行｛｝中的操作。

    3. awk '样式{操作}' 文件，对符合样式的数据进行括号里的操作。

awk 内置变量

```sh
ARGC               命令行参数个数
ARGV               命令行参数排列
ENVIRON            支持队列中系统环境变量的使用
FILENAME           awk浏览的文件名
FNR                浏览文件的记录数
FS                 设置输入域分隔符，等价于命令行 -F选项
NF                 浏览记录的域的个数
NR                 已读的记录数
OFS                输出域分隔符
ORS                输出记录分隔符
RS                 控制记录分隔符
```

```sh
$ echo 'BEGIN' | awk '{print $0 "\nline one\nline two\nline three"}'
BEGIN
line one
line two
line three

# 输出指定分割参数
$ route -n | awk '/UG[ \t]/{print $2}'

# 计算文件中的数值和
$ awk '{s+=$1} END {printf "%.0f", s}' mydatafile

# NF 表示的是浏览记录的域的个数，$NF 表示的最后一个Field（列），即输出最后一个字段的内容
$ free -m | grep buffers\/ | awk '{print $NF}'

ps aux | awk '{print $2}'  //获取所有进程PID

awk '/La/' 1.log   // 显示含La的数据行

awk '{print $1, $2}' 1.log   //显示每一行的第1和第2个字段

awk '/La/{print $1, $2}' 1.log   //将含有La关键词的数据行的第1以及第2个字段显示出来

awk -F : '/^root/{print $1, $2}'  /etc/passwd


awk 'BEGIN {count=0}{count++} END{print count}' /etc/passwd  //统计用户数

//BEGIN后紧跟的操作，在awk命令开始匹配第一行时执行，END后面紧跟的操作在处理完后执行

awk -F ':' 'BEGIN {count=0;} {name[count] = $1;count++;}; END{for (i = 0; i < NR; i++) print i, name[i]}' /etc/passwd  //显示所有账户


awk -F : 'NR > 1 && NR <=5 {print $1}' /etc/passwd  //显示一到五行
```

### sed

sed 是一种非交互式的流编辑器，可动态编辑文件。所谓的非交互式是说，sed 和传统的文本编辑器不同，并非和使用者直接互动，sed 处理的对象是文件的数据流。sed 的工作模式是，比对每一行数据，若符合样式，就执行指定的操作。

```sh
# 选项与参数
-n ：使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN 的数据一般都会被列出到终端上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来。
-e ：直接在命令列模式上进行 sed 的动作编辑；
-f ：直接将 sed 的动作写在一个文件内， -f filename 则可以运行 filename 内的 sed 动作；
-r ：sed 的动作支持的是延伸型正规表示法的语法。(默认是基础正规表示法语法)
-i ：直接修改读取的文件内容，而不是输出到终端。

# 函数
a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
d ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
p ：列印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！
```

常用示例：

```sh
# 在第一行和第二行间插入一行123Abc
sed -i '2i 123Abc' 1.log
# 在第二行和第三行间插入一行 123Abc
sed -i '2a 123Abc' 1.log

sed '1,4d' 1.log  //删除1到4行数据，剩下的显示出来。d是sed的删除命令。这里的删除并不是修改了源文件

# 删除最后一行
sed '$d' 1.log
# 删除匹配到包含'LA'字符行的数据，剩下的显示。//代表搜索
sed '/LA/d' 1.log

sed '/[0-9]\{3\}/d' 1.log   //删除包含三位数字的行，注意{3}个数指定的大括号转义

sed '/LA/!d' 1.log  // 反选 ，把不含LA行的数据删除

sed '/^$/d' 1.log //删除空白行

//如果想显示匹配到的呢？

sed '/a/p' 1.log   //由于默认sed也会显示不符合的数据行，所以要用-n，抑制这个操作

sed -n '/a/p' 1.log


//替换字符，把a替换成A
sed -n 's/a/A/p' 1.log  //s是替换的命令，第一个//中的字符是搜索目标（a），第二个//是要替换的字符A

//上面的只会替换匹配到的第一个，如果我想所有替换呢

sed -n 's/a/A/gp' 1.log   // g 全局替换

sed -n 's/a//gp' 1.log //删除所有的a

sed -n 's/^...//gp' 1.log  //删除每行的前三个字符

sed -n 's/...$//gp' 1.log  //删除每行结尾的三个字符

sed -n 's/\(A\)/\1BC/gp' 1.log  // 在A后面追加BC，\1表示搜索里面括号里的字符


sed -n '/AAA/s/234/567/p' 1.log  //找到包含字符AAA这一行，并把其中的234替换成567

sed -n '/AAA/,/BBB/s/234/567/p' 1.log //找到包含字符AAA或者BBB的行，并把其中的234替换成567

sed -n '1,4s/234/567/p' 1.log  //将1到4行中的234.替换成567

cat 1.log | sed -e '3,$d' -e 's/A/a/g'   //删除3行以后的数据，并把剩余的数据替换A为a

sed -i '1d' 1.log   //直接修改文件，删除第一行
```

## 交互式编辑

### Nano

Nano 是 Linux 上一个简单的文本编辑器，比起 Vim 来说它轻量很多，易于上手。

| 含义     | 快捷键             |
| -------- | ------------------ |
| 标记     | `Ctrl+6` / `Alt+A` |
| 复制整行 | `Alt+6`            |
| 剪贴整行 | `Ctrl+K`           |
| 粘贴     | `Ctrl+U`           |
| 查找     | `Ctrl+W` (WhereIs) |
| 继续查找 | `Alt+W`            |
| 上一页   | `Ctrl+Y`           |
| 下一页   | `Ctrl+V`           |
| 保存     | `Ctrl+O`           |
| 退出     | `Ctrl+X`           |

### Vim

- 搜索匹配与替换

Vim 中可以使用 `:s` 命令来替换字符串：

```sh
# 替换当前行第一个 vivian 为 sky
：s/vivian/sky/

# 替换当前行所有 vivian 为 sky
：s/vivian/sky/g

# n 为数字，若 n 为 .，表示从当前行开始到最后一行
# 替换第 n 行开始到最后一行中每一行的第一个 vivian 为 sky
：n，$s/vivian/sky/

# 替换第 n 行开始到最后一行中每一行所有 vivian 为 sky
：n，$s/vivian/sky/g

# 替换每一行的第一个 vivian 为 sky
：%s/vivian/sky/(等同于 ：g/vivian/s//sky/)

# 替换每一行中所有 vivian 为 sky
：%s/vivian/sky/g(等同于 ：g/vivian/s//sky/g)
```

# 系统进程

## Systemd

[init 进程](./Linux-CheatSheet.md)是 Linux 系统 Booting 之后的首个进程，其作为守护进程运行直至系统关闭；传统的 Linux 中的服务控制方式也主要依赖于 sysvinit 机制:

```sh
$ sudo /etc/init.d/apache2 start
# 或者
$ service apache2 start
```

当 sysvinit 系统初始化的时候，它是串行启动，并且会将所有可能用到的后台服务进程全部启动运行；系统必须等待所有的服务都启动就绪之后，才允许用户登录，导致启动时间过长与系统资源浪费。并且 init 进程只是执行启动脚本，不管其他事情，脚本需要自己处理各种情况，使得脚本复杂度增加很多。Systemd 就是为了解决这些问题而诞生的。它的设计目标是，为系统的启动和管理提供一套完整的解决方案；Systemd 并不是一个命令，而是一组命令，涉及到系统管理的方方面面。

```sh
# 查看 Systemd 的版本
$ systemctl --version

# 重启系统
$ sudo systemctl reboot

# 关闭系统，切断电源
$ sudo systemctl poweroff

# CPU 停止工作
$ sudo systemctl halt

# 查看启动耗时
$ systemd-analyze

# 查看每个服务的启动耗时
$ systemd-analyze blame

# 显示瀑布状的启动过程流
$ systemd-analyze critical-chain

# 显示指定服务的启动流
$ systemd-analyze critical-chain atd.service
```

Systemd 可以管理所有系统资源。不同的资源统称为 Unit(单位)，Unit 一共分成 12 种。

- Service unit：系统服务
- Target unit：多个 Unit 构成的一个组
- Device Unit：硬件设备
- Mount Unit：文件系统的挂载点
- Automount Unit：自动挂载点
- Path Unit：文件或路径
- Scope Unit：不是由 Systemd 启动的外部进程
- Slice Unit：进程组
- Snapshot Unit：Systemd 快照，可以切回某个快照
- Socket Unit：进程间通信的 socket
- Swap Unit：swap 文件
- Timer Unit：定时器

systemctl status 命令用于查看系统状态和单个 Unit 的状态。

```sh
# 显示系统状态
$ systemctl status

# 显示单个 Unit 的状态
$ sysystemctl status bluetooth.service

# 显示远程主机的某个 Unit 的状态
$ systemctl -H root@rhel7.example.com status httpd.service
```

我们最常用的就是 Unit 管理命令：

```sh
# 立即启动一个服务
$ sudo systemctl start apache.service

# 立即停止一个服务
$ sudo systemctl stop apache.service

# 重启一个服务
$ sudo systemctl restart apache.service

# 杀死一个服务的所有子进程
$ sudo systemctl kill apache.service

# 重新加载一个服务的配置文件
$ sudo systemctl reload apache.service

# 重载所有修改过的配置文件
$ sudo systemctl daemon-reload

# 显示某个 Unit 的所有底层参数
$ systemctl show httpd.service

# 显示某个 Unit 的指定属性的值
$ systemctl show -p CPUShares httpd.service

# 设置某个 Unit 的指定属性
$ sudo systemctl set-property httpd.service CPUShares=500
```

每一个 Unit 都有一个配置文件，告诉 Systemd 怎么启动这个 Unit。Systemd 默认从目录 `/etc/systemd/system/` 读取配置文件。但是，里面存放的大部分文件都是符号链接，指向目录 `/usr/lib/systemd/system/`，真正的配置文件存放在那个目录。systemctl enable 命令用于在上面两个目录之间，建立符号链接关系。配置文件的基础格式如下：

```sh
[Unit]
Description=ATD daemon

[Service]
Type=forking
ExecStart=/usr/bin/atd

[Install]
WantedBy=multi-user.target
```

## 系统检视

### 版本型号

- 使用 `hostname` 查看当前主机名，使用 `sudo hostname newName` 修改当前主机名

```sh
# 显示当前主机的信息
$ hostnamectl

# 设置主机名。
$ sudo hostnamectl set-hostname rhel7
```

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

查看系统当前的 CPU 与内存情况：

```sh
$ lscpu
```

### 进程监控

- 使用 `top` 查看进程资源占用情况，也可以使用扩展 htop 或者 gtop；如果针对容器监控，可以使用 [ctop](https://github.com/bcicen/ctop)。

- 使用 `pstree -p` 查看当前进程树，使用 `ps -A` 查看所有进程信息，使用 `ps -aux` 查看所有正在内存中的程序，使用 `ps -ef` 查看所有连同命令行的进程信息；使用 `ps -u root` 显示指定用户信息；使用 `ps -ef | grep ssh` 查看特定进程。

## 进程管理

### 进程关闭

```sh
$ pkill -f java
```

### 资源限制

ulimit 命令用来限制系统用户对 shell 资源的访问。ulimit 用于限制 shell 启动进程所占用的资源，支持以下各种类型的限制：所创建的内核文件的大小、进程数据块的大小、Shell 进程创建文件的大小、内存锁住的大小、常驻内存集的大小、打开文件描述符的数量、分配堆栈的最大大小、CPU 时间、单个用户的最大线程数、Shell 进程所能使用的最大虚拟内存。同时，它支持硬资源和软资源的限制。

我们常用的就是在 Web 服务器上修改每个进程可以打开的最大文件数：

```sh
$ ulimit -n 4096 # 将每个进程可以打开的文件数目加大到4096，缺省为1024
```

其他建议设置成无限制（unlimited）的一些重要设置是：

```sh
$ ulimit -d unlimited # 数据段长度
$ ulimit -m unlimited # 最大内存大小
$ ulimit -s unlimited # 堆栈大小
$ ulimit -t unlimited # CPU 时间
$ ulimit -v unlimited # 虚拟内存
```

我们也可以通过配置文件的方式永久写入：

```sh
vi /etc/security/limits.conf
# 添加如下的行
* soft noproc 11000
* hard noproc 11000
* soft nofile 4100
* hard nofile 4100
# 说明：* 代表针对所有用户，noproc 是代表最大进程数，nofile 是代表最大文件打开数
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

# 网络

## 状态

```sh
# 查看指定端口的占用情况
$ lsof -i:80
$ kill -9 $(lsof -t -i:8080)
# 查看某个进程的 TCP 连接
$ lsof -p <pid> | grep TCP

# 查看 TCP 连接数
$ netstat -an
# 统计80端口连接数
$ netstat -nat|grep -i "80"|wc -l
# 统计 IP 地址连接数
netstat -na|grep ESTABLISHED|awk {print $5}|awk -F: {print $1}| sort |uniq -c|sort -r +0n

netstat -na|grep SYN|awk {print $5}|awk -F: {print $1}|sort|uniq -c|sort -r +0n
```

## 配置

## 请求

### 文件下载

```sh
# 展示 curl 的进度
$ curl --progress-bar -T "${SOME_LARGE_FILE}" "${UPLOAD_URL}" | tee /dev/null
```

# Todos

- [ ] [the-art-of-command-line](https://parg.co/bXZ)
- [ ] [Linux Commands Cheat Sheet](https://parg.co/Uqu)
