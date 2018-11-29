#### Mo.js

Mo.js 是一个"简洁、高效"图形动画库，拥有流畅的动画和惊人的用户体验，在任何设备上，屏幕密度独立的效果都很好，你可以绘制内置的形状或者自定义形状，随便，只要你喜欢，你还可以绘制多个动画，再让它们串联在一起，逼话不多说详细的请浏览 [Mo.js 官方网站](http://mojs.io/)

![模块](https://upload-images.jianshu.io/upload_images/5531211-1e94335e604969f0.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 基础

**首先你需要引入优雅的 Mo.js**

```
<script src="http://cdn.jsdelivr.net/mojs/latest/mo.min.js"></script>
```

##### 使用 Shape 模块绘制图形

该模块提供的 Api 使用起来非常方便，也很好理解，原理便是通过 Javascript 生成 SVG 图形，需要注意的是，画出来的 SVG 默认给一个 DIV 包裹着，初始位置是绝对定位全屏居中

**1. 画一个矩形**
![img](https://upload-images.jianshu.io/upload_images/5531211-cd43ab4c12c388f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```js
let rect = new mojs.Shape({
  shape: 'rect', // 定义形状为矩形
  isShowStart: true // 定义初始化之后就显示
});
```

**2. 画一个圆形**
![img](https://upload-images.jianshu.io/upload_images/5531211-f85b2cd72fc15014.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
let circle = new mojs.Shape({
  shape: 'circle', // 定义形状为圆形
  fill: 'cyan', // 填充颜色
  isShowStart: true
})
```

**3. 空心 + 边框**
![img](https://upload-images.jianshu.io/upload_images/5531211-39a771c3040e8183.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
let circle = new mojs.Shape({
  shape: 'circle',
  stroke: 'cyan', // 画笔颜色
  fill: 'none', // 不填充
  isShowStart: true
})
```

**4. 多图形**
![img](https://upload-images.jianshu.io/upload_images/5531211-807e9cb0150c955e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实例化两个 Shape 就行了

```
let circle1 = new mojs.Shape({
  shape: 'circle',
  stroke: 'cyan',
  fill: 'none',
  isShowStart: true
})

let circle2 = new mojs.Shape({
  shape: 'circle',
  radius: 30, // 半径
  isShowStart: true
})
```

**5. 控制位置**
![img](https://upload-images.jianshu.io/upload_images/5531211-89fa98bca0d9516b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

修改圆形二的 x，y 值（相对当前位置进行移动）

```
// 在原先的基础上增加
let circle2 = new mojs.Shape({
  x: 100,
  y: 100
})
```

##### 使用 Shape 模块绘制动画

**1. 过渡属性设置以及播放动画**
![img](https://upload-images.jianshu.io/upload_images/5531211-aed11eeb206eb667.gif?imageMogr2/auto-orient/strip)

旋转角度从-180 过渡至 0

```
let rect = new mojs.Shape({
  shape: 'rect',
  fill: 'none',
  stroke: 'cyan',
  radius: 10,
  strokeWidth: 20, // 画笔宽度
  angle: {
    [-180]: 0 // 使用对象的形式设置，key为开始值，val为结束值（任何属性都可以设置过渡）
  },
// duration: 400 // 默认为300ms
}).play()
```

Play 方法为播放动画，上述是链式调用，相当于

```
let rect = new mojs.Shape();
rect.play();
```

去掉 isShowStart，因为这里的 Play 方法也是初始化就调用，用不着初始化就显示该图形（除非你的动画是延迟执行的）

**2. 其他过渡参数**
![img](https://upload-images.jianshu.io/upload_images/5531211-4db82ab65cc783e4.gif?imageMogr2/auto-orient/strip)

动画轮流反向播放，即 x 坐标从-100 至 100、100 至-100、-100 至 100 共 3 次

```
let rect = new mojs.Shape({
  x: {
    [-100]: 100
  },
  delay: 500, // 动画延迟500ms执行
  repeat: 2, // 动画重复的次数
  isYoyo: true, // 是否轮流反向播放（类似css3中的animation-direction）
  isShowEnd: false // 动画结束后图形是否显示，默认为true
}).play()
```

这里没有设置 isShowStart，动画又延迟 500ms 执行，所以一开始图形是不显示的

##### Then 方法

Then 方法其实就是字面上的意思“然后”，一个动画执行完之后的回调函数

![img](https://upload-images.jianshu.io/upload_images/5531211-03531e2844a782f4.gif?imageMogr2/auto-orient/strip)

上面动画分两步，旋转、缩小

```
new mojs.Shape({
  shape: 'rect',
  fill: 'none',
  stroke: 'cyan',
  radius: 10,
  strokeWidth: 20,
  angle: {
    [-180]: 0
  },
  duration: 600
}).then({
  strokeWidth: {
    50: 0 // 画笔大小由50过渡到0，所以图形消失了
  },
  stroke: {
    'magenta': 'yellow' // 画笔颜色由magenta过渡到yellow
  }
}).play()
```

##### 回调函数

这里只列出 3 个常用的，回调函数内用 this 访问当前实例化对象

```
new mojs.Shape({
  onStart (isForward, isYoyo) {
    // 动画开始执行
  },
  onComplete (isForward, isYoyo) {
    // 动画执行完毕

    // this举例，如动画执行完成需要移除DOM
    this.el.remove()
  },
  onProgress (p, isForward, isYoyo) {
    // 动画执行时
  }
})
```

##### 其他方法

这里只列出常用的 4 个常用的，都是实例化对象的方法

```
new mojs.Shape()
.play() // 执行动画
.pause() // 暂停动画
.stop() // 结束动画
.replay() // 重播动画，相当于stop + play
```

##### Timeline

把多个图形动画一起执行

```
let rect = mojs.Shape();
let circle = mojs.Shape();

// 相当于rect.play() && circle.play()
new mojs.Timeline().add(rect, circle).play();
```

![img](https://upload-images.jianshu.io/upload_images/5531211-7e0dafcad285c50e.gif?imageMogr2/auto-orient/strip)

每个图形的动画都是一样，只是颜色、形状、延迟播放的时间不一样

```
// 默认参数
const OPTIONS = {
  fill: 'none',
  radius: 50,
  strokeWidth: {
    50: 0
  },
  scale: {
    0: 1
  },
  angle: {
    [-100]: 0
  }
}

// 延迟时间跟动画
let delay = 0,
  delayStep = 150;

// 矩形
let rect = new mojs.Shape({
  ...OPTIONS,
  shape: 'rect',
  stroke: 'cyan'
});

// 圆形
let circle = new mojs.Shape({
  ...OPTIONS,
  shape: 'circle',
  stroke: 'yellow',
  radius: 25,
  strokeWidth: {
    25: 0
  },
  x: -35,
  y: -35,
  delay: delay += delayStep // 延迟执行的时间
});

// 三角形
let triangle= new mojs.Shape({
  ...OPTIONS,
  shape: 'polygon',
  stroke: 'magenta',
  radius: 25,
  strokeWidth: {
    25: 0
  },
  x: -35,
  y: -35,
 delay: delay += delayStep
});

// 五边形
let polygon= new mojs.Shape({
  ...OPTIONS,
  shape: 'polygon',
  points: 5,
  stroke: '#00F87F',
  x: -20,
  y: -35,
 delay: delay += delayStep
});

// 其他图形省略...

// 添加至timeline一起执行
new mojs.Timeline().add(rect, circle, triangle, polygon).play()
```

##### Tune

播放前重置参数

```
new mojs.Shape()
.tune({
  // 新的参数
})
.play()
```

先画两个圆形

```
const OPTIONS = {
  shape: 'circle',
  fill: 'none',
  radius: 25,
  stroke: 'cyan',
  scale: {
    0: 1
  },
  easing: 'cubic.out'
}

let circle1 = new mojs.Shape({
  ...OPTIONS,
  strokeWidth: {
    50: 0
  }
}).play()

let circle2 = new mojs.Shape({
  ...OPTIONS,
  radius: 10,
  stroke: 'magenta',
  strokeWidth: {
  15: 0
  },
  delay: 200
}).play()
```

把 play 去掉，默认参数加上 top:0, left: 0（原本是页面的中心）,鼠标点击的时候，动态设置 x，y 的值

![img](https://upload-images.jianshu.io/upload_images/5531211-e0ba784730a29aea.gif?imageMogr2/auto-orient/strip)

```
document.addEventListener('click', e => {
  // 鼠标点击时的x，y坐标
  let x = e.pageX,
    y = e.pageY;

  // 播放圆形1动画
  circle1.tune({
    x,
    y
  }).replay()

  // 播放圆形2动画
  circle2.tune({
    x,
    y
  }).replay()
})
```

但是这样子写，一直都只存在一个 circle1 跟 circle2，第一次点击还没执行完动画，我就点击第二次，会重新播放动画

![img](https://upload-images.jianshu.io/upload_images/5531211-0d3b2b94f298a000.gif?imageMogr2/auto-orient/strip)

我们可以选择在点击的时候才生成匿名的圆形对象，并且动画执行完毕之后移除 DOM，这样子我们就不需要用到 tune 了

```
document.addEventListener('click', e => {
  let x = e.pageX,
    y = e.pageY;

  new mojs.Shape({
    ...OPTIONS,
    strokeWidth: {
      50: 0
    },
    x,
    y,
    onComplete() {
      this.el.remove()
    }
  }).play()

  new mojs.Shape({
    ...OPTIONS,
    radius: 10,
    stroke: 'magenta',
    strokeWidth: {
      15: 0
    },
    delay: 200,
    x,
    y,
    onComplete() {
      this.el.remove()
    }
  }).play()
})
```

效果如下，也可以不移除 DOM，不过你点 100 次，页面就会生成 100 个 DOM

![img](https://upload-images.jianshu.io/upload_images/5531211-f9e9bfa16956668b.gif?imageMogr2/auto-orient/strip)

监听 click 改成监听 mousemove

![img](https://upload-images.jianshu.io/upload_images/5531211-ba588de2daff5d3c.gif?imageMogr2/auto-orient/strip)

##### 如何给图形监听事件

```
let btn = new mojs.Shape();
btn.el.addEventListener(); // el表示svg的上层包裹dom
```

作者：白色风车 kai

链接：http://www.imooc.com/article/256064

来源：慕课网

本文首次发布于慕课网 ，转载请注明出处，谢谢合作
