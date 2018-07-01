[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

[🔆 中文版本](./JavaScript-CheatSheet.md) | [☀️ English Version](./JavaScript-CheatSheet.en.md)

# JavaScript CheatSheet | 现代 JavaScript 语法速览与实战清单

JavaScript CheatSheet 是对于 JavaScript 学习/实践过程中的语法与技巧进行盘点，其属于 [Awesome CheatSheet](https://github.com/wxyyxc1992/Awesome-CheatSheet/) 系列，致力于提升学习速度与研发效能，即可以将其当做速查手册，也可以作为轻量级的入门学习资料。 本文参考了许多优秀的文章与代码示范，统一声明在了 [JavaScript Links](https://github.com/wxyyxc1992/Awesome-Reference/blob/master/ProgrammingLanguage/JavaScript/JavaScript-Links.md)；如果希望深入了解某方面的内容，可以继续阅读[]()，或者前往 [coding-snippets/javascript]() 查看使用 JavaScript 解决常见的数据结构与算法、设计模式、业务功能方面的代码实现。

# 基础语法

# 表达式与控制流

## 操作符

### 解构操作符

```js
let a;
{a} = {a:1}

// a = 1
```

某些场景下我们需要进行深层解构，同时保存某个浅层属性值：

```js
const {
  a,
  a: { b }
} = { a: { b: 1 } };

// a = {b:1}, b = 1
```

### 扩展操作符

扩展操作符(Spread Syntax)，即 ...，其允许某个将某个 Iterable 对象扩展到当前位置：

```js
const mid = [3, 4];
const arr = [1, 2, ...mid, 5, 6]; // [1, 2, 3, 4, 5, 6]

const arr = [2, 4, 8, 6, 0];
const max = Math.max(...arr); // 8

const str = 'hello';
const chars = [...str]; // ["h", "e", "l", "l", "o"]
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

## Number | 数值类型

JavaScript 中并没有区分整型或者浮点类型，而是统一使用 Number 表示。

JavaScript 内置的 Math 对象提供了极大极小的整型值以及多个数值类型的工具函数：

```js
value = Number.MAX_SAFE_INTEGER / 10; // 900719925474099.1
value = (Number.MAX_SAFE_INTEGER / 10) * -1; // -900719925474099.1

// 向下取整
let intvalue = Math.floor(floatvalue);
let intvalue = Math.ceil(floatvalue);
let intvalue = Math.round(floatvalue);

// `Math.trunc` was added in ECMAScript 6
let intvalue = Math.trunc(floatvalue);
```

## String | 字符串类型

```js
str.substr(start[, length])
"abc".substr(1,2) // bc

str.substring(indexStart[, indexEnd])
"abc".substring(1,2) // b
```

## Regex | 正则表达式

对于常量正则表达式，可以使用正则字符串方式；而对于动态的正则表达式，可以使用正则表达式构造函数 :

```js
// Regular Expression Literal
const regexLiteral = /cat/;

// Regular Expression Constructor
const regexConstructor = new RegExp('cat');
```

- Symbols

  - `.` —  匹配除了换行之外的任意字符
  - `*` —  匹配前述的表达式零或多次
  - `+` —  匹配前述的表达式一或多次
    ? — Preceding expression is optional (Matches 0 or 1 times).
    ^ — Matches the beginning of the string.
    $ — Matches the end of the string.

Character groups

\d — Matches any single digit character.
\w — Matches any word character (alphanumeric & underscore).
[XYZ] — Character Set: Matches any single character from the character within the brackets. You can also do a range such as [A-Z][xyz]+ — Matches one or more of any of the characters in the set.
[^a-z] — Inside a character set, the ^ is used for negation. In this example, match anything that is NOT an uppercase letter.
Flags:
There are five optional flags. They can be used separately or together and are placed after the closing slash. Example: /[A-Z]/g I’ll only be introducing 2 here.
g — Global search
i — case insensitive search

Advanced

(x) — Capturing Parenthesis: Matches x and remembers it so we can use it later.
(?:x) — Non-capturing Parenthesis: Matches x and does not remembers it.
x(?=y) — Lookahead: Matches x only if it is followed by y.

### 匹配提取

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

较为常用的是 match 与 exec 方法。

```js
var s = '[description:"aoeu" uuid:"123sth"]';

var re = /\s*([^[:]+):\"([^"]+)"/g;
var m;
while ((m = re.exec(s))) {
  console.log(m[1], m[2]);
}
```

### 匹配模式

`/g` 标识标识全局匹配。`/y` 标识(Sticky 模式)表示匹配必须从 `re.lastIndex`，即上一次匹配的末尾处开始，该行为类似于 `^` 标识，不过不强制从首部开始。

```js
const str = '#foo#';
const regex = /foo/y;

regex.lastIndex = 1;
regex.test(str); // true
regex.lastIndex = 5;
regex.test(str); // false (lastIndex is taken into account with sticky flag)
regex.lastIndex; // 0 (reset after match failure)
```

如下实例较好地对比了全局模式与严格模式的区别，严格模式并不会自动忽略中间的非匹配对象：

```js
function matcher(regex, input) {
  return () => {
    const match = regex.exec(input);
    const lastIndex = regex.lastIndex;
    return { lastIndex, match };
  };
}
const input = 'haha haha haha';
const nextGlobal = matcher(/ha/g, input);
console.log(nextGlobal()); // <- { lastIndex: 2, match: ['ha'] }
console.log(nextGlobal()); // <- { lastIndex: 4, match: ['ha'] }
console.log(nextGlobal()); // <- { lastIndex: 7, match: ['ha'] }
const nextSticky = matcher(/ha/y, input);
console.log(nextSticky()); // <- { lastIndex: 2, match: ['ha'] }
console.log(nextSticky()); // <- { lastIndex: 4, match: ['ha'] }
console.log(nextSticky()); // <- { lastIndex: 0, match: null }
```

## DateTime | 时间与日期

如果是轻量级的时间与日期操作，推荐使用 [date-fns](https://github.com/date-fns/date-fns)。

```js
new Date();
// Fri Aug 21 2015 15:51:55 GMT+0800 (中国标准时间)
new Date(1293879600000);
new Date('2011-01-01T11:00:00');
new Date('2011/01/01 11:00:00');
new Date(2011, 0, 1, 11, 0, 0);
new Date('jan 01 2011,11 11:00:00');
new Date('Sat Jan 01 2011 11:00:00');
// Sat Jan 01 2011 11:00:00 GMT+0800 (中国标准时间)
new Date('sss');
new Date('2011/01/01T11:00:00');
new Date('2011-01-01-11:00:00');
new Date('1293879600000');
// Invalid Date
new Date('2011-01-01T11:00:00') - new Date('1992/02/11 12:00:12');
// 596069988000
```

# 集合类型

## Array | 数组

```js
// 使用 Array.from 创建序列数组
Array.from({
  length: 100
}).map((_, i) => i);
```

```js
const uniqueArray = arr => [...new Set(arr)];

uniqueArray([1, 2, 2, 3, 4, 4, 5]);
// [1,2,3,4,5]
```

### Array Like

```js
const arrayLike = {
  0: 'a',
  1: 'b',
  length: 2,
  *[Symbol.iterator]() {
    yield this[1];
    yield this[0];
  }
};

console.log(Array.from(arrayLike));
```

### Transform | 变换

`reduce()` 函数能够将某个函数作用于数组中的每个元素，从而将多个值转换为单个值；其典型的用法为计算数组和值，或者进行数组扁平化：

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

## Set

## Map

# 函数

## Definition | 函数定义

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

## 函数调用

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

## Object

```js
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

### 对象创建

```js
o = Object.create(Object.prototype, {
  // foo 会成为所创建对象的数据属性
  foo: { writable: true, configurable: true, value: 'hello' },
  // bar 会成为所创建对象的访问器属性
  bar: {
    configurable: false,
    get: function() {
      return 10;
    },
    set: function(value) {
      console.log('Setting `o.bar` to', value);
    }
  }
});
```

### 对象拷贝

```js
var o1 = { a: 1 };
var o2 = { b: 2 };
var o3 = { c: 3 };

var obj = Object.assign(o1, o2, o3);
console.log(obj); // { a: 1, b: 2, c: 3 }
console.log(o1); // { a: 1, b: 2, c: 3 }, 注意目标对象自身也会改变。
```

# 其他

## ES6 Module | 模块

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

## Error Handling | 异常处理

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
