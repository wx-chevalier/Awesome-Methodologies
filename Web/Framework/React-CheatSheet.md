[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# 2018 React 设计理念，语法纵览与实践清单

这是一篇非常冗长的文章，是笔者 []() 系列的提炼。

# Principles | 设计理念

## 小而美的视图层

任何一个编程生态都会经历三个阶段，第一个是原始时期，由于需要在语言与基础的 API 上进行扩充，这个阶段会催生大量的 Tools。第二个阶段，随着做的东西的复杂化，需要更多的组织，会引入大量的设计模式啊，架构模式的概念，这个阶段会催生大量的 Frameworks。第三个阶段，随着需求的进一步复杂与团队的扩充，就进入了工程化的阶段，各类分层 MVC，MVP，MVVM 之类，可视化开发，自动化测试，团队协同系统。这个阶段会出现大量的小而美的 Library。React 与 VueJS 都是所谓小而美的视图层 Library，而不是 Angular 2 这样兼容并包的 Frameworks。

React 并没有提供很多复杂的概念与繁琐的 API，而是以最少化为目标，专注于提供清晰简洁而抽象的视图层解决方案，同时对于复杂的应用场景提供了灵活的扩展方案，典型的譬如根据不同的应用需求引入 MobX/Redux 这样的状态管理工具。React 在保证较好的扩展性、对于进阶研究学习所需要的基础知识完备度以及整个应用分层可测试性方面更胜一筹。

不过很多人对 React 的意见在于其陡峭的学习曲线与较高的上手门槛，特别是 JSX 以及大量的 ES6 语法的引入使得很多的传统的习惯了 jQuery 语法的前端开发者感觉学习成本可能会大于开发成本。与之相比 Vue 则是典型的所谓渐进式库，即可以按需渐进地引入各种依赖，学习相关地语法知识。不过这种自由也是有利有弊，所谓磨刀不误砍材工，React 相对较严格的规范对团队内部的代码样式风格的统一、代码质量保障等会有很好的加成。一言蔽之，笔者个人觉得 Vue 会更容易被纯粹的前端开发者的接受，毕竟从直接以 HTML 布局与 jQuery 进行数据操作切换到指令式的支持双向数据绑定的 Vue 代价会更小一点，特别是对现有代码库的改造需求更少，重构代价更低

React 及其相对严格的规范
可能会更容易被后端转来的开发者接受，可能在初学的时候会被一大堆概念弄混，但是熟练之后这种严谨的组件类与成员变量/方法的操作会更顺手一点。便如 Dan Abramov 所述，Facebook 推出 React 的初衷是为了能够在他们数以百计的跨平台子产品持续的迭代中保证组件的一致性与可复用性。

## 声明式组件

笔者在[现代 Web 开发导论/数据流驱动的界面]()一节中，对于广义的数据流驱动的界面有很多的理解，其一是界面层的从以 DOM 操作为核心到逻辑分离，其二是数据交互层的前后端分离。在 jQuery 时代，我们往往将 DOM 操作与逻辑操作混杂在一起，再加上模块机制的缺乏使得代码的可读性、可测试性与可维护性极低；随着项目复杂度的增加、开发人员的增加与时间的推移，项目的维护成本会以几何级数增长。随着 ES6 Modules 的广泛应用，我们在前端开发中更易于去实践 SRP 单一职责原则，也更方便地去编写单元测试、集成测试等来保证代码质量。而像 React、Vue 这样现代的视图层库为我们提供了声明式组件，托管了从数据变化到 DOM 操作之间的映射，使得开发者能够专注于业务逻辑本身。并且 Redux, MobX 这样独立的状态管理库，又可以将产品中的视图层与逻辑层剥离，保证了逻辑代码的易于测试性与跨端迁移性，促进了前端的工程化步伐。

而随着从传统的前后端未分离的巨石型 Web 应用，到 SPA 这样的富客户端前后端分离应用，前后端各成体系，能够应用不同的技术选型与项目架构。原本由服务端负责的数据渲染工作交由前端进行，并且规定前端与服务端之间只能通过标准化协议进行通信，给与了双方更好地灵活性与适应性。前后端分离也促成了组织架构上的分离，由早期的服务端开发人员顺手去写个界面转变为完整的前端团队构建工程化的前端架构。近两年来随着无线技术的发展和各种智能设备的兴起，互联网应用演进到以 API 驱动的无线优先(Mobile First)和面向全渠道体验(omni-channel experience oriented)的时代，BFF 这样前端优先的 API 设计模式与 GraphQL 这样的查询语言也得到了大量的关注与应用。

声明式编程的核心理念在于描述做什么，通过声明式的方式我们能够以链式方法调用的形式对于输入的数据流进行一系列的变换处理。React 中，我们的任何组件都可以以声明式的语法表述，譬如我们要写包含多个选项的选择控件时：

```jsx
<select value={this.state.value} onChange={this.handleChange}>
  {somearray.map(element => (
    <option value={element.value}>{element.text}</option>
  ))}
</select>
```

React 广泛实践了[函数式编程]()的思想，将状态到界面抽象为了如下的映射函数：$UI=f(state)$。在 React 中 $f$ 可以看做是那个 render 函数，可以将 state 渲染成 Virtual DOM，Virtual DOM 再被 React 渲染成真正的 DOM。

## Virtual DOM

在组件中，我们不需要关心 DOM 是如何变更的，只需要在我们的业务逻辑中完成状态转变，React 会自动将这个变更显示在 UI 中。在浏览器渲染网页的过程中，加载到 HTML 文档后，会将文档解析并构建 DOM 树，然后将其与解析 CSS 生成的 CSSOM 树一起结合产生爱的结晶——RenderObject 树，然后将 RenderObject 树渲染成页面(当然中间可能会有一些优化，比如 RenderLayer 树)。这些过程都存在与渲染引擎之中，渲染引擎在浏览器中是于 JavaScript 引擎(JavaScriptCore 也好 V8 也好)分离开的，但为了方便 JS 操作 DOM 结构，渲染引擎会暴露一些接口供 JavaScript 调用。由于这两块相互分离，通信是需要付出代价的，因此 JavaScript 调用 DOM 提供的接口性能不咋地。各种性能优化的最佳实践也都在尽可能的减少 DOM 操作次数。而虚拟 DOM 干了什么？它直接用 JavaScript 实现了 DOM 树(大致上)。组件的 HTML 结构并不会直接生成 DOM，而是映射生成虚拟的 JavaScript DOM 结构，React 又通过在这个虚拟 DOM 上实现了一个  diff  算法找出最小变更，再把这些变更写入实际的 DOM 中。这个虚拟 DOM 以 JS 结构的形式存在，计算性能会比较好，而且由于减少了实际 DOM 操作次数，性能会有较大提升。React 渲染出来的 HTML 标记都包含了`data-reactid`属性，这有助于 React 中追踪 DOM 节点。很多人第一次学习 React 的时候都会觉得 JSX 语法看上去非常怪异，这种背离传统的 HTML 模板开发方式真的靠谱吗？(在 2.0 版本中 Vue 也引入了 JSX 语法支持)。我们并不能单纯地将 JSX 与传统的 HTML 模板相提并论，JSX 本质上是对于`React.createElement`函数的抽象，而该函数主要的作用是将朴素的 JavaScript 中的对象映射为某个 DOM 表示。其大概思想图示如下：

![](https://coding.net/u/hoteam/p/Cache/git/raw/master/2017/1/1/QQ20170104-01111.png)

在现代浏览器中，对于 JavaScript 的计算速度远快于对 DOM 进行操作，特别是在涉及到重绘与重渲染的情况下。并且以 JavaScript 对象代替与平台强相关的 DOM，也保证了多平台的支持，譬如在 ReactNative 的协助下我们很方便地可以将一套代码运行于 iOS、Android 等多平台。总结而言，JSX 本质上还是 JavaScript，因此我们在保留了 JavaScript 函数本身在组合、语法检查、调试方面优势的同时又能得到类似于 HTML 这样声明式用法的便利与较好的可读性。

## 基于组件的架构

React 的组件系统是其精华所在，其基于组件的架构不仅

[软件工程导论]()中介绍过，模块更多是为了

When Facebook released React.js in 2013 it redefined the way in which Front End Developers could build user interfaces. React.js, a JavaScript library, introduced a concept called Component-Based-Architecture, a method for encapsulating individual pieces of a larger user interface (aka components) into self-sustaining, independent micro-systems.

Essentially, if you’re using a client-side MVC framework like Ember.js, and to a lesser extent, Angular, you have templates that present the UI, routes that determine which templates to render, and services that define helper functions. Even if a template has routes and associated methods, all of these exist at different levels of an application’s architecture.

In the case of CBA, responsibility is split on a component-by-component basis. This means that the design, logic, and helper methods exist all within the same level of the architecture (generally the view). As aforementioned, everything that pertains to a particular component is defined within that component’s class.

Component-Based Architecture

# Component | 组件基础

## 组件定义

```js
constructor(props) {
super();
console.log(this.props); // undefined
console.log(props); // defined
}

constructor(props) {
super(props);
console.log(this.props); // props will get logged.
}
```

PropTypes.array, PropTypes.bool, PropTypes.func, PropTypes.number, PropTypes.object, PropTypes.string, PropTypes.symbol, 对于 React 可渲染的类型还包括 PropTypes.node 与 PropTypes.element

```js
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // 指定类实例
  optionalMessage: PropTypes.instanceOf(Message), // 枚举类型

  optionalEnum: PropTypes.oneOf(['News', 'Photos']), // 可能为多种类型

  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]), // 包含指定类型的数组

  optionalArrayOf: PropTypes.arrayOf(PropTypes.number), // 包含指定值类型的对象

  optionalObjectOf: PropTypes.objectOf(PropTypes.number), // 某个具体形状的对象

  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  })

  // ...
};
```

React 16 中为我们提供了 Portals，方便地将元素渲染到非当前组件树层级的节点：

```js
render() {
  // React 并不会创建新的 div，而是将其渲染到指定的 DOM 节点中
  return ReactDOM.createPortal(
    this.props.children,
    domNode
  );
}
```

### JSX

目前组件支持返回数组元素，我们也可以使用 React.Fragment 来返回多个子元素而不添加额外的 DOM 元素：

```js
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
```

## 组件与 DOM

```js
class VideoPlayer extends React.Component {
  constructor() {
    super();
    this.state = {
      isPlaying: false
    };
    this.handleVideoClick = this.handleVideoClick.bind(this);
  }

  handleVideoClick() {
    if (this.state.isPlaying) {
      this.video.pause();
    } else {
      this.video.play();
    }
    this.setState({ isPlaying: !this.state.isPlaying });
  }

  render() {
    return (
      <div>
        <video
          ref={video => (this.video = video)}
          onClick={this.handleVideoClick}
        >
          <source src="some.url" type="video/ogg" />
        </video>
      </div>
    );
  }
}
```

## 事件监听与响应

为了避免过多地事件监听，React 引入了 SyntheticEvent 来集中式地监听事件与调用响应函数；我们自定义的事件处理器都会被传入 SyntheticEvent 对象，其支持 `stopPropagation()` 与 `preventDefault()`，并且保证了跨浏览器的一致性。出于性能的考虑，SyntheticEvent 会复用传入的 Event 对象，因此我们避免直接异步读取 Event 对象的值，而是应该使用闭包将需要的值保存下来：

```js
function onClick(event) {
  console.log(event); // => nullified object.
  console.log(event.type); // => "click"
  const eventType = event.type; // => "click"

  this.setState({ eventType: event.type });
}
```

对于 DOM 事件标准中定义的 Capturing phase 与 Bubbling phase，React 也提供了 onClick 与 onClickCapture：

```js
<div onClickCapture={this.handleClickViaCapturing}>
  <button onClick={this.handleClick}>
    Click me, and my parent's `onClickCapture` will fire *first*!
  </button>
</div>
```

## 生命周期

![dz-97vzw4aabczj](https://user-images.githubusercontent.com/5803001/38792131-18812574-417e-11e8-97e5-d523160fdd34.jpg)

```js
componentDidUpdate(prevProps, prevState, snapshot);
```

在 React 16.3 中移除了 componentWillReceiveProps 之后，我们可以在类中定义 getDerivedStateFromProps 来完成状态的自动推断：

```js
static getDerivedStateFromProps(nextProps, prevState){
    if (nextProps.currentRow === prevState.lastRow){
        return null;
    }
    return {
        lastRow: nextProps.currentRow,
        isScrollingDown: nextProps.currentRow > prevState.lastRow
    }
}
```

值得一提的是，Fiber 会自动开启 StrictMode，

## 组件样式

### 样式类

```js
import cx from 'classnames';
import styles from './capsule.css';

// 使用 classnames
let className = cx(styles.base, {
  [styles.clickable]: this.props.clickable,
  [styles.withIcon]: !!this.props.icon
});
return <div className={className} />;

// 使用朴素的数组操作
return (
  <div
    classNames={[styles.base, styles.clickable, styles.withIcon].join(' ')}
  />
);
```

### CSS-in-JS

# Component Dataflow | 组件数据流

## Props

## State

```js
// 先获取元素下标，然后执行删除
const array = [...this.state.people]; // make a separate copy of the array
const index = array.indexOf(e.target.value);
array.splice(index, 1);

// 使用 filter 进行过滤删除
```

### 不可变对象

```js
removePeople(e) {
  var array = [...this.state.people]; // make a separate copy of the array
  var index = array.indexOf(e.target.value)
  array.splice(index, 1);
  this.setState({people: array});
},

removePeople(e) {
    this.setState({people: this.state.people.filter(function(person) {
        return person !== e.target.value
    })};
}
```

### 异步数据处理

```js
async componentDidMount() {
  try {
    const response = await fetch(`https://api.coinmarketcap.com/v1/ticker/?limit=10`);
    if (!response.ok) {
      throw Error(response.statusText);
    }
    const json = await response.json();
    this.setState({ data: json });
  } catch (error) {
    console.log(error);
  }
}
```

### 受控组件

React 中的组件又可以分为受控组件与非受控组件，所谓的非受控组件即 render 函数直接返回 `input` 这样的标签，状态直接存放在 DOM 而非组件类中。

```js
    render() {
        return (
            <input />
        )
    }
```

我们可以通过接管标签的改变事件与值，来将非受控组件转化为受控组件：

```js
  handleChange: function (propertyName, event) {
    const contact = this.state.contact;
    contact[propertyName] = event.target.value;
    this.setState({ contact: contact });
  },
  render: function () {
    return (
      <div>
        <input type="text" onChange={this.handleChange.bind(this, 'firstName')} value={this.state.contact.firstName}/>
        <input type="text" onChange={this.handleChange.bind(this, 'lastName')} value={this.state.contact.lastName}/>
        <input type="text" onChange={this.handleChange.bind(this, 'phone')} value={this.state.contact.lastName}/>
      </div>
    );
  }
```

## Context

# React Router

```js
const PrivateRoute = ({ component: Component, ...rest }) => (
  <Route {...rest} render={(props) => (
    fakeAuth.isAuthenticated === true
      ? <Component {...props} />
      : <Redirect to={{
          pathname: '/login',
          state: { from: props.location }
        }} />
  )} />
)

<PrivateRoute path='/protected' component={Protected} />
```

# 架构模式

## Functional React | 函数式 React

## HoC | 高阶组件

## renderProps

# 工程实践

16.3 版本中，React 为我们提供了 StrictMode 组件，来强制保证代码的最佳实践。

```jsx
<StrictMode>
  <App />
</StrictMode>
```

## 异常处理

# Ecosystem | React 生态圈

在跨平台开发领域，React Native 是当之无愧的跨平台开发首选。而 [Electron]() 与 [Proton Native]() 也都能为我们提供

Same syntax as React Native
Works with existing React libraries such as Redux
Cross platform
Native components. No more Electron
Compatible with all normal Node.js packages

Proton Native does the same to desktop that React Native did to mobile. Build cross-platform apps for the desktop, all while never leaving the React eco-system. Popular React packages such as Redux still work.

# TypeScript

React 的 TypeScript 类型声明[types/react](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react)

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
