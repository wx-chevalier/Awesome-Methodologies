# GraphQL CheatSheet | GraphQL 入门指引与实践清单

GraphQL 是由 Facebook 开源的查询语言，它并不是一个面向图数据库的查询语言，而是一个数据抽象层，包括数据格式、数据关联、查询方式定义与实现等等一揽子的东西。GraphQL 也并不是一个具体的后端编程框架，如果将 REST 看做适合于简单逻辑的查询标准，那么 GraphQL 可以做一个独立的抽象层，通过对于多个 REST 风格的简单的接口的排列组合提供更多复杂多变的查询方式；与 REST 相比，GraphQL 定义了更严格、可扩展、可维护的数据查询方式。GraphQL 与之前 Netflix 出品的 Falcor，都是致力于解决相同的问题：如何有效处理日益增长不断变化的 Web/Mobile 端复杂的数据需求，缓解服务端接口数目与内部逻辑复杂度的增加的困难。

![default](https://user-images.githubusercontent.com/5803001/39741543-ef8d4c50-52cc-11e8-9d16-c3f71329290a.jpg)

如上图所示，GraphQL 的特性在于单一端口与前端优先，遵循 [BFF](https://www.thoughtworks.com/radar/techniques/bff-backend-for-frontends) 的理念。GraphQL 为我们提供了声明式(Declarative)、分层可组合的(Hiearchial)、强类型控制(Static Type)的接口声明与交互方式，允许请求方(即客户端)而非响应方(即服务器端)决定查询的结果格式，从而返回可预测(Predictable)的结果类型，省去了客户端很多的异常情况处理与向后兼容的操作(Backwards Compatible)，提升了产品整体的健壮性。并且 GraphQL 能够将多源异构的后端接口合并为单一端点(EndPoint)，避免了客户端繁多的接口管理操作。

GraphQL offers a way to push all of the logic for specifying data requirements onto the client, and in return, the server will execute the (highly structured) data query against a known schema. The end result is a vastly simplified backend that also empowers the client to execute whichever queries they need.

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

## Schema

Schema 中定义了我们可以查询或者操作的数据属性与类型以及它们之间的关系，为开发者提供了清晰的接口数据格式定义；这种强类型定义也赋能了像 GraphiQL 这样能够自动补全的工具，并且促进了其他的譬如 IDE 集成插件、查询验证、代码生成、自动 Mock 等工具的出现。最常见的 GraphQL Schema 的表示方式就是 Schema Definition Language, SDL，即 [GraphQL specification](http://facebook.github.io/graphql/) 中提及的字符串模式，有点类似于传统的 IDL(Interface Definition Language) 或者 SDL(Schema Definition Language)。基础的 Schema 表示如下：

```gql
type Author {
  id: Int!
  firstName: String
  lastName: String
  posts: [Post]
}

type Post {
  id: Int!
  title: String
  author: Author
  votes: Int
}

type Query {
  posts: [Post]
  author(id: Int!): Author
}
```

## Type | 数据类型

## Query | 查询

## Mutation | 更改

GraphQL 为我们提供了 Mutation 类型，以进行数据操作。

## Fragments

Fragments 类似于组件，帮助我们解决代码中请求内容的重复问题，即可将某个元数据声明复用到不同的查询或者修改中。

```json
recentPost {
	title
	description
	author {
		...authorInfo
	}
}

fragment authorInfo as Author {
	id
	name
}
```

## Schema Representation | 表示

在实际的项目开发中，我们往往需要将标准的字符串形式的 SDL 转换成其他的 Schema 表示形式，其他常见的表示形式还包括了：GraphQl 内省查询结果与 GraphQL.js 的 GraphQLSchema 对象。GraphQL 的 API 是被要求自我注释的，每个 GraphQL API 应可以返回其结构，这就是所谓的内省(Introspection)，往往是 `__schema` 端口的返回结果：

```json
{
  __schema {
    types {
      kind
      name
      possibleTypes {
        name
      }
    }
  }
}
```

我们可以利用 graphql 库提供的 introspectionQuery 查询来进行获取：

```js
const { introspectionQuery } = require('graphql');
...
fetch('https://1jzxrj179.lp.gql.zone/graphql', {
  ...
  body: JSON.stringify({ query: introspectionQuery })
})
...
```

另一种常见的 Schema 的表示方式即是 GraphQL.js 的 GraphQLSchema 对象：

```js
const {
  GraphQLObjectType,
  GraphQLSchema,
  GraphQLNonNull,
  GraphQLInt
} = require('graphql');

const queryType = new GraphQLObjectType({
  name: 'Query',
  fields: {
    posts: {
      type: postType
    },
    author: {
      name: 'author',
      type: authorType,
      arguments: { id: { type: new GraphQLNonNull(GraphQLInt) } }
    }
  }
});

// ... postType and authorType defined similarly

const schema = new GraphQLSchema({
  query: queryType
});
```

我们可以利用 graphql 库提供的 buildSchema 或者 makeExecutableSchema 方法，将 SDL 转化为 GraphQLSchema 对象：

```js
const { buildSchema, makeExecutableSchema, printSchema } = require('graphql');

const sdlSchema = `...`;

const graphqlSchemaObj = buildSchema(sdlSchema);

const graphqlSchemaObj = makeExecutableSchema({
  typeDefs: sdlSchema,
  resolvers: {
    Query: {
      author: () => ({ firstName: 'Ada', lastName: 'Lovelace' })
    }
  }
});

// 转化为 SDL
console.log(printSchema(graphqlSchemaObj));
```

同样的，我们可以将内省的查询结果转化为 GraphQL Schema 对象：

```js
const { buildClientSchema } = require('graphql');
const fs = require('fs');

const introspectionSchemaResult = JSON.parse(fs.readFileSync('result.json'));
const graphqlSchemaObj = buildClientSchema(introspectionSchemaResult);
```

# 服务端开发

最简单的构建 GraphQL API 服务器的方式就是基于 Express，添加自定义的处理器：

```js
const express = require('express');
const graphqlHTTP = require('express-graphql');

...

const app = express();
app.use(
  '/graphql',
  graphqlHTTP({
    schema: schema,
    rootValue: root,
    graphiql: true
  })
);
app.listen(4000);
```

我们直接访问 `http://localhost:4000/graphql` 即可以进入 GraphiQL 交互查询工具。[Prisma](https://github.com/graphcool/prisma) 是非常不错的全栈架构，开发者只需要定义好数据结构，Prisma 即能够为我们自动构建包含数据库(Docker)的 GraphQL API，Prisma 也为我们提供了便捷的云化部署方案，较为适合个人项目。

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

# Ecosystem | 生态圈
