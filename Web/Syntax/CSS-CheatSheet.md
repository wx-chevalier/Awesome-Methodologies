[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# CSS CheatSheet | CSS 语法速览与实践清单

# Syntax

## Selector

```css
:root {
  --red: #ff6f69;
}

#title {
  color: var(--red);
}

.quote {
  color: var(--red);
}
```

```js
// 设置变量值
element.style.setProperty('--my-color', 'rebeccapurple');

// 获取变量值
element.style.getPropertyValue('--my-color');
// => 'rebeccapurple'

// 移除变量值
element.style.removeProperty('--my-color');
```

# Preprocessors

## SCSS

## Less

```less
@sectionHeight: calc(~'100vh - 130px');
```

# CSS Modularization | CSS 模块化

## CSS Modules

```js
import styles from './style.css';
// import { className } from "./style.css";

element.innerHTML = '<div class="' + styles.className + '">';
```

```less
:global {
  .global-class-name {
    color: green;
  }
}
```

```css
.className {
  color: green;
  background: red;
}

// 合并其他样式类
.otherClassName {
  composes: className;
  color: yellow;
}

// 合并来自其他文件的样式
.otherClassName {
  composes: className from './style.css';
}
```
