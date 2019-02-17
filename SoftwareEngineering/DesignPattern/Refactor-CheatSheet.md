[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheets)

# Clean Code, Review & Refactor | 简洁代码、审视与重构

Working Effectively with Legacy Code, by Michael Feathers. This book will give you an appreciation of what it is like to work with long-lived code bases, and how to write code now so future You (and future Your Colleagues) can be happy developers. Refactoring: Improving the Design of Existing Code, by Martin Fowler. You’ll get a whole new appreciation for the word “refactoring” after reading this book. Design Patterns: Elements of Reusable Object-Oriented Software, by Erich Gamma et al. This is also famously knows as the Gang of Four book. If you want a simpler version of this, check out Head First Design Patterns too.

- CamelCase 适用于类与接口名。
- snakeCase 适用于变量与方法名。
- SCREAMING_SNAKE_CASE 适用于常量名。
- sPoNgEbOb_SnAkE_cAsE 适用于讨厌的名字。

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.

Michael Feathers 在他的《修改代码的艺术》一书中，提出了当我们听到“遗留代码”的时候会想到什么：
如果你也和我一样，那么大抵会联想到错综复杂的、难以理清的结构，需要改变然而实际上又根本无法理解的代码；你会联想到那些不眠之夜，试图添加一个本该很容易就添加上去的特性；你会联想到自己是如何的垂头丧气，以及你的团队中的每个人对一个似乎没人管的代码库是如何打心底里感到厌烦的，这种代码简直让你生不如死。你内心深处甚至对于想一想怎样才能改善这种代码都感到痛苦。这种事情似乎太不值得我们付出努力了。
