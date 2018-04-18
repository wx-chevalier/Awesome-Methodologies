[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# React 语法速览与实践清单

# Principles: 设计理念

# Component: 组件基础

## 组件定义

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
