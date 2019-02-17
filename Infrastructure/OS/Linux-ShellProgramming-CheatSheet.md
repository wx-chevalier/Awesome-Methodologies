[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheets)

# Linux Shell Programming CheatSheet | Linux Shell 编程速览手册

Shell 是用户和 Linux（或者更准确的说，是用户和 Linux 内核）之间的接口程序，在提示符下输入的每个命令都由 Shell 先解释然后传给 Linux 内核。Shell 是一个命令语言解释器（command-language interpreter），拥有自己内建的 Shell 命令集；此外，Shell 也能被系统中其他有效的 Linux 实用程序和应用程序（utilities and application programs）所调用。

Shell 首先检查命令是否是内部命令，不是的话再检查是否是一个应用程序，这里的应用程序可以是 Linux 本身的实用程序，比如 ls rm，然后 Shell 试着在搜索路径(`$PATH`)里寻找这些应用程序。搜索路径是一个能找到可执行程序的目录列表。如果你键入的命令不是一个内部命令并且在路径里没有找到这个可执行文件，将会显示一条错误信息。而如果命令被成功的找到的话，Shell 的内部命令或应用程序将被分解为系统调用并传给 Linux 内核。

Bourne Again Shell (bash), 正如它的名字所暗示的，是 Bourne Shell 的扩展。bash 与 Bourne Shell 完全向后兼容，并且在 Bourne Shell 的基础上增加和增强了很多特性。bash 也包含了很多 C 和 Korn Shell 里的优点。bash 有很灵活和强大的编程接口，同时又有很友好的用户界面。

- /bin/sh (已经被 /bin/bash 所取代)
- /bin/bash (就是 Linux 默认的 Shell)
- /bin/ksh (KornShell 由 AT&T Bell lab. 发展出来的，兼容于 bash)
- /bin/tcsh (整合 C Shell ，提供更多的功能)
- /bin/csh (已经被 /bin/tcsh 所取代)
- /bin/zsh (基于 ksh 发展出来的，功能更强大的 Shell)

对于常用的 Bash 内部命令，可以参考 [Linux 常用命令与技巧清单](https://parg.co/oiT)。Shell 脚本往往以 `#!/bin/bash` 开始，用于标识系统这个脚本需要什么解释器来执行，即使用哪一种 Shell。我们也可以将路径作为解释器参数传入到具体的解释器中：

```sh
/bin/sh test.sh
/bin/php test.php
```

# 语法基础

## 变量

Shell 中存在三种变量：

- 局部变量：局部变量在脚本或命令中定义，仅在当前 shell 实例中有效，其他 shell 启动的程序不能访问局部变量。
- 环境变量：所有的程序，包括 shell 启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候 shell 脚本也可以定义环境变量。
- shell 变量：shell 变量是由 shell 程序设置的特殊变量。shell 变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了 shell 的正常运行。

定义变量时，变量名不加美元符号，并且变量名和等号之间不能有空格；使用一个定义过的变量，只要在变量名前面加美元符号即可，如：

```sh
your_name="qinjx"
echo $your_name
echo ${your_name}

# 删除变量
unset variable_name
```

变量名外面的花括号是可选的，为了帮助解释器识别变量的边界，区分譬如 `${skill}Script` 这样的情况。除了显式地直接赋值，还可以用语句给变量赋值，如：

```sh
for file in *; do
    if [ -f "$file" ]; then
        echo "$file"
    fi
done

for file in $(ls /etc)
for file in `ls /etc`
```

使用 readonly 命令可以将变量定义为只读变量，只读变量的值不能被改变：

```sh
myUrl="http://www.google.com"
readonly myUrl
```

## 数据结构

### 数值

```sh
((count++)) # increment value of variable 'count' by one.
((total+=current)) # set total = total+current.
((current>max?max=current:max=max)) # ternary expression.
```

### 字符串

字符串是 shell 编程中最常用最有用的数据类型，单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；双引号中则允许引入变量：

```sh
# 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"

# 使用单引号拼接
greeting_2='hello, '$your_name' !'
greeting_3='hello, ${your_name} !' # hello, ${your_name} !
```

字符串常见的操作如下：

```sh
string="string"

# 获取字符串长度
echo ${#string}

# 提取子字符串
${string:1:4}

# 查找子字符串
echo `expr index "$string" io`
```

### 数组

## 流程控制

### 条件

### 循环

# 文件系统

```sh
# 遍历并且判断文件是否存在
for file in Data/*.txt; do
    [ -e "$file" ] || continue
    # ... rest of the loop body
done



# 追加内容
for f in *; do
  echo "whatever" > tmpfile
  cat $f >> tmpfile
  mv tmpfile $f
done
```

```sh
if [ -d "$LINK_OR_DIR" ]; then
  if [ -L "$LINK_OR_DIR" ]; then
    # It is a symlink!
    # Symbolic link specific commands go here.
    rm "$LINK_OR_DIR"
  else
    # It's a directory!
    # Directory command goes here.
    rmdir "$LINK_OR_DIR"
  fi
fi
```

# Todos

- http://www.runoob.com/linux/linux-shell-variable.html
