[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# CSS CheatSheet | CSS 语法速览与实践清单

# Animation | 动画

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

# CSS Animation

CSS3 动画相关的几个属性是：transition, transform, animation；分别理解为过渡，变换，动画。transition 指过渡，就是从 a 点都 b 点，是有时间的，是连续的，一般针对常规 CSS 属性；transform 指变换，包含几个固定的属性：旋转、缩放、偏移等等，但是，效果就是很干涩机械的旋转移动，如果配合 transition 属性，变换就会很平滑。

## transition

当你的 left 属性发生变化的时候，执行动画效果（仅仅基于 left 的属性变化，其他的属性不会加入到动画变化里面去）。

```css
.position {
  left: 100px;
  top: 100px;
}

.animate {
  -webkit-transition: left 0.5s ease-out;
  left: 500px;
  top: 500px;
}
```

首先你的元素的 css 为 position。当你将其 cssList 增加 animate 或者替换 position 为 animate 的时候，元素的属性变化，触发 webkit-transition，以指定属性变化前的值为起始值，变化后的属性为结束值，执行动画效果。这是一个简单的两点变化过程，大大简化了 animation 属性的复杂程度。

```css
transition-property : _ //指定过渡的性质，比如 transition-property:backgrond 就是指 backgound 参与这个过渡
transition-duration: _//指定这个过渡的持续时间
transition-delay: _ //延迟过渡时间
transition-timing-function: _//指定过渡类型，有 ease | linear | ease-in | ease-out | ease-in-out | cubic-bezier
```

ease-in 表示先慢后快；ease-out 表示先快后慢；ease-in-out 表示先慢后快再慢，这里的 cubic-bezier 就是贝塞尔曲线，[Easing functions](https://easings.net/en#) 提供了更多的动画函数支持。

## transform

transform 指变换，即拉伸、压缩、旋转、偏移等等。

```css
.trans_skew {
  transform: skew(35deg);
}
.trans_scale {
  transform: scale(1, 0.5);
}
.trans_rotate {
  transform: rotate(45deg);
}
.trans_translate {
  transform: translate(10px, 20px);
}
```

transform 属性要可以加上 transition 的过渡特性:

```css
.trans_effect {
  transition: all 2s ease-in-out;
}
.trans_effect:hover {
  transform: rotate(720deg) scale(2, 2);
}
```

transform 属性是静态属性，一旦写到 style 里面，将会直接显示作用，无任何变化过程。transform 的主要用途是用来做元素的特殊变形，对于做设计的人来说并不是很陌生，简单的来说就是 css 的图形变形工具。关于图形变形的基础条件当中的原点设定，在 css 里面使用的是 transform-origin 来定义的。这个定义的原点应该是该 css 作用的元素的左上角为 0,0 来计算的（有待研究）。其他的属性则根据这个属性来计算进行计算。

- translate3d(x,y,z) 是用来控制元素的位置在页面上的三轴的位置的；

- rotate(deg)是用来控制元素旋转角度的；

- skew[x,y](deg) 这个属性是用来制作倾斜度的，做过设计的人可能会知道，这个是用来在 2d 里面创建 3d 透视图的时候必须的属性；

- scale3d(x,y,z) 用来放大缩小效果，属性是比值；

- matrix3d，css 矩阵。通过这个矩阵属性，涵盖了上面所有的属性值，但是个人觉得可读性极差（全都是数字和单位，背起来有点模糊），目前没有理由推荐使用。

## animation

animation 是典型的帧动画:

```css
@keyframes 'wobble' {
  0% {
    left: 100px;
  }
  30% {
    left: 300px;
  }
  100% {
    left: 500px;
  }
}

.animate {
  left: 100px;
  -webkit-animation: wobble 0.5s ease-out;
  -webkit-animation-fill-mode: backwards;
}
```

上面这个代码展示了一个 `keyframes 'wobble'`，其中 0％ 代表在变化中不同时间点的属性值，你可以较精确的控制动画变化中任何一个时间点的属性效果。而 animation 则根据这个 keyframes 提供的属性变化方式去计算元素动画当中的属性。与 transition 不同的是，keyframes 提供更多的控制，尤其是时间轴的控制。

另外在 animation 属性里面还有一个最重要的就是：animation-fill-mode，这个属性标示是以(from/0%)指定的样式还是以(to/100%)指定的样式为动画完成之后的样式。这个很方便我们控制动画的结尾样式，保证动画的整体连贯。
