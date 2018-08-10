[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Linux Shell Programming CheatSheet | Linux Shell 编程速览手册

Shell 是用户和 Linux（或者更准确的说，是用户和 Linux 内核）之间的接口程序。你在提示符下输入的每个命令都由 Shell 先解释然后传给 Linux 内核。Shell 是一个命令语言解释器（command-language interpreter）。拥有自己内建的 Shell 命令集。此外，Shell 也能被系统中其他有效的 Linux 实用程序和应用程序（utilities and application programs）所调用。

Shell 首先检查命令是否是内部命令，不是的话再检查是否是一个应用程序，这里的应用程序可以是 Linux 本身的实用程序，比如 ls rm，然后 Shell 试着在搜索路径($PATH)里寻找这些应用程序。搜索路径是一个能找到可执行程序的目录列表。如果你键入的命令不是一个内部命令并且在路径里没有找到这个可执行文件，将会显示一条错误信息。而如果命令被成功的找到的话，Shell 的内部命令或应用程序将被分解为系统调用并传给 Linux 内核。

Bourne Again Shell (bash), 正如它的名字所暗示的，是 Bourne Shell 的扩展。bash 与 Bourne Shell 完全向后兼容，并且在 Bourne Shell 的基础上增加和增强了很多特性。bash 也包含了很多 C 和 Korn Shell 里的优点。bash 有很灵活和强大的编程接口，同时又有很友好的用户界面。

- /bin/sh (已经被 /bin/bash 所取代)
- /bin/bash (就是 Linux 默认的 Shell)
- /bin/ksh (KornShell 由 AT&T Bell lab. 发展出来的，兼容于 bash)
- /bin/tcsh (整合 C Shell ，提供更多的功能)
- /bin/csh (已经被 /bin/tcsh 所取代)
- /bin/zsh (基于 ksh 发展出来的，功能更强大的 Shell)

对于常用的 Bash 内部命令，可以参考 [Linux 常用命令与技巧清单](https://parg.co/oiT)。

# 语法基础

## 变量

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
