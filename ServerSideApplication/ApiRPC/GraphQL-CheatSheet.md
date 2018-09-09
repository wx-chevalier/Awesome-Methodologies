[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# GraphQL CheatSheet | GraphQL 入门指引与实践清单

GraphQL 是由 Facebook 开源的查询语言标准，其并非具体的后端编程框架，而是一个包含了数据格式、数据关联、查询方式定义与实现等等一揽子东西的数据抽象层。GraphQL 并不能消融业务内在的复杂度，而是通过引入灵活的数据抽象层，尽量解耦前后端之间的直接关联或者阻塞；在满足日益增长不断变化的 Web/Mobile 端复杂的数据需求的同时，尽可能避免服务端内部逻辑复杂度的无序增加。GraphQL 能够用于实践 [BFF](https://www.thoughtworks.com/radar/techniques/bff-backend-for-frontends) 理念，其允许将部分数据组装/聚合地逻辑交于前端完成，即给予了前端灵活变更、快速迭代的空间，也能保证后端的相对中立性，不会疲于应付不同端或者不同界面设计的差异化数据格式要求。

![default](https://user-images.githubusercontent.com/5803001/39741543-ef8d4c50-52cc-11e8-9d16-c3f71329290a.jpg)

经典的 REST 架构模式立足资源，规范了基础的增删改查操作，却未能很好地处理资源之间的关联，及其衍生的一系列接口命名、代码分层等问题。接口名是对于某个逻辑的抽象描述，其往往会关联到某个特定的服务以及特定的多表查询语句，这就导致了接口、服务、SQL 语句与某个逻辑的强耦合性，而无法动态地应对业务逻辑的快速变化。笔者在早年间提出的 [RARF](https://parg.co/AvR) 架构模式中也探讨过，将请求再资源间响应式地流动与转换，单个资源仅需要关心与其他邻接资源的转换而不需要耦合于某个接口的返回，这也是典型的图模式。

与 REST 相比，GraphQL 为我们提供了声明式(Declarative)、分层可组合的(Hiearchial)、强类型控制(Static Type)的接口声明与交互方式；即保证了单一的查询端点，也提供了更严格、可扩展、可维护的数据查询方式(详见下文-数据模型层)。单一的查询端点能够让开发者免于考虑大量复杂的接口命名，直接使用图查询语言，也能更好地描述资源之间的关系；同时像 GraphiQL 这样的工具也能够帮我们快速生成交互式地接口文档。GraphQL 本质上是面向消费者的，客户端驱动的开发模式；它允许请求方(即客户端)而非响应方(即服务器端)决定查询的结果格式，从而返回可预测(Predictable)的结果类型，省去了客户端很多的异常情况处理与向后兼容的操作(Backwards Compatible)，提升了产品整体的健壮性。这样确实使得整个请求需要很多额外的数据参数与编码工作，但是它就将消费者与服务端解耦，并且强迫服务端遵守 Postel 法则(对自己严格，对他人宽容)。

不过 GraphQL 并非银弹，其并未缓解业务逻辑本身的复杂度，反而图查询方式在弱化各模块间的耦合的同时带来多次查询的性能损耗，在代码规范、模块划分不当的情况下可能导致渐进式地微服务切分割离也变得麻烦。于前端，直接用 Apollo Graphql React 这样的框架替代原有的状态管理，将组件直接绑定于接口，就是在破坏前端应用的自治性，与 SOLID 背道而驰，将独立的前端应用强耦合于后端。单一的端点在网络调试时也是不甚方便。Graphql 作为前端编写，维护的数据聚合层是非常好的选择，但是小型项目也可以在前端完成自聚合；将服务端的计算查询压力，传导给分布式的，性能日渐强大的客户端，也不失一个选择。

> [Backend-Boilerplate/graphql](https://github.com/wxyyxc1992/Backend-Boilerplate/blob/master/node/graphql) 或者 [Backend-Boilerplate/egg-graphql]()

## 基础使用

### GraphQL Schema

![](https://cdn-images-1.medium.com/max/1600/1*CzVPl58sR5he8UEGpYg2Zw.png)

Query 与 Mutation 是 GraphQL 的默认入口之一，

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

### Apollo GraphQL

Apollo GraphQL 为我们提供了全栈式的 GraphQL 开发工具与良好的体验，也可以看做 GraphQL 标准的一种实现方式。[graphql-tag](https://github.com/apollographql/graphql-tag) 提供了 GraphQL 的查询辅助工具，能够将某个查询转化为 GraphQL 的 AST 表示：

```js
import gql from 'graphql-tag';

const query = gql`
  {
    user(id: 5) {
      firstName
      lastName
    }
  }
`;

// query is now a GraphQL syntax tree object
console.log(query);

// {
//   "kind": "Document",
//   "definitions": [
//     {
//       "kind": "OperationDefinition",
//       "operation": "query",
//       "name": null,
```

[graphql-tools](https://github.com/apollographql/graphql-tools) 则提供了完整的 Schema 生成与合并工具，支持 Resolver, Interface, Union, Scalar:

```js
import { buildSchema, printSchema, makeExecutableSchema } from 'graphql-tools';

const sdlSchema = `...`;

// 仅仅生成 Schema 对象
const graphqlSchemaObj = buildSchema(sdlSchema);

// 生成可用于服务端的 Schema 对象
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

Schema 中定义了我们可以查询或者操作的数据属性与类型以及它们之间的关系，为开发者提供了清晰的接口数据格式定义；这种强类型定义也赋能了像 GraphiQL 这样能够自动补全的工具，并且促进了其他的譬如 IDE 集成插件、查询验证、代码生成、自动 Mock 等工具的出现。最常见的 GraphQL Schema 的表示方式就是 Schema Definition Language, SDL，即 [GraphQL specification](http://facebook.github.io/graphql/) 中提及的字符串模式，有点类似于传统的 IDL(Interface Definition Language) 或者 SDL(Schema Definition Language)。在 GraphQL 中，对于数据模型的抽象是通过 Type 来描述的，对于接口获取数据的逻辑是通过 Schema 来描述的。GraphQL 中每一个 Type 有若干 Field 组成，每个 Field 又分别指向某个 Type；Type 又可以分为 Scalar Type(标量类型) 与 Object Type(对象类型)。

GraphQL 中使用 `type` 关键字来指定类型名，类型还可以继承一到多个接口：

```gql
type Post implements Item {
  # ...
}
```

某个属性域包含了名称与类型，该类型可以是内建或自定义的标量类型，也可以是 Schema 中自定义的类型；对于非空的属性域可以使用 `!` 来标记：

```gql
age: Int!
```

较为全面的类型定义范例如下：

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

GraphQL Schema 中

```gql
type Query {}
type Mutation {}
type Subscription {}

schema {
  query: Query
  mutation: Mutation
  subscription: Subscription
}
```

# Type Basic | 类型基础

### Scalar Type | 标量类型

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

```js
import { makeExecutableSchema } from 'graphql-tools';
import GraphQLJSON from 'graphql-type-json';

const schemaString = `

scalar JSON

type Foo {
  aField: JSON
}

type Query {
  foo: Foo
}

`;

const resolveFunctions = {
  JSON: GraphQLJSON
};

const jsSchema = makeExecutableSchema({
  typeDefs: schemaString,
  resolvers: resolveFunctions
});
```

### Object Type | 对象类型

数组则是用大括号表示：

```gql
names: [String!]
```

### Type Modifier | 类型修饰

### Interface | 接口

在 GraphQL 中，`interface` 是一系列属性域的集合。

### Directive | 指令

我们可以通过指令来为任意的 Schema 定义添加额外的信息，指令并没有其固有的含义，每个 GraphQL 标准的实现都可以定义其独有功能。

```gql
name: String! @defaultValue(value: "new blogpost")
```

GraphQL 标准中规范了指令的定义与使用的方式：

```gql
directive @deprecated(
  reason: String = "No longer supported"
) on FIELD_DEFINITION | ENUM_VALUE

type ExampleType {
  newField: String
  oldField: String @deprecated(reason: "Use `newField`.")
}
```

在实际开发中，我们可以使用 graphql-tools 提供的 SchemaDirectiveVisitor 来快速开发自定义指令，譬如我们需要某个提示属性域被废弃的 @deprecated 指令：

```js
import { SchemaDirectiveVisitor } from "graphql-tools";

class DeprecatedDirective extends SchemaDirectiveVisitor {
  public visitFieldDefinition(field: GraphQLField<any, any>) {
    field.isDeprecated = true;
    field.deprecationReason = this.args.reason;
  }

  public visitEnumValue(value: GraphQLEnumValue) {
    value.isDeprecated = true;
    value.deprecationReason = this.args.reason;
  }
}
```

然后在声明 Schema

### Fragments

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

## Query & Mutation | 查询与更改

Query 与 Mutation 是 SDL 的

GraphQL 为我们提供了 Mutation 类型，以进行数据操作。

### Arguments | 参数

# 服务端开发

## 基础服务

### express-graphql

最简单的构建 GraphQL API 服务器的方式就是基于 Express，添加自定义的处理器：

```js
import express from 'express';
import graphqlHTTP from 'express-graphql';
import { buildSchema } from 'graphql';

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

// http://localhost:4000/graphql 进入 GraphiQL 交互查询工具
app.listen(4000);
```

[Prisma](https://github.com/graphcool/prisma) 是非常不错的全栈架构，开发者只需要定义好数据结构，Prisma 即能够为我们自动构建包含数据库(Docker)的 GraphQL API，Prisma 也为我们提供了便捷的云化部署方案，较为适合个人项目。

### Apollo Server

```js
const schema = `
  type Todo {
    ...
  }

  type TodoList {
    todos: [Todo]
  }

  type Query {
    todoList: TodoList
  }

  type Mutation {
    addTodo(
      text: String!
    ): Todo,
    toggleTodo(
      id: String!
    ): Todo
  }

  type Subscription {
    todoUpdated: Todo
  }

  schema {
    query: Query
    mutation: Mutation
    subscription: Subscription
  }
`;

const resolvers = {
  TodoList: {
    todos() {
      return todos;
    }
  },
  Query: {
    todoList() {
      return true;
    }
  },
  Mutation: {
    addTodo(_, { text }) {
      ...
    },
    toggleTodo(_, { id }, { ctx }) {
      ...
    }
  },
  Subscription: {
    todoUpdated: {
      ...
    }
  }
};

const executableSchema = makeExecutableSchema({
  typeDefs: schema,
  resolvers
});

router.post(
  '/graphql',
  graphqlKoa(ctx => ({
    schema: executableSchema,
    context: { ctx }
  }))
)
```

## 数据模型层

GraphQL 的一大优势就是避免了单个业务逻辑与查询语句的强绑定，SQL 具备良好的声明式可读性，但是其编程可组合性较差，我们往往会为某个涉及多资源的查询编写复杂的关联语句：

```sql
select * from user left join asset on user.asset_id = asset.id;
```

而 GraphQL 中则是将复杂的逻辑划分为多个原子查询，并在编程语言中完成数据的聚合，譬如我们需要查询用户关联的资产时，其 Schema 定义如下：

```gql
type User {
  asset: Asset
}
```

此时对于 Asset 资源，其并不需要了解完整的业务逻辑，只需要根据输入的 userId 获取到 asset 对象，对于数据的封装则是由 GraphQL 自动完成：

```js
const user = getUserById(userId);
const asset = getAssetById(user.assetId);

user.asset = asset;
```

显而易见地，这种方式可能导致单次请求处理中对于某个表的多次查询，[dataloader](https://github.com/facebook/dataloader) 即是 Facebook 开源的数据访问层辅助工具，其能够将多次对于数据库或者外部服务查询的语句合并处理。dataloader 的核心理念在于接收用户定义的批次加载函数：

```js
const DataLoader = require('dataloader');

const userLoader = new DataLoader(keys => myBatchGetUsers(keys));
```

当我们在业务逻辑中进行多次查询时，譬如：

```js
userLoader
  .load(1)
  .then(user => userLoader.load(user.invitedByID))
  .then(invitedBy => console.log(`User 1 was invited by ${invitedBy}`));

// Elsewhere in your application
userLoader
  .load(2)
  .then(user => userLoader.load(user.lastInvitedID))
  .then(lastInvited => console.log(`User 2 last invited ${lastInvited}`));

// 也可以同时加载多个数据
const [a, b] = await myLoader.loadMany(['a', 'b']);
```

dataloader 会把某个执行时间片(参考 [EventLoop](https://parg.co/AzO))内的独立查询合并处理，调用批量查询函数来进行单次查询；dataloader 还提供了缓存机制，当我们在某个执行时间片内查询了相同键的数据，dataloader 会自动返回缓存的 Promise 对象：

```js
const userLoader = new DataLoader(...)
const promise1A = userLoader.load(1)
const promise1B = userLoader.load(1)
assert(promise1A === promise1B)
```

## 分页与搜索

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

```js
import queries from './queries'
import { compose } from 'react-apollo';

class Test extends Component {
...

  render() {
    ...

    console.log(this.props.subjectsQuery, this.props.appsQuery); // should show both

    ...
  }
}

export default compose(
   graphql(queries.getSubjects, {
      name: "subjectsQuery"
   }),
   graphql(queries.getApps, {
      name: "appsQuery"
   }),
)(Test);
```

## Vue 集成

# Ecosystem | 生态圈
