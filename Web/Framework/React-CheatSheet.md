[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# React 语法速览与实践清单

# Principles: 设计理念

# Component: 组件基础

## 组件定义

```js
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // You can declare that a prop is a specific JS type. By default, these
  // are all optional.
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // Anything that can be rendered: numbers, strings, elements or an array
  // (or fragment) containing these types.
  optionalNode: PropTypes.node,

  // A React element.
  optionalElement: PropTypes.element,

  // You can also declare that a prop is an instance of a class. This uses
  // JS's instanceof operator.
  optionalMessage: PropTypes.instanceOf(Message),

  // You can ensure that your prop is limited to specific values by treating
  // it as an enum.
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // An object that could be one of many types
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // An array of a certain type
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // An object with property values of a certain type
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // An object taking on a particular shape
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),

  // You can chain any of the above with `isRequired` to make sure a warning
  // is shown if the prop isn't provided.
  requiredFunc: PropTypes.func.isRequired,

  // A value of any data type
  requiredAny: PropTypes.any.isRequired

  // ...
};
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

# Component Dataflow: 组件数据流

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
