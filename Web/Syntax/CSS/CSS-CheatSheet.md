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

// 获取变量值
element.style.getPropertyValue('--my-color');
// => 'rebeccapurple'

// 移除变量值
element.style.removeProperty('--my-color');
```
