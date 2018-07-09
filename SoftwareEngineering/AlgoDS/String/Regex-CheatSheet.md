[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Regex CheatSheet | 正则表达式清单

![](https://github.com/zeeshanu/learn-regex/raw/master/img/regexp-en.png)

- Symbols

  - `.` —  匹配除了换行之外的任意字符
  - `^` —  匹配字符串的首部
  - `$` —  匹配字符串的末尾

- Number

  - `*` —  匹配前述的表达式零或多次
  - `+` —  匹配前述的表达式一或多次
  - `?` —  匹配前述的表达式零或一次
  - `a{3}` - 匹配指定数目的 a
  - `a{3,}` - 匹配不少于 3 个的 a
  - `a{3,6}` - 匹配 3 到 6 个 a

- Character groups

  - `\d` —  匹配任意的数值
  - `\w` —  匹配任意的字符
  - `[XYZ]` —  数组中任一值匹配，多范围混搭的话，可以使用 `[A-Z][xyz]+` 来匹配集合中的任一字符
  - `[^a-z]` — `^` 用于进行反选，这里即表示匹配非 a-z 字符的其他值；`([^B]*)` 表示该位置是除了 B 之外的任意值。

- Advanced

  - `(x)` —  捕获圆括号，匹配 x 并且记录捕获项
  - `(x|y)` - 匹配 x 或者 y
  - `(?:x)` —  非匹配圆括号，仅匹配而不记录
  - `x(?=y)` —  预测匹配，仅匹配那些 y 之前的 x

- Flags

  - `g` —  全局搜索
  - `i` —  大小写敏感搜索
