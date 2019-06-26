[![返回目录](https://i.postimg.cc/JzFTMvjF/image.png)](https://github.com/wx-chevalier/Awesome-CheatSheets)

# Git CheatSheet | Git 命令速览与备忘清单

```sh
# 使用 credential 管理密码，拉取仓库时候仅需要输入一次
$ git global config credential.helper store
```

The name "git" was given by Linus Torvalds when he wrote the very first version. He described the tool as "the stupid content tracker" and the name as (depending on your way):

- random three-letter combination that is pronounceable, and not actually used by any common UNIX command. The fact that it is a mispronunciation of "get" may or may not be relevant.

- stupid. contemptible and despicable. simple. Take your pick from the dictionary of slang.

- "global information tracker": you're in a good mood, and it actually works for you. Angels sing, and a light suddenly fills the room.

- "g*dd*mn idiotic truckload of sh\*t": when it breaks

The major difference between Git and any other VCS (Subversion and friends included) is the way Git thinks about its data. Conceptually, most other systems store information as a list of file-based changes. These other systems (CVS, Subversion, Perforce, Bazaar, and so on) think of the information they store as a set of files and the changes made to each file over time (this is commonly described as delta-based version control).

![](https://cdn-images-1.medium.com/max/1600/1*6ywHRvYfgRVCSL_4xh1Mfw.png)

Git doesn’t think of or store its data this way. Instead, Git thinks of its data more like a series of snapshots of a miniature filesystem. With Git, every time you commit, or save the state of your project, Git basically takes a picture of what all your files look like at that moment and stores a reference to that snapshot. To be efficient, if files have not changed, Git doesn’t store the file again, just a link to the previous identical file it has already stored. Git thinks about its data more like a stream of snapshots.

![](https://cdn-images-1.medium.com/max/1600/0*V1iIPrfbxJFLOQEU.png)

- Snapshot — It records all your files at a given point of time so that you can look up at them anytime later. It is basically a way how GIT tracks your code history.

- Commit — The act of creating a snapshot is called a commit. One is advised to commit his code whenever there are significant changes made.

- Repository — The location or digital storage where all your files are stored.

- Head — The reference to the most recent commit is called Head.

- Branches —
  GIT follows a sort of tree like analogy for keeping track of code, when several people are collaborating on a project, the general procedure is to make a branch from the master branch, do the changes there and make a request to the master branch to merge the code.
  Basically, All commits live on some branch and there can be many branches in a single project repository.

![](https://cdn-images-1.medium.com/max/1600/0*-8t0j0AN8GL2OP9y.png)

对于新手而言，在日常工作中还是尽量使用界面工具，避免意外操作。[SourceTree]()、[GitHub Desktop]()

# Configuration | 配置

## Management | 配置管理

```sh
# 列举所有的别名与配置
git config --list

# Git 别名配置
git config --global alias.<handle> <command> git config --global alias.st status

# 设置 Git 为大小写敏感
git config --global core.ignorecase false
```

我们也可以查看常用的辅助查询命令：

```sh
# 在 Git 命令行里查看 everyday git
git help everyday

# 显示 git 常用的帮助命令
git help -g

# 获取 Git Bash 的自动补全
curl http://git.io/vfhol > ~/.git-completion.bash && echo '[ -f ~/.git-completion.bash ] && . ~/.git-completion.bash' >> ~/.bashrc

# 设置自动更正
git config --global help.autocorrect 1
```

## User | 用户配置

```sh
# 配置 HTTP 代理
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080

# 配置 Socks 代理
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'

# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --global http.sslVerify false
```

```sh
gitalias['alias.cp']='cherry-pick'
gitalias['alias.st']='status -sb'
gitalias['alias.cl']='clone'
gitalias['alias.ci']='commit'
gitalias['alias.co']='checkout'
gitalias['alias.br']='branch'
gitalias['alias.dc']='diff --cached'
gitalias['alias.lg']="log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %Cblue<%an>%Creset' --abbrev-commit --date=relative --all"
gitalias['alias.last']='log -1 --stat'
gitalias['alias.unstage']='reset HEAD --'
```

# Repository | 仓库配置与管理

## Manipulation | 操作

初始化某个仓库：

```sh
# 初始化一个版本仓库
git init

#Clone远程版本库
git clone git@xbc.me:wordpress.git
git clone https://username@github.com/username/repository.git

# 添加远程版本库origin，语法为 git remote add [shortname] [url]
git remote add origin git@xbc.me:wordpress.git

# 查看远程仓库
git remote -v
```

查看远端仓库相关信息：

```sh
# 获取所有远端引用配置
git remote
# 或者
git remote show

# 修改某个远端的地址
git remote set-url origin <URL>
```

查看仓库的统计信息：

```sh
# 查看当前仓库中的所有未打包的 objects 和磁盘占用
git count-objects --human-readable

# 从 object 数据库中删除所有不可达的 object
git gc --prune=now --aggressive
```

## .gitignore

注意忽略只对未跟踪文件有效，对于已加入版本库的文件无效，Git 内置三级忽略文件机制：

- 版本库共享式忽略文件，版本库中目录下的 .gitignore 文件作用于整个目录及子目录，会随着该版本库同其他人共享。

- 本地的针对具体版本库的独享式忽略文件，即在版本库 .git 目录下的文件 info/exclude 中设置文件忽略

- 本地的全局的独享式忽略文件，通过 Git 的配置变量 core.excludesfile 指定的一个忽略文件(指定文件名)，其设置的忽略对所有本地版本库均有效。设置方法如下(文件名可以任意设置)：

```sh
git config --global core.excludesfile ~/.gitignore
```

Git 的忽略文件遵循以下语法规则：

- 忽略文件中的空行或以井号(#)开始的行将会被忽略。
- 可以使用 Linux 通配符。例如：星号(\*)代表任意多个字符，问号(？)代表一个字符，方括号([abc])代表可选字符范围，大括号({string1,string2,...})代表可选的字符串等。
- 如果名称的最前面有一个感叹号(!)，表示例外规则，将不被忽略。
- 如果名称的最前面是一个路径分隔符(/)，表示要忽略的文件在此目录下，而子目录中的文件不忽略。
- 如果名称的最后面是一个路径分隔符(/)，表示要忽略的是此目录下该名称的子目录，而非文件(默认文件或目录都忽略)。

# Commit | 提交

```sh
# 管道命令，可用于脚本
$ git diff-tree --no-commit-id --name-only -r bd61ad98

# 查看某次提交中修改的文件
$ git show --pretty="" --name-only bd61ad98
```

## Tracking | 追踪

查看当前追踪的文件信息

```sh
# 展示所有被追踪的文件
git ls-files -t

# 展示所有未被追踪的分支
git ls-files --others

# 展示所有被忽略的文件
git ls-files --others -i --exclude-standard
git check-ignore *
git status --ignored
```

```sh
# 从工作目录中删除文件，同时会暂存信息
$ git rm [file]

# 从版本控制中移除该文件
$ git rm --cached [file]

# 本地重命名并且提交
$ git mv [file-original] [file-renamed]
```

## Stash | 贮存

git stash -- The command saves your local modifications away and reverts the working directory to match the HEAD commit. It allows you to store your uncommited modifications into a buffer area called stash, and deletes it from the branch you are working on. You may later retreive them by applying the stash.

## Undo | 撤销

# Branch | 分支

Git 中的分支实际上只是 Commit 指针。

```sh
# 命令行中查看版本树
$ git log --pretty=oneline --graph --decorate --all

# 内置的可视化界面查看版本树
$ gitk --all

# 根据提交人过滤 Commit 信息
$ git log --author="username" --pretty=format:"%h - %an, %ar : %s"
```

## Manipulation | 操作

```sh
# 创建某个分支
git branch BRANCH_NAME

# 创建并且切换到某个分支
git checkout -b BRANCH_NAME
```

### Head

分支是

## Merge | 分支合并

--force 会使用本地分支的提交覆盖远端推送分支的提交。也就是说，如果其他人在相同的分支推送了新的提交，你的这一举动将“删除”他的那些提交！就算在强制推送之前先 fetch 并且 merge 或 rebase 了也是不安全的，因为这些操作到推送之间依然存在时间差，别人的提交可能发生在这个时间差之内。使用此参数推送，如果远端有其他人推送了新的提交，那么推送将被拒绝，这种拒绝和没有加 --force 参数时的拒绝是一样的。

### cherry-pick

git cherry-pick 可以选择某一个分支中的一个或几个 commit(s) 来进行操作，譬如我们存在多个稳定开发版本，在不能完全合并分支的情况下又想把某些功能合入到某个分支，那就可以利用 cherry-pick 对已经存在的 commit 进行再次提交。注意，当执行完 cherry-pick 以后，将会生成一个新的提交；这个新的提交的哈希值和原来的不同，但标识名一样。

```sh
# 选择某个其他分支的 commit 合并到当前分支
$ git cherry-pick <commit id>

# 如果出现冲突，则类似于 Rebase 进行解决
# 手动查看冲突文件
$ git status

# 设置文件已经解决冲突
$ git add ...

# 设置 cherry-pick 继续执行
$ git cherry-pick --continue
$ git cherry-pick --quit
$ git cherry-pick --abort
```

## Rebase

顾名思义，Rebase(变基)有移花接木之效果，能将特性分支移接到主分支之上，常用于优化提交历史，或者修改本地的提交信息。

首先查看本地的 Commit 历史：

```sh
$ git log --pretty=oneline

a931ac7c808e2471b22b5bd20f0cad046b1c5d0d c
b76d157d507e819d7511132bdb5a80dd421d854f b
df239176e1a2ffac927d8b496ea00d5488481db5 a
```

运行 Git Rebase:

```sh
# 使用 Git Rebase，对最后两个提交进行操作
git rebase --interactive HEAD~2
```

```s
pick b76d157 b
squash a931ac7 c //change pick to squash

# Rebase df23917..a931ac7 onto df23917
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
#
```

and save-quitting your editor, you’ll get another editor whose contents are

```s
# This is a combination of 2 commits.
# The first commit's message is:

b

# This is the 2nd commit message:

c
```

done

```sh
$ git log --pretty=oneline
18fd73d3ce748f2a58d1b566c03dd9dafe0b6b4f b and c
df239176e1a2ffac927d8b496ea00d5488481db5 a
```

# Workflow | 协作

## Commit Message | 提交信息规范

目前规范使用较多的是 [Angular 团队的规范](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md%23-git-commit-guidelines), 继而衍生了 [Conventional Commits specification](https://conventionalcommits.org/). 很多工具也是基于此规范, 它的 message 格式如下:

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

我们通过 git commit 命令带出的 vim 界面填写的最终结果应该类似如上这个结构, 大致分为三个部分(使用空行分割):

- 标题行: 必填, 描述主要修改类型和内容
- 主题内容: 描述为什么修改, 做了什么样的修改, 以及开发的思路等等
- 页脚注释: 放 Breaking Changes 或 Closed Issues

分别由如下部分构成:

- type: commit 的类型
  - feat: 新特性
  - fix: 修改问题
  - refactor: 代码重构
  - docs: 文档修改
  - style: 代码格式修改, 注意不是 css 修改
  - test: 测试用例修改
  - chore: 其他修改, 比如构建流程, 依赖管理.
- scope: commit 影响的范围, 比如: route, component, utils, build...
- subject: commit 的概述, 建议符合 [50/72 formatting](https：//stackoverflow.com/questions/2290016/git-commit-messages-50-72-formatting)
- body: commit 具体修改内容, 可以分为多行, 建议符合 [50/72 formatting](https：//stackoverflow.com/questions/2290016/git-commit-messages-50-72-formatting)
- footer: 一些备注, 通常是 BREAKING CHANGE 或修复的 bug 的链接.

这样一个符合规范的 commit message, 就好像是一份邮件。如果你只是个人的项目, 或者想尝试一下这样的规范格式, 那么你可以为 git 设置 commit template, 每次 git commit 的时候在 vim 中带出, 时刻提醒自己:

修改 ~/.gitconfig, 添加:

```
[commit]
template = ~/.gitmessage
```

新建 ~/.gitmessage 内容可以如下:

```
# head: <type>(<scope>): <subject>
# - type: feat, fix, docs, style, refactor, test, chore
# - scope: can be empty (eg. if the change is a global or difficult to assign to a single component)
# - subject: start with verb (such as 'change'), 50-character line
#
# body: 72-character wrapped. This should answer:
# * Why was this change necessary?
# * How does it address the problem?
# * Are there any side effects?
#
# footer:
# - Include a link to the ticket, if any.
# - BREAKING CHANGE
#
```

## Statistics | 统计数据

```sh
# 查看 Git 上的个人代码量，username 需要修改为真实用户名
$ git log --author="username" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -
# added lines: 120745, removed lines: 71738, total lines: 49007

# 统计每个人增删行数
git log --format='%aN' | sort -u | while read name; do echo -en "$name\t"; git log --author="$name" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -; done

# 统计提交数前五名
$ git log --pretty='%aN' | sort | uniq -c | sort -k1 -n -r | head -n 5
```

# Git 提交信息

规范来自 [Angular's commit convension](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#)，参考 [vue commit convension](https://github.com/vuejs/vue/blob/dev/.github/COMMIT_CONVENTION.md)

我们暂时先不需严格按照上面两个规范中提到的去写提交信息(commit message )，但是建议先要做到以下几点，大家习惯了写提交信息后再考虑细化我们的这份规范。

基本格式：

- < 类型 >( 作用域 ): < 主题 >

* < 空行 >

- 提交详情

* < 空行 >

- < 脚注 >

注意每一行尽量不要超过 100 个字符(各种 git 预览工具在预览提交信息的时候会截断超过部分，不方便以后浏览提交信息)。

**类型**

- feat：特性(feature )

* fix: bug 修复

- docs: 文档

* style: 代码格式，例如丢了个分号，格式化代码，修改了缩进之类的

- refactor: 没有修改功能的情况下重写、优化、整理了代码

* test: 添加之前遗漏的测试

- chore: 维护，零星的工作。我们约定：**当该次提交不属于上面几种时，使用该类型，注意如果提交明确属于上面几种时，尽量不要使用该类型，它不够明确**

**作用域**

不同项目作用域可能划分方式可能不同，这个可能需要项目相关人员自己摸索，大家最开始使用此规范时可以使用比较大的作用域，像前端代码，如果暂时没有细分，可以考虑使用这样几个：config, component, container, state 。当然，这个视项目而定，有好的想法可以一起更新此文档。

**如果找不到好的作用域，使用 \* 占位。**

**主题**

简短的关于该次提交做了什么的信息。

- 我们约定提交信息包括后面的提交详情一律**使用中文**

* 结尾不要添加句号，点号作结尾

**提交详情**

- 使用中文

* 主题没有描述清楚的，这里可以多行详细描述

**脚注**

这里放下面几类信息

- **Breaking change**

* 发生了与之前版本不兼容的改变，如后端某个之前发布过接口发生改变

\*

- **引用 Issue、任务**

* Closes #1

- 即该提交完成任务 1。

* 如果使用 Coding.net 管理任务，如下图位置可以找到任务编号(使用其它的平台应该也有类似编号)

- ![img](https://images-cdn.shimo.im/QzC5zCcKi9gd7XTr/image.png!thumbnail)

\*

**例子**

- feat(component): 添加 ResponseViewer 组件

-

* \- 组件描述 blabla

- \- 其它描述 blabla

* fix(config): 修改 ... 插件配置导致的 ... 问题

*

- 详细描述，... 参考资料的网址 ...

-

* Closes #32

# 链接

- https://learnxinyminutes.com/docs/zh-cn/git-cn/
