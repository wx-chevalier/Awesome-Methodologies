[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# GraphQL CheatSheet | GraphQL 入门指引与实践清单

GraphQL 是由 Facebook 开源的查询语言，其并非具体的后端编程框架，而是一个包含了数据格式、数据关联、查询方式定义与实现等等一揽子东西的数据抽象层。GraphQL 并不能消融业务内在的复杂度，而是通过引入灵活的数据抽象层，尽量解耦前后端之间的直接关联或者阻塞；在满足日益增长不断变化的 Web/Mobile 端复杂的数据需求的同时，尽可能避免服务端内部逻辑复杂度的无序增加。GraphQL 能够用于实践 [BFF](https://www.thoughtworks.com/radar/techniques/bff-backend-for-frontends) 理念，其允许将部分数据组装/聚合地逻辑交于前端完成，即给予了前端灵活变更、快速迭代的空间，也能保证后端的相对中立性，不会疲于应付不同端或者不同界面设计的差异化数据格式要求。

![default](https://user-images.githubusercontent.com/5803001/39741543-ef8d4c50-52cc-11e8-9d16-c3f71329290a.jpg)

与 REST 相比，GraphQL 为我们提供了声明式(Declarative)、分层可组合的(Hiearchial)、强类型控制(Static Type)的接口声明与交互方式；即保证了单一的查询端点与更严格、可扩展、可维护的数据查询方式。GraphQL 本质上是面向消费者的，客户端驱动的开发模式；它允许请求方(即客户端)而非响应方(即服务器端)决定查询的结果格式，从而返回可预测(Predictable)的结果类型，省去了客户端很多的异常情况处理与向后兼容的操作(Backwards Compatible)，提升了产品整体的健壮性。这样确实使得整个请求需要很多额外的数据参数与编码工作，但是它就将消费者与服务端解耦，并且强迫服务端遵守 Postel 法则(对自己严格，对他人宽容)。

## 基础使用

![](https://cdn-images-1.medium.com/max/1600/1*CzVPl58sR5he8UEGpYg2Zw.png)

[Backend-Boilerplate/graphql](https://github.com/wxyyxc1992/Backend-Boilerplate/blob/master/node/graphql)

SDL 是标准的 GraphQL Schema 的表示方式，在编程中我们往往会将其转化为 GraphQL.js 的 GraphQLSchema 对象，或者其他语言中的等效描述对象。

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

另一种常见的 Schema 的表示方式即是 GraphQL.js 的 GraphQLSchema 对象，该类型对象才能够被服务端或者客户端的解析代码所使用：

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

// 转化为
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

## 典型案例

参考 [howtographql](https://www.howtographql.com/basics/3-big-picture/) 中的介绍，

# Schema Definition Language

Schema 中定义了我们可以查询或者操作的数据属性与类型以及它们之间的关系，为开发者提供了清晰的接口数据格式定义；这种强类型定义也赋能了像 GraphiQL 这样能够自动补全的工具，并且促进了其他的譬如 IDE 集成插件、查询验证、代码生成、自动 Mock 等工具的出现。最常见的 GraphQL Schema 的表示方式就是 Schema Definition Language, SDL，即 [GraphQL specification](http://facebook.github.io/graphql/) 中提及的字符串模式，有点类似于传统的 IDL(Interface Definition Language) 或者 SDL(Schema Definition Language)。基础的 Schema 表示如下：

```gql
type Post {
  id: String!
  title: String!
  publishedAt: DateTime!
  likes: Int! @default(value: 0)
  blog: Blog @relation(name: "Posts")
}

type Blog {
  id: String!
  name: String!
  description: String
  posts: [Post!]! @relation(name: "Posts")
}
```

Schema 定义的核心部分即是类型与属性域，其他的信息还包括自定义指令，譬如 `@default` 来定义默认值等等。

## Type | 数据类型

GraphQL  中使用 `type` 关键字来指定类型名，类型还可以继承一到多个接口：

```gql
type Post implements Item {
  # ...
}
```

某个属性域包含了名称与类型，该类型可以是内建或自定义的标量类型，也可以是 Schema 中自定义的类型；对于非空的属性域可以使用 `!` 来标记：

```gql
age: Int!
```

数组则是用大括号表示：

```gql
names: [String!]
```

### Scalar

GraphQL 内建提供了以下标量类型：

- Int
- Float
- String
- Boolean
- ID

Enum 是较为特殊的标量类型，其包含了可能的值：

```gql
enum Category {
  PROGRAMMING_LANGUAGES
  API_DESIGN
}
```

### Interface | 接口

在 GraphQL 中，`interface` 是一系列属性域的集合。

### Directive | 指令

我们可以通过指令来为任意的 Schema 定义添加额外的信息，譬如如下方式：

```gql
name: String! @defaultValue(value: "new blogpost")
```

指令并没有其固有的含义，每个 GraphQL 实现都可以定义其独有功能。

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

# 服务端开发

## 基础服务

最简单的构建 GraphQL API 服务器的方式就是基于 Express，添加自定义的处理器：

```js
const express = require('express');
const graphqlHTTP = require('express-graphql');

// Construct a schema, using GraphQL schema language
const schema = buildSchema(`
  type Query {
    hello: String
  }
`);

// The root provides a resolver function for each API endpoint
const root = {
  hello: () => 'Hello world!'
};

const app = express();

app.use(
  '/graphql',
  graphqlHTTP({
    schema,
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

## 查询

## 修改

# 客户端接入

目前最为著名的 Apollo 客户端框架当属 [Apollo Client](https://github.com/apollographql/apollo-client)，其也可以很方便地与 React、Angular、Vue 等常见的前端框架集成使用。

如果我们使用 Apollo 技术栈，那么还可以使用 [graphql-tag](https://github.com/apollographql/graphql-tag)

## 基础客户端

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

## React 集成

## Vue 集成

# Ecosystem | 生态圈
