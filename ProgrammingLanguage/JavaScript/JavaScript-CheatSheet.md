[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

[🔆 中文版本](./JavaScript-CheatSheet.md) | [☀️ English Version](./JavaScript-CheatSheet.en.md)

# 现代 JavaScript 语法速览与实战清单

JavaScript CheatSheet 是对于 JavaScript 学习/实践过程中的语法与技巧进行盘点，其属于 [Awesome CheatSheet](https://github.com/wxyyxc1992/Awesome-CheatSheet/) 系列，致力于提升学习速度与研发效能，即可以将其当做速查手册，也可以作为轻量级的入门学习资料。 本文参考了许多优秀的文章与代码示范，统一声明在了 [JavaScript Links](https://github.com/wxyyxc1992/Awesome-Reference/blob/master/ProgrammingLanguage/JavaScript/JavaScript-Links.md)；如果希望深入了解某方面的内容，可以继续阅读[]()，或者前往 [coding-snippets/javascript]() 查看使用 JavaScript 解决常见的数据结构与算法、设计模式、业务功能方面的代码实现。

# 基础语法

# 表达式与控制流

## 操作符

扩展操作符(Spread Syntax)，即 ...，其允许某个将某个 Iterable 对象扩展到当前位置：

```js
var mid = [3, 4];
var arr = [1, 2, ...mid, 5, 6]; // [1, 2, 3, 4, 5, 6]

var arr = [2, 4, 8, 6, 0];
var max = Math.max(...arr); // 8

var str = 'hello';
var chars = [...str]; // ["h", "e", "l", "l", "o"]
```

# 基本数据类型

JavaScript 内置了 7 种基础数据类型：null, undefined,

```js
typeof 0; // number
typeof true; // boolean
typeof 'Hello'; // string
typeof Math; // object
typeof null; // object  !!
typeof Symbol('Hi'); // symbol (New ES6)
```

## 类型判断与变量比较

### 隐式转换

```
// 0[object Object]1
{} + [] + {} + [1]

// NaN[object Object]
{} + [1,2] + {} + []
```

```js
// false，等式两侧存在 NaN，则为 false
NaN == NaN

// true, 先进行 Bool 操作转化为 false，然后两侧都变为数字 0
[] == ![]
```

## Regex: 正则表达式

对于常量正则表达式，可以使用正则字符串方式；而对于动态的正则表达式，可以使用正则表达式构造函数 :

```js
// Regular Expression Literal
const regexLiteral = /cat/;

// Regular Expression Constructor
const regexConstructor = new RegExp('cat');
```

正则表达式可以用来判断元素存在性，用于字符串替换等：

```js
const str1 = 'the cat says meow';
const hasCat = /cat/;
hasCat.test(str1);
// true

function removeCc(str) {
  return str.replace(/([A-Z])/g, ' $1');
}
removeCc('camelCase'); // 'camel Case'
removeCc('helloWorldItIsMe'); // 'hello World It Is Me'
```

* Symbols

| 符号 | 描述                                                           |
| ---- | -------------------------------------------------------------- |
| .    | (period) Matches any single character, except for line breaks. |
| \*   | Matches the preceding expression 0 or more times.              |
| +    | Matches the preceding expression 1 or more times.              |
| ?    | Preceding expression is optional (Matches 0 or 1 times).       |
| ^    | Matches the beginning of the string.                           |
| $    | Matches the end of the string.                                 |

* Character groups

| 符号   | 描述                                                                                                                      |
| ------ | ------------------------------------------------------------------------------------------------------------------------- |
| \d     | Matches any single digit character.                                                                                       |
| \w     | Matches any word character (alphanumeric & underscore).                                                                   |
| [XYZ]  | Character Set: Matches any single character from the character within the brackets. You can also do a range such as [A-Z] |
| [XYZ]+ | Matches one or more of any of the characters in the set.                                                                  |
| [^a-z] | Inside a character set, the ^ is used for negation. In this example, match anything that is NOT an uppercase letter.      |

* Flags: There are five optional flags. They can be used separately or together and are placed after the closing slash. Example: /[A-Z]/g I’ll only be introducing 2 here.

| 符号 | 描述                    |
| ---- | ----------------------- |
| g    | Global search           |
| i    | case insensitive search |

* Advanced

| 符号   | 描述                                                                      |
| ------ | ------------------------------------------------------------------------- |
| (x)    | Capturing Parenthesis: Matches x and remembers it so we can use it later. |
| (?:x)  | Non-capturing Parenthesis: Matches x and does not remembers it.           |
| x(?=y) | Lookahead: Matches x only if it is followed by y.                         |

# 集合类型

## Array: 数组

```js
const uniqueArray = arr => [...new Set(arr)];

uniqueArray([1, 2, 2, 3, 4, 4, 5]);
// [1,2,3,4,5]
```

### Array Like

### Transform: 变换

`reduce()` 函数能够将某个函数作用于数组中的每个  元素，从而将多个值转换为单个值；其典型的用法为计算数组和值，或者进行数组扁平化：

```js
// 指定初始值
let result = arr.reduce(callback, initValue);

// 计算数组和值
let sum = arr.reduce((acc, val) => {
  return acc + val;
});

// 使用 reduce 进行数组扁平化
const flatten = arr => arr.reduce((a, v) => a.concat(v), []);
// flatten([1,[2],3,4]) -> [1,2,3,4]

// 深度扁平化
const flattenDepth = (arr, depth = 1) =>
  depth != 1
    ? arr.reduce(
        (a, v) => a.concat(Array.isArray(v) ? flattenDepth(v, depth - 1) : v),
        []
      )
    : arr.reduce((a, v) => a.concat(v), []);
// flattenDepth([1,[2],[[[3],4],5]], 2) -> [1,2,[3],4,5]
```

# 函数

## Definition: 函数定义

基础的函数定义分为了函数表达式(Function Expression)与函数声明(Function Declaration)，函数表达式并不会被提升到作用域首部，而函数声明则会被提升：

```js
// Function Expression
var sum = function(a, b) {
  return a + b;
};

// Function Declaration
function sum(a, b) {
  return a + b;
}
```

## 参数

ES6 中引入了所谓的默认参数:

```js
// 传统的默认参数编写方式
function filterEvil(array, evil) {
  evil = evil || 'darth vader';
  return array.filter(item => item !== evil);
}

// ES6 默认参数
function filterEvil(array, evil = 'darth vader') {
  return array.filter(item => item !== evil);
}

// 默认参数可以用来进行必要参数检测
const isRequired = () => {
  throw new Error('param is required');
};

function filterEvil(array, evil = isRequired()) {
  return array.filter(item => item !== evil);
}
```

## Call: 函数调用

可以使用 apply 来连接两个数组：

```js
let countries = ['Moldova', 'Ukraine'];
let otherCountries = ['USA', 'Japan'];
countries.push.apply(countries, otherCountries);
console.log(countries); // => ['Moldova', 'Ukraine', 'USA', 'Japan']
```

较为全面的 JavaScript 中函数调用方式列举如下：

```js
console.log(1);
(_ => console.log(2))();
eval('console.log(3);');
console.log.call(null, 4);
console.log.apply(null, [5]);
new Function('console.log(6)')();
Reflect.apply(console.log, null, [7])
Reflect.construct(function(){console.log(8)}, []);
Function.prototype.apply.call(console.log, null, [9]);
Function.prototype.call.call(console.log, null, 10);
new (require('vm').Script)('console.log(11)‘).runInThisContext();
```

# 类与对象

# 其他

## ES6 Module: 模块

ES2015 Modules 中主要的关键字就是 `import` 与 `export`，前者负责导入模块而后者负责导出模块。完整的导出语法如下所示：

```js
// default exports
export default 42;
export default {};
export default [];
export default foo;
export default function () {}
export default class {}
export default function foo () {}
export default class foo {}

// variables exports
export var foo = 1;
export var foo = function () {};
export var bar; // lazy initialization
export let foo = 2;
export let bar; // lazy initialization
export const foo = 3;
export function foo () {}
export class foo {}

// named exports
export {foo};
export {foo, bar};
export {foo as bar};
export {foo as default};
export {foo as default, bar};

// exports from
export * from "foo";
export {foo} from "foo";
export {foo, bar} from "foo";
export {foo as bar} from "foo";
export {foo as default} from "foo";
export {foo as default, bar} from "foo";
export {default} from "foo";
export {default as foo} from "foo";
```

相对应的完整的支持的导入方式如下所示：

```js
// default imports
import foo from "foo";
import {default as foo} from "foo";

// named imports
import {bar} from "foo";
import {bar, baz} from "foo";
import {bar as baz} from "foo";
import {bar as baz, xyz} from "foo";

// glob imports
import * as foo from "foo";

// mixing imports
import foo, {baz as xyz} from "foo";
import * as bar, {baz as xyz} from "foo";
import foo, * as bar, {baz as xyz} from "foo";
```

## Error Handling: 异常处理

```js
try {
  let hello = prompt('Type hello');
  if (hello !== 'hello') {
    throw new Error("Oops, you didn't type hello");
  }
} catch (e) {
  alert(e.message);
} finally {
  alert('thanks for playing!');
}
```

本部分主要是针对于 JavaScript 中常用的数据结构类型进行分析说明。

# Array

# Object

> * [Maps,Sets And Iterators in JavaScript](http://bjorn.tipling.com/maps-sets-and-iterators-in-javascript)
> * [javascript-properties](https://mathiasbynens.be/notes/javascript-properties)

Looking at [the ECMAScript spec grammar](http://ecma-international.org/ecma-262/6.0/#sec-object-initializer), we can see that a property name can be either an *identifier name* (i.e. identifiers + reserved words), a *string literal*, or a *numeric literal*.

Identifier names are a superset of identifiers; any [valid identifier](https://mathiasbynens.be/notes/javascript-identifiers-es6) and any reserved word is a valid identifier name.

A [string literal](http://es5.github.io/x7.html#x7.8.4) is any valid string, encapsulated in either single (`'`) or double (`"`) quotes. `'foo'`, `"bar"`,`'qu\'ux'`, `""` (the empty string), and `'Ich \u2665 B\xFCcher'` are all valid string literals.

A [numeric literal](http://es5.github.io/x7.html#x7.8.3) can be either a decimal literal (e.g. `0`, `123`, `123.`, `.123`, `1.23`, `1e23`, `1E-23`, `1e+23`, `12`, but not `01`, `+123` or `-123`) or a hex integer literal (`0[xX][0-9a-fA-F]+` in regex, e.g. `0xFFFF`, `0X123`,`0xaBcD`).

```javascript
var object = {
  // `abc` is a valid identifier; no quotes are needed
  abc: 1,
  // `123` is a numeric literal; no quotes are needed
  123: 2,
  // `012` is an octal literal with value `10` and thus isn’t allowed in strict mode; but if you insist on using it, quotes aren’t needed
  012: 3,
  // `π` is a valid identifier; no quotes are needed
  π: Math.PI,
  // `var` is a valid identifier name (although it’s a reserved word); no quotes are needed
  var: 4,
  // `foo bar` is not a valid identifier name; quotes are required
  'foo bar': 5,
  // `foo-bar` is not a valid identifier name; quotes are required
  'foo-bar': 6,
  // the empty string is not a valid identifier name; quotes are required
  '': 7
};
```

[JSON](http://json.org/) only allows string literals that are quoted in double quotes (`"`) as property names.

To get or set a value from an object based on a property name, you can always use *bracket notation*. Let’s say we want to get the value for the property name `abc` from the object in the previous example this way:

```
object['abc']; // 1
```

Bracket notation can safely be used for all property names.

As you may know, there is an alternative that can be used in *some* cases: *dot notation*.

```
object.abc; // 1
```

**Dot notation can only be used when the property name is a valid identifier name.** It cannot be used for property names that are numeric literals, or for strings that aren’t valid identifier names.

## 创建添加

**Object.create()** 方法创建一个拥有指定原型和若干个指定属性的对象。

语法

```
Object.create(proto, [ propertiesObject ])
```

```
var o;

// 创建一个原型为null的空对象
o = Object.create(null);


o = {};
// 以字面量方式创建的空对象就相当于:
o = Object.create(Object.prototype);


o = Object.create(Object.prototype, {
  // foo会成为所创建对象的数据属性
  foo: { writable:true, configurable:true, value: "hello" },
  // bar会成为所创建对象的访问器属性
  bar: {
    configurable: false,
    get: function() { return 10 },
    set: function(value) { console.log("Setting `o.bar` to", value) }
}})


function Constructor(){}
o = new Constructor();
// 上面的一句就相当于:
o = Object.create(Constructor.prototype);
// 当然,如果在Constructor函数中有一些初始化代码,Object.create不能执行那些代码


// 创建一个以另一个空对象为原型,且拥有一个属性p的对象
o = Object.create({}, { p: { value: 42 } })

// 省略了的属性特性默认为false,所以属性p是不可写,不可枚举,不可配置的:
o.p = 24
o.p
//42

o.q = 12
for (var prop in o) {
   console.log(prop)
}
//"q"

delete o.p
//false

//创建一个可写的,可枚举的,可配置的属性p
o2 = Object.create({}, { p: { value: 42, writable: true, enumerable: true, configurable: true } });
```

### 复制:`Object.assign`

**Object.assign()**  方法可以把任意多个的**源对象**所拥有的**自身可枚举属性**拷贝给目标对象，然后返回目标对象。`Object.assign`  方法只会拷贝源对象自身的并且可枚举的属性到目标对象身上。注意，对于访问器属性，该方法会执行那个访问器属性的  `getter`  函数，然后把得到的值拷贝给目标对象，如果你想拷贝访问器属性本身，请使用  [`Object.getOwnPropertyDescriptor()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor)  和[`Object.defineProperties()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties)  方法。

注意，[`字符串`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String)类型和  [`symbol`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol)  类型的属性都会被拷贝。

注意，在属性拷贝过程中可能会产生异常，比如目标对象的某个只读属性和源对象的某个属性同名，这时该方法会抛出一个  [`TypeError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypeError)  异常，拷贝过程中断，已经拷贝成功的属性不会受到影响，还未拷贝的属性将不会再被拷贝。

注意， `Object.assign`  会跳过那些值为  [`null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null)  或  [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)  的源对象。

```
Object.assign(target, ...sources)
```

* 例子：浅拷贝一个对象

```
var obj = { a: 1 };
var copy = Object.assign({}, obj);
console.log(copy); // { a: 1 }
```

* 例子：合并若干个对象

```
var o1 = { a: 1 };
var o2 = { b: 2 };
var o3 = { c: 3 };

var obj = Object.assign(o1, o2, o3);
console.log(obj); // { a: 1, b: 2, c: 3 }
console.log(o1);  // { a: 1, b: 2, c: 3 }, 注意目标对象自身也会改变。
```

* 例子：拷贝 symbol 类型的属性

```
var o1 = { a: 1 };
var o2 = { [Symbol("foo")]: 2 };

var obj = Object.assign({}, o1, o2);
console.log(obj); // { a: 1, [Symbol("foo")]: 2 }
```

* 例子：继承属性和不可枚举属性是不能拷贝的

```
var obj = Object.create({foo: 1}, { // foo 是个继承属性。
    bar: {
        value: 2  // bar 是个不可枚举属性。
    },
    baz: {
        value: 3,
        enumerable: true  // baz 是个自身可枚举属性。
    }
});

var copy = Object.assign({}, obj);
console.log(copy); // { baz: 3 }
```

* 例子：原始值会被隐式转换成其包装对象

```
var v1 = "123";
var v2 = true;
var v3 = 10;
var v4 = Symbol("foo")

var obj = Object.assign({}, v1, null, v2, undefined, v3, v4);
// 源对象如果是原始值，会被自动转换成它们的包装对象，
// 而 null 和 undefined 这两种原始值会被完全忽略。
// 注意，只有字符串的包装对象才有可能有自身可枚举属性。
console.log(obj); // { "0": "1", "1": "2", "2": "3" }
```

* 例子：拷贝属性过程中发生异常

```
var target = Object.defineProperty({}, "foo", {
    value: 1,
    writeable: false
}); // target 的 foo 属性是个只读属性。

Object.assign(target, {bar: 2}, {foo2: 3, foo: 3, foo3: 3}, {baz: 4});
// TypeError: "foo" is read-only
// 注意这个异常是在拷贝第二个源对象的第二个属性时发生的。

console.log(target.bar);  // 2，说明第一个源对象拷贝成功了。
console.log(target.foo2); // 3，说明第二个源对象的第一个属性也拷贝成功了。
console.log(target.foo);  // 1，只读属性不能被覆盖，所以第二个源对象的第二个属性拷贝失败了。
console.log(target.foo3); // undefined，异常之后 assign 方法就退出了，第三个属性是不会被拷贝到的。
console.log(target.baz);  // undefined，第三个源对象更是不会被拷贝到的。
```

不过需要注意的是，assign 是浅拷贝，或者说，它是一级深拷贝，举两个例子说明：

```javascript
const defaultOpt = {
    title: {
        text: 'hello world',
        subtext: 'It\'s my world.'
    }
};

const opt = Object.assign({}, defaultOpt, {
    title: {
        subtext: 'Yes, your world.'
    }
});

console.log(opt);

// 预期结果
{
    title: {
        text: 'hello world',
        subtext: 'Yes, your world.'
    }
}
// 实际结果
{
    title: {
        subtext: 'Yes, your world.'
    }
}
```

上面这个例子中，对于对象的一级子元素而言，只会替换引用，而不会动态的添加内容。那么，其实 assign 并没有解决对象的引用混乱问题，参考下下面这个例子：

```javascript
const defaultOpt = {
    title: {
        text: 'hello world',
        subtext: 'It\'s my world.'
    }
};

const opt1 = Object.assign({}, defaultOpt);
const opt2 = Object.assign({}, defaultOpt);
opt2.title.subtext = 'Yes, your world.';

console.log('opt1:');
console.log(opt1);
console.log('opt2:');
console.log(opt2);

// 结果
opt1:
{
    title: {
        text: 'hello world',
        subtext: 'Yes, your world.'
    }
}
opt2:
{
    title: {
        text: 'hello world',
        subtext: 'Yes, your world.'
    }
}
```

# ES6-Collection

## Map

## Set

# [Immutable.js](https://facebook.github.io/immutable-js/docs/#/fromJS)

Immutable.js 虽然和 React 同期出现且跟 React 配合很爽，但它可不是 React 工具集里的（它的光芒被掩盖了），它是一个完全独立的库，无论基于什么框架都可以用它。意义在于它弥补了 Javascript 没有不可变数据结构的问题。不可变数据结构是函数式编程中必备的。前端工程师被 OOP 洗脑太久了，组件根本上就是函数用法，FP 的特点更适用于前端开发。

Javascript 中对象都是参考类型，也就是`a={a:1}; b=a; b.a=10;`你发现`a.a`也变成 10 了。可变的好处是节省内存或是利用可变性做一些事情，但是，在复杂的开发中它的副作用远比好处大的多。于是才有了浅 copy 和深 copy，就是为了解决这个问题。举个常见例子：

```
var  defaultConfig = { /* 默认值 */};

var config = $.extend({}, defaultConfig, initConfig); // jQuery用法。initConfig是自定义值

var config = $.extend(true, {}, defaultConfig, initConfig); // 如果对象是多层的，就用到deep-copy了
```

而

```javascript
var stateV1 = Immutable.fromJS({
  users: [{ name: 'Foo' }, { name: 'Bar' }]
});
var stateV2 = stateV1.updateIn(['users', 1], function() {
  return Immutable.fromJS({
    name: 'Barbar'
  });
});
stateV1 === stateV2; // false
stateV1.getIn(['users', 0]) === stateV2.getIn(['users', 0]); // true
stateV1.getIn(['users', 1]) === stateV2.getIn(['users', 1]); // false
```

如上，我们可以使用===来通过引用来比较对象，这意味着我们能够方便快速的进行对象比较，并且它能够和 React 中的 PureRenderMixin 兼容。基于此，我们可以在整个应用构建中使用 Immutable.js。也就是说，我们的 Flux Store 应该是一个具有不变性的对象，并且我们通过 将具有不变性的数据作为属性传递给我们的应用程序。

## 创建与判断

Deeply converts plain JS objects and arrays to Immutable Maps and Lists.

```
fromJS(json: any, reviver?: (k: any, v: Iterable<any, any>) => any): any
```

如果`reviver`这个属性被提供了，那么它会传入一个 Seq 对象并且被循环调用，对于顶层对象，它的默认的键为`""`。

```
Immutable.fromJS({a: {b: [10, 20, 30]}, c: 40}, function (key, value) {
  var isIndexed = Immutable.Iterable.isIndexed(value);
  return isIndexed ? value.toList() : value.toOrderedMap();
});

// true, "b", {b: [10, 20, 30]}
// false, "a", {a: {b: [10, 20, 30]}, c: 40}
// false, "", {"": {a: {b: [10, 20, 30]}, c: 40}}
```

对于转化而来的 Immutable 对象，可以通过`Iterable.is*`方法来判断其是列表还是映射或者其他数据类型。

True if `maybeIterable` is an Iterable, or any of its subclasses.`Iterable.isIterable(maybeIterable: any): boolean`

* If an `Iterable`, that same `Iterable`.
* If an Array-like, an `IndexedIterable`.
* If an Object with an Iterator, an `IndexedIterable`.
* If an Iterator, an `IndexedIterable`.
* If an Object, a `KeyedIterable`.

- Iterable.isKeyed()

True if `maybeKeyed` is a KeyedIterable, or any of its subclasses.

```
Iterable.isKeyed(maybeKeyed: any): boolean
```

* Iterable.isIndexed()

True if `maybeIndexed` is a IndexedIterable, or any of its subclasses.

```
Iterable.isIndexed(maybeIndexed: any): boolean
```

* Iterable.isAssociative()

True if `maybeAssociative` is either a keyed or indexed Iterable.

```
Iterable.isAssociative(maybeAssociative: any): boolean
```

* Iterable.isOrdered()

True if `maybeOrdered` is an Iterable where iteration order is well defined. True for IndexedIterable as well as OrderedMap and OrderedSet.

```
Iterable.isOrdered(maybeOrdered: any): boolean
```

而本身这个 Iterable 对象是可以转化为其他类型。

## Map

### 创建增删

#### update

#### merge

### 索引遍历

## Set
