[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# TypeScript 语法实践速览

参考了 [Let's Learn TypeScript](https://parg.co/Uik), [TypeScript Cheat Sheet](https://github.com/frontdevops/typescript-cheat-sheet)

```ts
import * as React from 'react';
import formatPrice from '../utils/formatPrice';

export interface IPriceProps {
  num: number;
  symbol: '$' | '€' | '£';
}

const Price: React.SFC<IPriceProps> = ({ num, symbol }: IPriceProps) => (
  <div>
    <h3>{formatPrice(num, symbol)}</h3>
  </div>
);
```

```ts
// 使用 const 能够有效减少编译之后的代码量，参考 https://parg.co/UxX
export const enum Colors {
  RED,
  BLUE,
  GREEN
}
```

# Introduction

## Reference

* [TypeScript Deep Dive](https://basarat.gitbooks.io/typescript/content/index.html)

## Type Definition

### [DefinitelyTyped](http://definitelytyped.org/guides.html)

# DataStructure(数据类型)

## 基础类型（Basic Types）

为了使程序正常工作，我们需要以下基本数据类型：numbers 数字，structures 结构，strings 字符串，boolean 布尔型值等等。在 typescript 中，通过注入一种方便的枚举类型来帮助 typescript 支持 javascript 同样的数据类型。

## 1、Boolean 布尔型

Javascript 和 typeScript(其他语言同理)同样叫做布尔型‘boolean’的值只包含最简单的两个值真和假（true/false）。

| 1   | var isDone: boolean = false; |
| --- | ---------------------------- |
|     |                              |

## 2、Number 数字

跟 javaScript 一样，typescript 所有的数都是浮点数类型。这些浮点数获得类型‘number’。

| 1   | var height: number = 6; |
| --- | ----------------------- |
|     |                         |

## 3、string 字符串

另一种在网页和服务器端的基本数据类型是字符型数据。跟其他语言一样，我们用‘string’来表示字符类型的数据。跟 javascript 一样，typescript 也是用单引号‘’或双引号“”来包裹字符串型数据。

| 12  | var name: string = "bob";name = 'smith'; |
| --- | ---------------------------------------- |
|     |                                          |

## 4、Array 数组

同样 typescript 也允许使用数组。数组类型可以用两种方法来表示。第一种使用元素的类型和后面紧跟的大括号‘[]’来表示素组每一个元素的类型：

| 1   | var list:number[] = [1, 2, 3]; |
| --- | ------------------------------ |
|     |                                |

第二种方法是使用一个通用数组类型，Array<elemType>:

| 1   | var list:Array<number> = [1, 2, 3]; |
| --- | ----------------------------------- |
|     |                                     |

## 5、Enum 枚举

一个对 javascript 很有用的附加标准数据类型是枚举类型’enum’,和 C#一样，枚举类型是一种十分友好的给出名称和名称的值的方法。

| 12  | enum Color {Red = 1, Green, Blue};var c: Color = Color.Green; |
| --- | ------------------------------------------------------------- |
|     |                                                               |

或者甚至可以手动的设置枚举类型中所有的值。

| 12  | enum Color {Red = 1, Green = 2, Blue = 4};var c: Color = Color.Green; |
| --- | --------------------------------------------------------------------- |
|     |                                                                       |

枚举类型的一种非常方便的特性是根据枚举类型元素的数字的值可以访问到这个元素的名称。例如，我们只知道数值 2，但我们并不确定上面枚举类型具体的情况，我们可以查询其相应的名字。

| 1234 | enum Color {Red = 1, Green, Blue};var colorName: string = Color[2]; alert(colorName); |
| ---- | ------------------------------------------------------------------------------------- |
|      |                                                                                       |

[![typescript](http://14ms.net/wp-content/uploads/2015/05/typescript1-300x109.jpg)](http://14ms.net/wp-content/uploads/2015/05/typescript1.jpg)

## 6、Any 类型

我们可能需要描述我们在写应用时还不确定的变量。这些值可能来自一些动态内容，例如来自第三方库或用户。在这种情况下，我们想要在编译时跳过其类型检查，这时我们需要给它添加‘any’标签。

| 123 | var notSure: any = 4;notSure = "maybe a string instead";notSure = false; // okay, definitely a boolean |
| --- | ------------------------------------------------------------------------------------------------------ |
|     |                                                                                                        |

Any 类型是一种非常   好的方法帮助我们使用现有的 JavaScript，允许你设置在编译时跳入跳出类型检查。

如果你知道部分类型而不是全部‘any’类型也是很好帮助。例如，你有个混合类型元素的数组：

| 123 | var list:any[] = [1, true, "free"]; list[1] = 100; |
| --- | -------------------------------------------------- |
|     |                                                    |

## 7、Viod 空类型

跟‘any’相反的是‘void’类型，表示完全不含有任何类型的数据。你可能通常在没有返回值的函数中看到它。

| 123 | function warnUser(): void {    alert("This is my warning message");} |
| --- | -------------------------------------------------------------------- |
|     |                                                                      |

Typescript  的一个核心原则是类型检查关注于值的‘形状’。有时被称作‘鸭子类型’（duck typing）或‘结构类型’（structural subtyping）。在 typescript 里，接口扮演了命名这些类型的角色，同时也是在你的代码和链接你的外部工程里定义链接的有力方法。

## 1、我们的第一个接口 interface

查看接口如何工作的最简单的方法是开始一个简单的实例：

| 123456 | function printLabel(labelledObj: {label: string}) {  console.log(labelledObj.label);} var myObj = {size: 10, label: "Size 10 Object"};printLabel(myObj); |
| ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
|        |                                                                                                                                                          |

类型检查将检查调用‘printLable’。‘printLable’函数有一个带有原型‘lable’类型为字符串 string 的对象参数。需要注意的是我们传入的对象实属性际要比此要多，但是最终编辑器只是检查符合参数要求的那个属性。

我们再写一次同样的例子，这一次使用接口来描述此函数需要一个字符串类型的属性‘lable’：

| 12345678910 | interface LabelledValue {  label: string;} function printLabel(labelledObj: LabelledValue) {  console.log(labelledObj.label);} var myObj = {size: 10, label: "Size 10 Object"};printLabel(myObj); |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|             |                                                                                                                                                                                                   |

接口‘LabledValue’是一个可以用来描述我们前面例子函数需求的名称。它同样要求有一个属性叫做‘lable’并且类型是字符串 string。需要注意的是我们不需要像其他语言那样明确的声明我们传入‘printLable’实例的对象。这里主要是‘形状’其关键作用。如果传入函数的对象符合要求就行。

值得指出的是类型检查器不会要求这些属性按照一定的顺序，只要接口的要求的属性出现并且是正确的类型即可。

## 2、可选属性

不是接口的所有属性都是必须的。一些存在特定的条件或者可能根本不存在。这些可选的属性在选择袋（option bags）即传入函数的对象只有一两个属性时非常受欢迎。

这里以下面的模式为例：

| 1234567891011121314151617 | interface SquareConfig {  color?: string;  width?: number;} function createSquare(config: SquareConfig): {color: string; area: number} {  var newSquare = {color: "white", area: 100};  if (config.color) {    newSquare.color = config.color;  }  if (config.width) {    newSquare.area = config.width \* config.width;  }  return newSquare;} var mySquare = createSquare({color: "black"}); |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                           |                                                                                                                                                                                                                                                                                                                                                                                                |

可选的接口和其他接口类似，只是在每个可选属性里加入‘？’作为属性的声明。

进一步的可选属性是可以描述可能可用的属性同时捕捉到不应该被使用的属性。例如，我们传入‘createSquare’函数里的属性名字打错啦，编译器将会通知我们一个错误消息：

| 1234567891011121314151617 | interface SquareConfig {  color?: string;  width?: number;} function createSquare(config: SquareConfig): {color: string; area: number} {  var newSquare = {color: "white", area: 100};  if (config.color) {    newSquare.color = config.collor;  // Type-checker can catch the mistyped name here  }  if (config.width) {    newSquare.area = config.width \* config.width;  }  return newSquare;} var mySquare = createSquare({color: "black"}); |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                           |                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

## 3、函数类型

接口也能够描述 javascript 对象包含的更宽范围的‘形状’。在添加一个对象的属性时，接口同样能描述函数的类型。

要用接口描述一个函数的类型，我们给这个接口一个调用签名。这是一个只有参数列表和返回值的函数声明。

| 123 | interface SearchFunc {  (source: string, subString: string): boolean;} |
| --- | ---------------------------------------------------------------------- |
|     |                                                                        |

一但我们定义了之后就可以像其他接口一样使用函数类型的接口。这里，我们展示如何通过创建一个函数类型的变量然后设置他的函数值是同样的类型。

| 12345678910 | var mySearch: SearchFunc;mySearch = function(source: string, subString: string) {  var result = source.search(subString);  if (result == -1) {    return false;  }  else {    return true;  }} |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|             |                                                                                                                                                                                                |

对于函数类型的正确类型检查来说，参数的名字并不一定要一致。例如我们可以将上面的例子改写成下面这样：

| 12345678910 | var mySearch: SearchFunc;mySearch = function(source: string, subString: string) {  var result = source.search(subString);  if (result == -1) {    return false;  }  else {    return true;  }} |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|             |                                                                                                                                                                                                |

函数的参数检查时每个类型对应参数的位置互相检查。同样我们的函数表达式的返回类型暗示了它返回的值（这里是真或假）。若函数返回的是数字或字符串，类型检查其将通知我们 SearchFunc 返回值与接口不匹配。

## 4、数组类型 Array Types

跟我们用接口来描述函数类型相似，我们也能描述数组类型。数组类型有一个‘index’类型用来描述允许索引的对象的类型和一个相应的返回类型来访问索引。

| 123456 | interface StringArray {  [index: number]: string;} var myArray: StringArray;myArray = ["Bob", "Fred"]; |
| ------ | ------------------------------------------------------------------------------------------------------ |
|        |                                                                                                        |

这里有两种索引支持类型，字符串和数字。可以同时支持两种索引形式，但约束数值类型的索引必须是字符类型索引返回类型的子类型。

索引签名是一种非常强有力的描述数组和‘dictionary’模式的方法，同时它也强制所有属性必须符合它的返回类型。在下面这个例子里，属性并没有符合通用索引，类型检查器将返回一个错误：

| 1234 | interface Dictionary {  [index: string]: string;  length: number;    // error, the type of 'length' is not a subtype of the indexer} |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------ |
|      |                                                                                                                                      |

## 5、类类型 class types

实现一个接口

在 C#或 Java 等语言里一种最常用的接口是明确地限制一个类符合特定的要求，在 typescript 里也是可以实现的。

| 12345678 | interface ClockInterface {    currentTime: Date;} class Clock implements ClockInterface  {    currentTime: Date;    constructor(h: number, m: number) { }} |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
|          |                                                                                                                                                            |

你也可以在一个接口中描述实现类中的方法，我们也用‘setTime’作为例子：

| 123456789101112 | interface ClockInterface {    currentTime: Date;    setTime(d: Date);} class Clock implements ClockInterface  {    currentTime: Date;    setTime(d: Date) {        this.currentTime = d;    }    constructor(h: number, m: number) { }} |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                 |                                                                                                                                                                                                                                         |

接口描述类的公共方面而不是其私有方面。这将阻止你用它来检查一个类实例也有特殊的私有方面的类型。

## 6、静态/实例的类之间的区别

在处理类和接口时，它有助于记住一个类两种类型：静态的类型和实例的类型。您可能会注意到,如果你通过构造签名创建一个接口并尝试创建一个类实现该接口你会得到一个错误：

| 12345678 | interface ClockInterface {    new (hour: number, minute: number);} class Clock implements ClockInterface  {    currentTime: Date;    constructor(h: number, m: number) { }} |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|          |                                                                                                                                                                             |

这是因为当一个类使用接口时，只检查实例的类。由于构造函数是静态的将不会被检查。

然而，您将需要直接使用类“静态”的一面。在这个例子我们直接使用类：

| 1234567891011 | interface ClockStatic {    new (hour: number, minute: number);} class Clock  {    currentTime: Date;    constructor(h: number, m: number) { }} var cs: ClockStatic = Clock;var newClock = new cs(7, 30); |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|               |                                                                                                                                                                                                          |

## 7、扩展接口

就像类，接口可以互相扩展。这个处理的任务复制一个接口的成员到另一个，在你单独的接口为可重用的组件时有更多的自由。

| 1234567891011 | interface Shape {    color: string;} interface Square extends Shape {    sideLength: number;} var square = <Square>{};square.color = "blue";square.sideLength = 10; |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|               |                                                                                                                                                                     |

一个接口可以扩展多个接口,创建一个组合的所有接口。

| 12345678910111213141516 | interface Shape {    color: string;} interface PenStroke {    penWidth: number;} interface Square extends Shape, PenStroke {    sideLength: number;} var square = <Square>{};square.color = "blue";square.sideLength = 10;square.penWidth = 5.0; |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|                         |                                                                                                                                                                                                                                                  |

## 8、混合型

正如我们前面所提到的，接口可以描述出现在现实世界 JavaScript 的丰富的类型。由于 JavaScript 的动态和灵活的性质，你偶尔会遇到一个对象,是上面描述的一些类型的组合。

例子是一个对象,作为一个函数和一个对象的附加属性：

| 12345678910 | interface Counter {    (start: number): string;    interval: number;    reset(): void;} var c: Counter;c(10);c.reset();c.interval = 5.0; |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
|             |                                                                                                                                          |

与第三方 JavaScript 交互，您可能需要使用像上面完全描述的形状类型的模式。

传统 JavaScript 关注功能和基于原型的继承为基本手段,建立可重用的组件，但这可能会觉得有点尴尬对程序员来说当舒适使用面向对象的方法来临时,类所继承的功能和对象是由其创建的。从 ECMAScript 6，JavaScript 程序员可以使用这种面向对象的基于类的方法构建应用程序。在 typescript 里，我们现在允许开发人员使用这些技术，无需等待下一个版本的 JavaScript 在所有主要的浏览器和平台 JavaScript 和编译它们。

## 1、类

让我们看看一个简单的基于类的例子：

| 1234567891011 | class Greeter {    greeting: string;    constructor(message: string) {        this.greeting = message;    }    greet() {        return "Hello, " + this.greeting;    }} var greeter = new Greeter("world"); |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|               |                                                                                                                                                                                                             |

如果你用过 C#或 java 的话对于以上语法会很熟悉。我们生命了一个新类‘Greeter’，这个类有三个成员，一个属性‘greeting’，一个构造器，一个方法‘greet’。

你需要注意的是当指定类的一个成员时在前面加上‘this’，这表示,这是一个成员的访问。

在最后一行我们构造一个 Greeter 的实例使用“new”。这要求我们前面定义的构造函数，创建一个新对象的 Greeter 形状,并运行构造函数来初始化它。

## 2、继承

在  TypeScript，我们可以使用常见的面向对象模式。当然，基于类的编程是最基本的模式之一是能够使用继承扩展现有的类来创建新的

让我们看一个例子:

| 1234567891011121314151617181920212223242526272829 | class Animal {    name:string;    constructor(theName: string) { this.name = theName; }    move(meters: number = 0) {        alert(this.name + " moved " + meters + "m.");    }} class Snake extends Animal {    constructor(name: string) { super(name); }    move(meters = 5) {        alert("Slithering...");        super.move(meters);    }} class Horse extends Animal {    constructor(name: string) { super(name); }    move(meters = 45) {        alert("Galloping...");        super.move(meters);    }} var sam = new Snake("Sammy the Python");var tom: Animal = new Horse("Tommy the Palomino"); sam.move();tom.move(34); |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                                                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

这个例子中有很多其他语言所包含的继承特性。在这里,我们看到使用‘extends’的关键字来创建一个子类。你可以看到这个“Horse”和“Snake”子类基类‘Animal’,获得其功能。

示例还展示了能够覆盖基类中的方法与专门子类的方法。这里‘Snake’和‘Horse’都创建了一个‘move’方法覆盖了基类‘Animal’的‘move’，赋给每个类特定的功能。

## 3、私有/公邮修改器

默认的公有

你可能注意到上面的例子我们没有使用‘public’关键字使类的每一个成员都可见。类似 C#的语言需要明确的关键字‘public’来时每个成员可见。在 TypeScript，默认每个成员都是公有的。

你还可以马克私有成员,所以你控制什么是公开可见的类。我们可以写前一节的“Animal”类如下：

| 1234567 | class Animal {    private name:string;    constructor(theName: string) { this.name = theName; }    move(meters: number) {        alert(this.name + " moved " + meters + "m.");    }} |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|         |                                                                                                                                                                                      |

## 4、智能化私有

TypeScript 是一种结构化类型系统。当我们比较两种不同类型时，无需关注他们从哪里来，如果每个成员的类型相兼容，那么就说他们兼容。

当我们比较私有类型的成员时却有所不同。对于两种兼容的类型来说，如果其中一个是私有成员，那另一额也一定得是起源于相同的声明的私有成员。

让我们来看一个例子：

| 1234567891011121314151617181920 | class Animal {    private name:string;    constructor(theName: string) { this.name = theName; }} class Rhino extends Animal { constructor() { super("Rhino"); }} class Employee {    private name:string;    constructor(theName: string) { this.name = theName; } } var animal = new Animal("Goat");var rhino = new Rhino();var employee = new Employee("Bob"); animal = rhino;animal = employee; //error: Animal and Employee are not compatible |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                                 |                                                                                                                                                                                                                                                                                                                                                                                                                                                    |

在这个例子里，有两个类‘Animal’和‘Rhino’，‘Rhino’是‘Animal’的子类。我们还有一个新类‘Employee’，它的结构和‘Animal’完全相同。我们创建这些类的一些实例然后把他们互相赋值来看看会发生什么。由于‘Animal’和‘Rhino’都有来自‘Animal’的同一声明的私有属性名字（name）：字符串（string），他们是兼容的。然而，对于‘Employee’是不同的。当我们试着将一个‘Employee’赋值给‘Animal’发生了一个类型不兼容的错误。甚至‘Employee’有同样的叫‘name’的私有成员，但这个跟‘Animal’里创建的并不是同一个。

## 5、参数属性

关键字“public”和“private”也给你一个通过创建参数属性为创建和初始化类的成员的简写。属性让您可以创建并初始化一个成员在同一步骤里。这里有一个前面的例子进一步修订。注意我们完全放弃“theName”,构造器里创建和初始化’name‘成员时只使用缩短的‘private name:string‘参数。

| 123456 | class Animal {    constructor(private name: string) { }    move(meters: number) {        alert(this.name + " moved " + meters + "m.");    }} |
| ------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
|        |                                                                                                                                              |

以这种方式使用“private”创建并初始化一个私有成员，公有成员也同理。

##  6、存取标记

TypeScript 支持 getter / setter 的拦截访问一个对象的成员。这将给你一种更精细的成员访问对象的方式。

让我们实验衣蛾使用’get‘和’set‘的类。首先，我们从一个不用获取和设置的例子开始。

| 123456789 | class Employee {    fullName: string;} var employee = new Employee();employee.fullName = "Bob Smith";if (employee.fullName) {    alert(employee.fullName);} |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
|           |                                                                                                                                                             |

虽然允许人们直接改变 fullName 十分方便，但是如果我们人们可以随意改变名字这可能给我们带来麻烦。

在这种情况下，我们在修改 Employee 之前确保有个可用的密码。将直接进入 fullName 替换成检查密码的’set‘。我们添加一个相应的“get”让前面的示例无缝地继续工作。

| 123456789101112131415161718192021222324 | var passcode = "secret passcode"; class Employee {    private \_fullName: string;     get fullName(): string {        return this.\_fullName;    }     set fullName(newName: string) {        if (passcode && passcode == "secret passcode") {            this.\_fullName = newName;        }        else {            alert("Error: Unauthorized update of employee!");        }    }} var employee = new Employee();employee.fullName = "Bob Smith";if (employee.fullName) {    alert(employee.fullName);} |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|                                         |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |

现在我们的访问器检查密码来验证我们自己，我们可以修改密码来看看当密码不匹配时警告框将告诉我们没有更新 Employee 的权限。

## 7、静态属性

到目前为止，我们只讨论了类的实例成员。当对象实例化的时候将会出现。我们也能建立类的静态 static 成员。将在类本身出现而不是类的实例里。在这个例子里，我们在’origin‘上使用’static‘，作为 girds 的通用值。每个实例通过类里预先设置好的名字来访问。跟’this‘类似，在静态实例访问处我们使用’Grid‘。

| 123456789101112131415 | class Grid {    static origin = {x: 0, y: 0};    calculateDistanceFromOrigin(point: {x: number; y: number;}) {        var xDist = (point.x - Grid.origin.x);        var yDist = (point.y - Grid.origin.y);        return Math.sqrt(xDist _ xDist + yDist _ yDist) / this.scale;    }    constructor (public scale: number) { }} var grid1 = new Grid(1.0);  // 1x scalevar grid2 = new Grid(5.0);  // 5x scale alert(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));alert(grid2.calculateDistanceFromOrigin({x: 10, y: 10})); |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

## 8、进阶技术

### 8.1 构造函数

当你在 TypeScript 声明一个类，你实际上是同时创建多个声明。第一个是类的实例的类型。

| 12345678910111213 | class Greeter {    greeting: string;    constructor(message: string) {        this.greeting = message;    }    greet() {        return "Hello, " + this.greeting;    }} var greeter: Greeter;greeter = new Greeter("world");alert(greeter.greet()); |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                   |                                                                                                                                                                                                                                                     |

在这里,当我们说“var greeter”,我们使用 Greeter 的类型实例的类。这几乎是其他面向对象语言的程序员的第二天性。

我们还创建另一个值,我们称之为*constructor function\*\*。*当我们’new‘类的实例时的函数调用。在实践中看看这是什么样子，让我们看一下上面的例子创建的 JavaScript：

| 12345678910111213 | var Greeter = (function () {    function Greeter(message) {        this.greeting = message;    }    Greeter.prototype.greet = function () {        return "Hello, " + this.greeting;    };    return Greeter;})(); var greeter;greeter = new Greeter("world");alert(greeter.greet()); |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                   |                                                                                                                                                                                                                                                                                       |

这里‘var Greeter’将设置构造函数。当我们调用‘new’操作符并运行该函数时，我们得到一个类的实例。构造函数也包含类的所有静态成员。另一种认识每个类的方法是其由实例 instance 和静态 static 两方面组成。

下面我们修改一下这个例子来看看这有什么不同：

| 123456789101112131415161718192021 | class Greeter {    static standardGreeting = "Hello, there";    greeting: string;    greet() {        if (this.greeting) {            return "Hello, " + this.greeting;        }        else {            return Greeter.standardGreeting;        }    }} var greeter1: Greeter;greeter1 = new Greeter();alert(greeter1.greet()); var greeterMaker: typeof Greeter = Greeter;greeterMaker.standardGreeting = "Hey there!";var greeter2:Greeter = new greeterMaker();alert(greeter2.greet()); |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |

这个例子里，‘greeter’的工作方式和前面相同。我们实例化‘Greeter’类，然后使用这个对象。这在前面我们见过。

下一步，我们直接使用类。这里我们建一个变量叫做‘greeterMaker’。这个变量将会保存这个类自己，或者说这是另一种构造函数。这里我们使用‘typeof Greeter’，意思是‘把 Greeter 类他自己的类型给我’而不是实例化这个类型。或者更精确的说是“把符号 Greeter 的类型给我”，即构造函数的类型。这将包含创建 Greeter 类实例的构造函数的所有静态成员.。跟前面调用一样，我们通过使用‘new’操作符在‘greeterMaker’前面来创建新的‘Greeter’实例。

### 8.2 使用一个类作为一个接口

正如我们在前一节中说，类声明创建两件事：一种代表类的实例和构造函数。因为类创建类型，你可以在使用接口的地方使用它：

| 12345678910 | class Point {    x: number;    y: number;} interface Point3d extends Point {    z: number;} var point3d: Point3d = {x: 1, y: 2, z: 3}; |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------- |
|             |                                                                                                                                        |
