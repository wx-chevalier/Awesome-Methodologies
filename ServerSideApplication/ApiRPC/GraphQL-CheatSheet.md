# GraphQL CheatSheet | GraphQL 入门指引与实践清单

GraphQL 是由 Facebook 开源的查询语言，它并不是一个面向图数据库的查询语言，而是一个数据抽象层，包括数据格式、数据关联、查询方式定义与实现等等一揽子的东西。GraphQL 也并不是一个具体的后端编程框架，如果将 REST 看做适合于简单逻辑的查询标准，那么 GraphQL 可以做一个独立的抽象层，通过对于多个 REST 风格的简单的接口的排列组合提供更多复杂多变的查询方式；与 REST 相比，GraphQL 定义了更严格、可扩展、可维护的数据查询方式。GraphQL 与之前 Netflix 出品的 Falcor，都是致力于解决相同的问题：如何有效处理日益增长不断变化的 Web/Mobile 端复杂的数据需求，缓解服务端接口数目与内部逻辑复杂度的增加的困难。

![default](https://user-images.githubusercontent.com/5803001/39741543-ef8d4c50-52cc-11e8-9d16-c3f71329290a.jpg)

如上图所示，GraphQL 的特性在于单一端口与前端优先，遵循 [BFF](https://www.thoughtworks.com/radar/techniques/bff-backend-for-frontends) 的理念。首先 GraphQL 能够将多源异构的后端接口合并为单一端点（EndPoint），避免了客户端繁多的接口管理操作；其次，GraphQL 通过声明式的请求方式与强类型控制，允许请求方（即客户端）而非响应方（即服务器端）决定查询的结果格式，可以自由组合来满足需求，并且能够返回可预测的结果类型，省去了客户端很多的异常情况处理操作，提升了产品整体的健壮性。

![](https://cdn-images-1.medium.com/max/1600/1*CzVPl58sR5he8UEGpYg2Zw.png)

[Backend-Boilerplate/graphql](https://github.com/wxyyxc1992/Backend-Boilerplate/blob/master/node/graphql)

```js
const { graphql, buildSchema } = require('graphql');

// 使用 GraphQL schema language 定义 Schema
const schema = buildSchema(`
  type Query {
    hello: String
  }
`);

// 为每个 API 端点提供解析函数
const root = {
  hello: () => {
    return 'Hello world!';
  }
};

// 执行查询请求，并且获取结果
graphql(schema, '{ hello }', root).then(response => {
  console.log(response);
});
```

# Schema 定义

# 服务端开发

[Prisma](https://github.com/graphcool/prisma) 是非常不错的全栈架构，开发者只需要定义好数据结构，Prisma 即能够为我们自动构建包含数据库（Docker）的 GraphQL API，Prisma 也为我们提供了便捷的云化部署方案，较为适合个人项目。

```yaml
- graphql
    - schema
    - resolver
    - connector
    - directive
    - ducks
```

# 客户端使用

目前最为著名的 Apollo 客户端框架当属 [Apollo Client](https://github.com/apollographql/apollo-client)，其也可以很方便地与 React、Angular、Vue 等常见的前端框架集成使用。

```js
import ApolloClient from 'apollo-boost';

const client = new ApolloClient({
  uri: 'https://graphql.example.com'
});
```

[graphql-tag](https://github.com/apollographql/graphql-tag)

```js
import gql from 'graphql-tag';

client
  .query({
    query: gql`
      query TodoApp {
        todos {
          id
          text
          completed
        }
      }
    `
  })
  .then(data => console.log(data))
  .catch(error => console.error(error));
```
