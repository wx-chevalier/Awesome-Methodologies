[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Egg 快速入门与实践手册

Eggjs 的[官方文档](https://eggjs.org/zh-cn/)已然深入浅出地介绍了 Eggjs 的用法，以及 [eggjs/examples](https://github.com/eggjs/examples) 中的示例。

# 基础使用

在 [egg-core/egg.js](https://github.com/eggjs/egg-core/blob/master/lib/egg.js) 文件中可以了解到：

```js
class EggCore extends KoaApplication {}
```

Egg.js 也是遵循了约定优于配置的理念，

自动加载与依赖注入的机制带来便利的同时，也削弱了显式依赖声明的可读性，可能会使初学者一头雾水；但是沉心细读，很快就能掌握其使用。

## 配置

## TypeScript

# 内部对象

## Application

譬如数据库实例等全局对象。

```js
// app/extend/application.js
const REDIS_SYMBOL = Symbox('REDIS_SYMBOL');

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

## Session 与 Cookie

## 权限校验

## 安全过滤

# 响应

## 静态资源

## 模板渲染

## 国际化

# 服务调用

## 定时服务

# 数据存储

## 数据库连接

[egg-mysql](https://github.com/eggjs/egg-mysql)

## Sequelize ORM

# 开发实践

## 异常处理

## 测试

## 中间件

# 线上部署
