[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Node.js CheatSheet | Node.js 语法基础、实践技巧与开源框架清单

Node.js 的包管理，或者说依赖管理使用了语义化版本的规范，版本的发布分为四个不同的层次：使用 1.0.0 表示新发布，使用 1.0.1 这样第三位数字表示错误修复等小版本更新；使用 1.1.0 这样的第二位数字表示新特性等兼容性更新；使用 2.0.0 这样第一位数字表示大版本的更新。相对应地，在 package.json 声明依赖版本时，我们也可以指定不同的兼容范围：

* Patch releases：1.0 或者 1.0.x 或者 ~1.0.4

* Minor releases: 1 或者 1.x 或者 ^1.0.4

* Major releases: \* 或者 x

# IO

## Stream

## 文件读写

`__dirname` 指向当前文件所在目录。

我们可以使用 join 与 resolve 两个辅助函数来进行路径构造，区别在于 join 只是进行简单连接，而 resolve 会进行根路径重置：

```js
path.join('/a', '/b'); // Outputs '/a/b'

path.resolve('/a', '/b'); // Outputs '/b'
```
