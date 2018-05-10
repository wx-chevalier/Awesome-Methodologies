[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# 2018 React 语法纵览与实践清单

# Principles | 设计理念

# Component | 组件基础

## 组件定义

  constructor(props) { 
    super();
    console.log(this.props); // undefined
    console.log(props); // defined
  }

  constructor(props) { 
    super(props);
    console.log(this.props); // props will get logged.
  }

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

## JSX

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

class VideoPlayer extends React.Component {
 constructor() {
    super();
    this.state = {
      isPlaying: false,
    }
    this.handleVideoClick = this.handleVideoClick.bind(this);
  }
  
  handleVideoClick() {
    if (this.state.isPlaying) {
      this.video.pause();
    }
    else {
      this.video.play();
    }
    this.setState({ isPlaying: !this.state.isPlaying });
  }
  
  render() {
    return (
      <div>
        <video
          ref={video => this.video = video}
          onClick={this.handleVideoClick}
        >
         <source
            src="some.url"
            type="video/ogg"
         />
        </video>
      </div>
    )
  }
}


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

# 编程模式

## HoC: 高阶组件

## renderProps

# 工程实践

16.3 版本中，React 为我们提供了 StrictMode 组件，来强制保证代码的最佳实践。

```jsx
<StrictMode>
  <App />
</StrictMode>
```

## 异常处理

# TypeScript
