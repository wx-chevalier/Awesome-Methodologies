[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheets)

![1_sso_vplej49wti_ubptvgq](https://user-images.githubusercontent.com/5803001/40587814-0000e1f2-6207-11e8-9e38-2e478a645c31.png)

> 📌 本文侧重于盘点 TypeScript 中类型声明与校验规则相关的知识点，对于与 ECMAScript 语法使用重合的部分建议阅读 [JavaScript CheatSheet](https://parg.co/Yha) 或者 [ECMAScript CheatSheet](https://parg.co/YhW)，对于 TypeScript 在 React/Redux 中的实践可以参阅 [React CheatSheet/TypeScript]()。需要声明的是，本文参考了 [TypeScript Links]() 中列举的很多文章或书籍，特别是官方的 [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/basic-types.html) 很值得仔细阅读。

# TypeScript CheatSheet | TypeScript 语法实践速览与实践清单

TypeScript 是由 MicroSoft 出品的 JavaScript 超集，它在兼容 JavaScript 的所有特性的基础上，附带了静态类型的支持；TypeScript 还允许我们使用尚未正式发布的 ECMAScript 的语言特性，在编译时进行类似于 Babel 这样的降级转化。

JavaScript 本身乃动态类型的语言，即是在运行时才进行类型校验；该特性赋予了其快速原型化的能力，却在构建大型 JavaScript 应用时力有不逮，其无法在编译时帮助规避可能的类型错误，也无法利用自动补全、自动重构等工具特性。TypeScript 的静态类型特性则帮助我们在编译时尽可能规避类型错误，并且 TypeScript 会尽可能地从上下文信息中进行类型推导，以避免像 Java 等静态类型语言中过于冗余的麻烦。

可以参考 [fe-boilerplat/\*-ts]() 或者 [Backend-Boilerplate/node]()，如果想了解 TypeScript 在 React 中的应用，可以参考 [React CheatSheet/TypeScript 节]()。们可以通过 npm 安装 TypeScript 的依赖包：

```sh
# 全局安装
$ npm install -g typescript

# 检测是否安装成功
$ tsc -v
Version 2.8.3
```

TypeScript 源文件一般使用 `.ts` 或者 `.tsx` 为后缀，其并不能直接运行在浏览器中而需要进行编译转化，TypeScript 的官方提供了 `tsc` 命令来进行文件编译：

```sh
$ tsc main.ts

# 同时编译多个文件
$ tsc main.ts worker.ts

# 编译当前目录下的全部 ts 文件，并不会递归编译
$ tsc *.ts

# 启动后台常驻编译程序
$ tsc main.ts --watch
```

在实际的项目中，我们也往往会在项目根目录配置 tsconfig.json 文件，来个性化配置 TypeScript 的编译参数：

```json
{
  "compilerOptions": {
    "outDir": "./dist/es",
    "declarationDir": "./dist/types",
    "target": "es5",
    "module": "commonjs",
    "jsx": "react",
    "downlevelIteration": true,
    "moduleResolution": "node",
    "allowUnreachableCode": true,
    "declaration": true,
    "experimentalDecorators": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "pretty": true,
    "skipLibCheck": true,
    "sourceMap": true,
    "strictNullChecks": true,
    "suppressImplicitAnyIndexErrors": true,
    "lib": ["dom", "es2015"],
    "baseUrl": "src"
  },
  "include": ["src/**/*", "typings/**/*"]
}
```

也可以使用 [ts-node](https://github.com/TypeStrong/ts-node) 快速地直接运行 TypeScript 文件。

# 类型机制

# Basic Types | 基础类型

TypeScript 允许我们使用 any 关键字来描述不确定的类型，编译器会自动忽略对该类型的校验：

```ts
let notSure: any = 4;
notSure = 'maybe a string instead';
notSure = false; // okay, definitely a boolean
```

与 Object 类型相比，

## Primitive Types

TypeScript 为我们提供了 number, string, boolean, 等原始类型。跟 JavaScript 一样，TypeScript 所有的数都是浮点数类型，使用 number 关键字描述：

```ts
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
```

如果我们在代码中希望显式区分 int, float 等具体的数值类型，那么可以使用 type 来建立别名：

```ts
type int = number;

type float = number;
```

布尔类型的值使用 boolean 关键字描述，包含 true/false 两个值：

```ts
let isDone: boolean = false;
```

所有的字符串类型使用 string 关键字描述，包括双引号、单引号以及模板字符串方式声明的字符串对象：

```ts
let fullName: string = `Bob Bobbington`;
let age: number = 37;
let sentence: string = `Hello, my name is ${fullName}.

I'll be ${age + 1} years old next month.`;
```

值得一提的是，就像 Java 中 Long 与 long 的区别，TypeScript 会将 Number, String, Boolean 当做包含特定属性与方法的 Object 类型进行处理，我们应该尽量避免使用复合类似以避免意外地操作：

```ts
/* WRONG */
function reverse(s: String): String;

/* OK */
function reverse(s: string): string;
```

## Enum | 枚举类型

类似于 Java 或者 C# 语言中的枚举类型，我们能够使用 enum 关键字来创建预定义好的常量集合：

```ts
enum StoryType {
  Video,
  Article,
  Tutorial
}
let st: StoryType = StoryType.Article;
```

值得一提的是，枚举类型被转化为 ES5 代码后，会使用数字来内置表示：

```js
var StoryType;
(function(StoryType) {
  StoryType[(StoryType['Video'] = 0)] = 'Video';
  StoryType[(StoryType['Article'] = 1)] = 'Article';
  StoryType[(StoryType['Tutorial'] = 2)] = 'Tutorial';
})(StoryType || (StoryType = {}));
var st = StoryType.Article;
```

我们同样可以指定枚举值对应的数值：

```js
enum StoryType {Video = 10, Article = 20, Tutorial=30}
```

从 Typescript 2.4 开始，支持了枚举类型使用字符串做为 value：

```ts
enum Colors {
  Red = 'RED',
  Green = 'GREEN',
  Blue = 'BLUE'
}
```

## Arrays & Tuple | 数组与元组

在 TypeScript 中，我们能够创建 Typed Arrays 或者 Generic Arrays，Typed Arrays 的创建方式如下：

```ts
const tags: string[] = ['javascript', 'programming'];
tags.push('typescript');
tags.forEach(function(tag) {
  console.log(`Tag ${tag}`);
});
```

Generic Arrays 的创建方式如下：

```ts
let storyLikedBy: Array<number> = [1, 2, 3];
```

我们也可以通过 interface 关键字来自定义数组类型：

```js
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ['Bob', 'Fred'];

let myStr: string = myArray[0];
```

多维数组的声明与初始化方式如下：

```ts
class Something {
  private things: Thing[][];

  constructor() {
    things = [];

    for (var i: number = 0; i < 10; i++) {
      this.things[i] = [];
      for (var j: number = 0; j < 10; j++) {
        this.things[i][j] = new Thing();
      }
    }
  }
}
```

从 2.7 版本开始，我们可以更精确的描述每一项的类型与数组总长度：

```ts
interface Data extends Array<number> {
  0: number;
  1: number;
  length: 2;
}
```

TypeScript 同时提供了 Tuple 元组类型，允许返回包含不同的已知类型的数组：

```js
let storyTitles = [
  'Learning TypeScript',
  'Getting started with TypeScript',
  'Building your first app with TypeScript'
];

let titlesAndLengths: [string, number][] = storyTitles.map(function(title) {
  let tuple: [string, number] = [title, title.length];
  return tuple;
});
```

元组类型也可以通过 interface 接口方式声明：

```ts
interface KeyValuePair extends Array<number | string> {
  0: string;
  1: number;
}
```

# 复杂类型

## 接口

## 函数

### 函数定义

TypeScript 中函数的声明与 JavaScript 中保持一致，不过其允许指定额外的类型信息：

```ts
let stories: [string, string[]][] = [];

function addStory(title: string, tags: string[]): void {
  stories.push([title, tags]);
}
```

同样可以在 Lambda 表达式中指定类型：

```ts
let sortByLength: (x: string, y: string) => number = (x, y) =>
  x.length - y.length;
tags.sort(sortByLength);
```

也可以在函数参数中指定可选参数：

```ts
function storySummary(title: string, description?: string) {
  if (description) {
    return title + description;
  } else {
    return title;
  }
}
```

或者使用默认值：

```ts
function storySummary(title: string, description: string = '') {
  return title + description;
}
```

值得一提的是，当我们确定某个函数并不返回值时，需要注意不能使用 any 来替代 void，以避免误用返回值的情形：

```ts
function fn(x: () => void) {
  var k = x(); // oops! meant to do something else
  k.doSomething(); // error, but would be OK if the return type had been 'any'
}
```

JavaScript 中并不支持函数重载，但是在 TypeScript 中我们可以通过参数的不同实现重载：

```ts
declare function createStore(
  reducer: Reducer,
  preloadedState: PreloadedState,
  enhancer: Enhancer
);
declare function createStore(reducer: Reducer, enhancer: Enhancer);
```

### Generator | 生成器

```ts
function* numbers(): IterableIterator<number> {
  console.log('Inside numbers; start');
  yield 1;
  console.log('Inside numbers; after the first yield');
  yield 2;
  console.log('Inside numbers; end');
}
```

```ts
// 迭代器结果的类型声明
interface IteratorResult<CompletedType, SuspendedType> {
  value: CompletedType | SuspendedType;
  done: this is { value: CompletedType };
}
```

### Decorator | 装饰器

TypeScript 内建支持装饰器语法，需要我们在编译配置中开启装饰器参数：

```json
{
  "compilerOptions": {
    // ...
    "experimentalDecorators": true
  }
  // ...
}
```

装饰器提供了声明式的语法来修改类的结构或者属性声明，以简单的日志装饰器为例：

```ts
export function Log() {
  return function(
    target: Object,
    propertyKey: string,
    descriptor: TypedPropertyDescriptor<any>
  ) {
    // 保留原函数引用
    let originalFunction = descriptor.value || descriptor.get;

    // 定义包裹函数
    function wrapper() {
      let startedAt = +new Date();
      let returnValue = originalFunction.apply(this);
      let endedAt = +new Date();
      console.log(
        `${propertyKey} executed in ${endedAt - startedAt} milliseconds`
      );
      return returnValue;
    }

    // 将描述对象中的函数引用指向包裹函数
    if (descriptor.value) descriptor.value = wrapper;
    else if (descriptor.get) descriptor.get = wrapper;
  };
}
```

其使用方式如下：

```js
import { Log } from './log';

export class Entity {
  // ...

  @Log()
  get title(): string {
    Entity.wait(1572);
    return this._title;
  }

  // ...
}
```

# Advanced Types | 进阶类型
