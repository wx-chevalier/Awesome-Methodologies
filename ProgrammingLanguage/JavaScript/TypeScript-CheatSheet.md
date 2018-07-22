[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

![1_sso_vplej49wti_ubptvgq](https://user-images.githubusercontent.com/5803001/40587814-0000e1f2-6207-11e8-9e38-2e478a645c31.png)

> 本文侧重于盘点 TypeScript 中类型声明与校验规则相关的知识点，对于与 ECMAScript 语法使用重合的部分建议阅读 [JavaScript CheatSheet](https://parg.co/Yha) 或者 [ECMAScript CheatSheet](https://parg.co/YhW)，对于 TypeScript 在 React/Redux 中的实践可以参阅 [React CheatSheet/TypeScript]()。需要声明的是，本文参考了 [TypeScript Links]() 中列举的很多文章或书籍，特别是官方的 [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/basic-types.html) 很值得仔细阅读。

# TypeScript CheatSheet | TypeScript 语法实践速览与实践清单

TypeScript 是由 MicroSoft 出品的 JavaScript 超集，它在兼容 JavaScript 的所有特性的基础上，附带了静态类型的支持；TypeScript 还允许我们使用尚未正式发布的 ECMAScript 的语言特性，在编译时进行类似于 Babel 这样的降级转化。

JavaScript 本身乃动态类型的语言，即是在运行时才进行类型校验；该特性赋予了其快速原型化的能力，却在构建大型 JavaScript 应用时力有不逮，其无法在编译时帮助规避可能的类型错误，也无法利用自动补全、自动重构等工具特性。TypeScript 的静态类型特性则帮助我们在编译时尽可能规避类型错误，并且 TypeScript 会尽可能地从上下文信息中进行类型推导，以避免像 Java 等静态类型语言中过于冗余的麻烦。

可以参考 [fe-boilerplat/\*-ts]() 或者 [Backend-Boilerplate/node]()，如果想了解 TypeScript 在 React 中的应用，可以参考 [React CheatSheet/TypeScript 节]()。我们可以通过 npm 安装 TypeScript 的依赖包：

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
    "module": "commonjs",
    "target": "es2015",
    "removeComments": true,
    "outDir": "./bin"
  },
  "include": ["src/**/*"]
}
```

也可以使用 [ts-node](https://github.com/TypeStrong/ts-node) 快速地直接运行 TypeScript 文件。

# 类型机制

## 类型声明

### type

`type` 关键字能够用于为基础类型(primitive type)，联合类型(union type)，以及交叉类型(intersection)取类型别名；TypeScript 还支持利用 `typeof` 关键字取变量类型，并且赋值给类型变量：

```ts
let Some = Math.round(Math.random()) ? '' : 1;

type numOrStr = typeof Some;

let foo: numOrStr;
foo = 123;
foo = 'abc';
foo = {}; // Error!
```

值得一提的是，自 2.9 版本开始，`typeof` 关键字支持动态 `import` 的类型推导：

```ts
const zipUtil: typeof import('./utils/create-zip-file') = await import('./utils/create-zip-file');
```

### interface

interface 关键字同样能够用于类型声明，用于定义对象的行为与约束；TypeScript 遵循所谓的 Structural Typing，即类型的适配与一致性依赖于实际的结构：

```js
type RestrictedStyleAttribute = "color" | "background-color" | "font-weight";

interface Foo {
  // 必要属性
  required: Type;

  // 可选属性
  optional?: Type;

  // Hash map，匹配任意字符串类型的键
  [key: string]: Type;

  // 转化为序列类型
  [id: number]: Type;

  // 匹配某些固定的键名
   [T in RestrictedStyleAttribute]: string;
}
```

譬如简单的接口定义如下：

```ts
interface Story {
  title: string;
  description?: string;
  tags: string[];
}
```

然后，任意定义包含 `title` 与 `tags` 属性的对象都会被当做 Story 接口的实例：

```ts
let story1: Story = {
  title: 'Learning TypeScript',
  tags: ['typescript', 'learning']
};
```

接口中同样可以定义函数：

```ts
interface StoryExtractor {
  extract(url: string): Story;
}

let extractor: StoryExtractor = { extract: url => story1 };
```

或者简写为：

```ts
interface StoryExtractor {
  (url: string): Story;
}

let extractor: StoryExtractor = url => story1;
```

对于接口的使用，我们将会在下文进行详细的讨论。早期版本中，interface 声明的类型能够用于扩展或者继承的场景，并且能够进行声明合并，而 type 声明的类型就无此等特性。不过自从 TypeScript 2.1 之后，type 与 interface 声明的类型都能够得到正确的错误提示，也能够应用于大部分的继承、合并的场景。

```ts
// TS Error:
// Interface:
Argument of type '{ x: number; }' is not assignable to parameter of type 'PointInterface'. Property 'y' is missing in type '{ x: number; }'.
// Type alias:
Argument of type '{ x: number; }' is not assignable to parameter of type 'PointType'. Property 'y' is missing in type '{ x: number; }'.
```

我们可以使用重复定义某个接口，其声明会自动合并；而我们无法使用 type 来重复声明相同的类型变量：

```ts
interface Box {
  height: number;
  width: number;
}

interface Box {
  scale: number;
}

const box: Box = { height: 5, width: 6, scale: 10 };
```

## 类型推导

我们可以使用 `typeof`, `instanceof`, `in` 来实现手动类型推导，`typeof` 可以获取变量的数据类型：

```ts
function foo(x: string | number) {
  if (typeof x === 'string') {
    return x; // string
  }
  return x; // number
}
```

`instanceof`  可以用于判断某个对象是否是某个类的实例：

```ts
function f1(x: B | C | D) {
  if (x instanceof B) {
    x; // B
  } else if (x instanceof C) {
    x; // C
  } else {
    x; // D
  }
}
```

`in` 用于更方便地进行 `object` 类型的推导：

```ts
interface A {
  a: number;
}
interface B {
  b: string;
}

function foo(x: A | B) {
  if ('a' in x) {
    return x.a;
  }
  return x.b;
}
```

## 类型转换

TypeScript 会在变量属性访问时进行强制空检测，这就促成了大量的前置检测代码，其在提高整体代码安全性的同时，对配置文件这样的静态数据就会造成冗余：

```ts
const config = {
  port: 8000
};

if (config) {
  console.log(config.port);
}
```

TypeScript 2.0 中提供了非空断言标志符：

```ts
console.log(config!.port);
```

## 模块化

### 模块与命名空间

TypeScript 同样支持 namespace 关键字声明的命名空间与 module 关键字声明的模块，自 TypeScript 1.5 之后，内部模块(Internal modules)即是命名空间，而外部模块(External modules)即是模块。在 ES6 Modules 普及之后，我们推荐优先使用模块机制，并且使用 import 而不是 `/// <reference ... />` 来导入模块文件。

```ts
// Usage when declaring an external module
declare module 'foo' {
  var foo: NodeJS.Foo;
  export = foo;
}
```

如果我们需要扩展 Node.js 的内置对象的方法或者属性，可以使用 interface 的自动合并的特性：

```ts
// some import
// AND/OR some export

declare namespace NodeJS {
  interface Global {
    spotConfig: any;
  }
}
```

### 类型暴露与引用

当我们希望使用那些标准的 JavaScript 代码库时，我们同样需要了解该库提供 API 的参数类型；这些类型往往定义在 `.d.ts` 声明文件中。早期的类型声明文件都需要手动地编写与导入，而 [DefinitelyTyped](http://definitelytyped.org/) 是目前最大的开源类型声明库，其会自动抓取库的类型声明文件，保障我们更加顺滑地使用 TypeScript。如果我们需要在代码中使用第三方库或者全局提供的变量，则可以使用 declare 关键字声明，譬如我们要使用 Node.js 中 process 对象，则可以进行如下的显式声明：

```ts
declare var require: (moduleId: string) => any;
declare var process: any;
```

如果是某个未包含类型声明的 NPM 库，则可以使用 declare 声明其命名空间，譬如 [antd/typings](https://parg.co/mIm) 中对于 rc 项目的引用：

```ts
declare module 'rc-queue-anim';
```

而当我们发布自己的项目时，在 package.json 中，可以通过 typings 属性指定需要暴露的类型声明文件；譬如 [redux](https://github.com/reduxjs/redux/blob/master/package.json) 的类型声明在 index.d.ts 中：

```json
{
  "typings": "./index.d.ts"
}
```

而后在 index.d.ts 文件中，导出内部类型，或者带类型描述的函数：

```js
// 类型
export type Reducer<S = any, A extends Action = AnyAction> = (state: S | undefined, action: A) => S;

// 函数
export function combineReducers<S>(reducers: ReducersMapObject<S, any>): Reducer<S>;
export function combineReducers<S, A extends Action = AnyAction>(reducers: ReducersMapObject<S, A>): Reducer<S, A>;
```

`.d.ts` 文件同样可以相互引用：

```ts
/// <reference path="custom-typings.d.ts" />
```

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

## 空类型

TypeScript 提供了 null, undefined, never, void 这几种空类型，它们都是其他类型的子类型，因为任何有类型的值都有可能是空（也就是执行期间可能没有值）。

nerver 用于处理函数的异常流程，譬如永远不会返回值或者抛出异常：

```ts
function fail(message: string): never {
  throw new Error(message);
}
```

never 类型的典型应用场景，就是处理函数中可能的不可达代码，譬如在调用上述的 `fail` 函数时，若其为非 never 类型，则会抛出不是所有的代码路径都返回值的异常：

```ts
function foo(x: string | number): boolean {
  if (typeof x === 'string') {
    return true;
  } else if (typeof x === 'number') {
    return false;
  }

  return fail('Unexhaustive!');
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

## 类

从 ES6 开始，JavaScript 内建支持使用 class 关键字来声明类，而 TypeScript 允许我们以 implements 来实现某个接口，或者以 extends 关键字来继承某个类：

```ts
class Child extends Parent implements IChild, IOtherChild {
  // 类属性
  property: Type;

  // 类属性默认值
  defaultProperty: Type = 'default value';

  // 私有属性
  private _privateProperty: Type;

  // 静态属性
  static staticProperty: Type;

  // 构造函数
  constructor(arg1: Type) {
    super(arg1);
  }

  // 私有方法
  private _privateMethod(): Type {}

  methodProperty: (arg1: Type) => ReturnType;

  overloadedMethod(arg1: Type): ReturnType;

  overloadedMethod(arg1: OtherType): ReturnType;

  overloadedMethod(arg1: CommonT): CommonReturnT {}

  // 静态方法
  static staticMethod(): ReturnType {}

  subclassedMethod(arg1: Type): ReturnType {
    super.subclassedMethod(arg1);
  }
}
```

### 继承与实现

```ts
class TextStory implements Story {
  title: string;
  tags: string[];

  static storyWithNoTags(title: string): TextStory {
    return new TextStory(title, []);
  }

  constructor(title: string, ...tags) {
    this.title = title;
    this.tags = tags;
  }

  summary() {
    return `TextStory ${this.title}`;
  }
}

// 使用静态方法创建类对象
let story = TextStory.storyWithNoTags('Learning TypeScript');

class TutorialStory extends TextStory {
  constructor(title: string, ...tags) {
    // 调用父类构造函数
    super(title, tags);
  }

  // 复写父类的方法
  summary() {
    return `TutorialStory: ${this.title}`;
  }
}
```

现在 TypeScript 允许我们同时实现多个由 type 或者 interface 声明的类型，并且能够利用交叉操作：

```ts
class Point {
  x: number;
  y: number;
}

interface Shape {
  area(): number;
}

type Perimeter = {
  perimiter(): number;
};

type RectangleShape = Shape & Perimeter & Point;

class Rectangle implements RectangleShape {}

// 等价于
class Rectangle implements Shape, Perimeter, Point {}
```

### 抽象类

TypeScript 中我们同样可以定义抽象类(Abstract class)，即包含抽象方法的类；抽象类不能够被直接初始化，需要通过子类继承并且实现抽象方法。

```ts
abstract class StoryProcessorTemplate {
  public process(url: string): Story {
    var title: string = this.extractTitle(url);
    var text: string = this.extractText(url);
    var tags: string[] = this.extractTags(text);
    return {
      title: title,
      tags: tags
    };
  }

  abstract extractTitle(url: string): string;

  abstract extractText(url: string): string;

  abstract extractTags(url: string): string[];
}
```

## JSX

# Advanced Types | 进阶类型

## Union Type | 联合类型

TypeScript 中允许定义联合类型，即指定某个变量可能为 A 类型也可能为 B 类型：

```js
let stringOrNumber: string | number = 1;
stringOrNumber = 'hello';
```

## Partial Type | 偏类型

在实际开发中，我们往往只希望用到某个接口的部分属性，特别是在实体类的定义中：

```ts
interface UserModel {
  email: string;
  password: string;
  address: string;
  phone: string;
}

class User {
  // 这里强制传入完全符合 UserModel 结构定义的对象，否则会抛出错误
  update(user: UserModel) {
    // Update user
  }
}
```

如果我们将接口属性定义为了可选属性，那么又会面临大量的空判断；TypeScript 2.1 之后为我们提供了 Partial 关键字，其内部的类型声明类似于：

```ts
type Partial<T> = { [P in keyof T]?: T[P] };
```

我们可以用其声明部分校验：

```ts
class User {
  update(user: Partial<UserModel>) {
    // Update user
  }
}

type ComponentConfig = {
  optionOne: string;
  optionTwo: string;
  optionThree: string;
};

// 这里的使用场景是传入部分配置项
export class SomeComponent {
  private _defaultConfig: Partial<ComponentConfig> = {
    optionOne: '...'
  };
}
```

Partial 同样能够用于类的声明中：

```ts
type RectangleShape = Partial<Shape & Perimeter> & Point;
```

## Generics | 泛型

泛型允许我们灵活地定义某些函数或者类接收的参数类型，更易于创建灵活而可控地可重用组件，泛型函数定义格式如下：

```ts
<T>(items :T[], callback :(item :T) => T) :T[]
```

这里我们以简单地创建数组的函数为例：

```ts
function genericFunc<T>(argument: T): T[] {
  var arrayOfT: T[] = []; // Create empty array of type T.
  arrayOfT.push(argument); // Push, now arrayOfT = [argument].
  return arrayOfT;
}

var arrayFromString = genericFunc<string>('beep');
console.log(arrayFromString[0]); // "beep"
console.log(typeof arrayFromString[0]); // String

var arrayFromNumber = genericFunc(42);
console.log(arrayFromNumber[0]); // 42
console.log(typeof arrayFromNumber[0]); // number
```

```ts
// 接口泛型
interface Pair<T1, T2> {
  first: T1;
  second: T2;
}

// 泛型类属性
class Pair<T> {
  fst: T;
  snd: T;
}
```

我们还可以指定泛型子类，即指定某个类型必须是实现某个接口或者继承自某个类：

```ts
interface HasLength {
  length: number;
}

function addLengths<T extends HasLength>(t1: T, t2: T): number {
  return t1.length + t2.length;
}

addLengths('hello', 'abc');
addLengths([1, 2, 3], [100, 11, 99]);
```

TypeScript 2.3 之后支持泛型默认参数，可以某些场景减少函数类型重载的代码量，譬如：

```ts
declare function create<T extends HTMLElement = HTMLDivElement, U = T[]>(
  element?: T,
  children?: U
): Container<T, U>;
```

## Index Types | 索引类型

TypeScript 2.1 中为我们引入了 keyof 关键字，能够获取某个类型 T 的属性名列表，其返回结果也是联合类型，譬如：

```ts
interface Person {
  name: string;
  age: number;
  location: string;
}

type K1 = keyof Person; // "name" | "age" | "location"
type K2 = keyof Person[]; // "length" | "push" | "pop" | "concat" | ...
type K3 = keyof { [x: string]: Person }; // string
```

这即是所谓的索引存取类型，或者搜索类型，我们经常会将其用于限制

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key]; // Inferred type is T[K]
}

function setProperty<T, K extends keyof T>(obj: T, key: K, value: T[K]) {
  obj[key] = value;
}

let x = { foo: 10, bar: 'hello!' };

let foo = getProperty(x, 'foo'); // number
let bar = getProperty(x, 'bar'); // string

let oops = getProperty(x, 'wargarbl'); // Error! "wargarbl" is not "foo" | "bar"

setProperty(x, 'foo', 'string'); // Error!, string expected number
```
