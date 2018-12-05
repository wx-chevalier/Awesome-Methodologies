[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Egg 快速入门与实践手册

Eggjs 的[官方文档](https://eggjs.org/zh-cn/)已然深入浅出地介绍了 Eggjs 的用法，以及 [eggjs/examples](https://github.com/eggjs/examples) 中的示例。

当我们的代码达到十万以上时，TypeScript 和单元测试、集成测试给予了我们重构的勇气。

# 基础使用

在 [egg-core/egg.js](https://github.com/eggjs/egg-core/blob/master/lib/egg.js) 文件中可以了解到：

```js
class EggCore extends KoaApplication {}
```

Egg.js 也是遵循了约定优于配置的理念，

自动加载与依赖注入的机制带来便利的同时，也削弱了显式依赖声明的可读性，可能会使初学者一头雾水；但是沉心细读，很快就能掌握其使用。

如果我们希望在 VSCode 中调试 Egg.js 应用，那么可以参考 [Node.js CheatSheet]() 中的 VSCode 集成部分，首先安装 [vscode-eggjs](https://github.com/eggjs/vscode-eggjs)，然后为 VSCode 添加如下配置：

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch Egg",
      "type": "node",
      "request": "launch",
      "cwd": "${workspaceRoot}",
      "runtimeExecutable": "npm",
      "windows": { "runtimeExecutable": "npm.cmd" },
      // 启动我们的 egg-bin debug 并默认是 brk
      "runtimeArgs": ["run", "debug", "--", "--inspect-brk"],
      // 日志输出到 Terminal，否则启动期的日志看不到
      "console": "integratedTerminal",
      "protocol": "auto",
      // 进程重启后自动 attach
      "restart": true,
      // 因为无需再 proxy，故改回原来的 9229 端口
      "port": 9229,
      // 自动 attach 子进程
      "autoAttachChildProcesses": true
    }
  ]
}
```

## 配置

## TypeScript

# 内置对象

## Application

在 `app/extend/application.js` 文件中我们可以去扩展现有的 Application 对象，而在 `app.js` 中我们可以监听 Egg 的生命周期事件，或者进行启动前操作。

application.js 中我们可以去定义譬如数据库实例等全局对象：

```js
// app/extend/application.js
const REDIS_SYMBOL = Symbol('REDIS_SYMBOL');

module.exports = {
  get redis() {
    const options = this.config.redis;

    // 判断是否初始化，若为初始化则创建实例
    if (!this[REDIS_SYMBOL]) {
      this[REDIS_SYMBOL] = new RedisClient(options);
    }

    return this[REDIS_SYMBOL];
  }
};
```

## Context

```js
// app/extend/context.js
module.exports = {
  get redis() {
    return this.app.redis;
  }
};
```

# 请求

## 参数处理

如果希望在 Get 请求中传递数组：

```js
// http://localhost:7001/api/pocs?type[]=1&type[]=2

ctx.queries.type; // ['1','2']
```

## Session 与 Cookie

## 权限校验

推荐使用 JSON Web Token[koajs/jwt](https://github.com/koajs/jwt)

## 安全过滤

## 文件上传

```js
const awaitWriteStream = require('await-stream-ready').write;
const sendToWormhole = require('stream-wormhole');
...
const stream = await ctx.getFileStream();

const filename =
  md5(stream.filename) + path.extname(stream.filename).toLocaleLowerCase();
//文件生成绝对路径

const target = path.join(this.config.baseDir, 'app/public/uploads', filename);

//生成一个文件写入文件流
const writeStream = fs.createWriteStream(target);
try {
  //异步把文件流写入
  await awaitWriteStream(stream.pipe(writeStream));
} catch (err) {
  //如果出现错误，关闭管道
  await sendToWormhole(stream);
  throw err;
}
...
```

# 响应

## 响应体设置

更全面的 Node.js 流相关介绍参考 [Node.js CheatSheet](https://parg.co/m56)。

## 静态资源

## 模板渲染

## 国际化

# 服务调用

## 定时服务

# 数据存储

## 数据库连接

[egg-mysql](https://github.com/eggjs/egg-mysql)

## Sequelize ORM

## Knex

# 开发实践

## 异常处理

## 测试

```sh
$ TESTS=test/app/service.test.ts npm test
```

## 中间件

## GraphQL

```sh
.
├── app
│   ├── graphql
|   |   ├── common
|   |   |   └── directive.js  // 自定义directive
│   │   ├── project
│   │   │   └── schema.graphql
│   │   └── user  // 一个graphql模型
│   │       ├── connector.js
│   │       ├── resolver.js
│   │       └── schema.graphql
│   ├── model
│   │   └── user.js
│   ├── public
│   └── router.js
```

# 线上部署
