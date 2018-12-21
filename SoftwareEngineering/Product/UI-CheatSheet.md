# 交互设计指南

# 设计基础

## 色彩

![image](https://user-images.githubusercontent.com/5803001/48763697-c535b800-ece8-11e8-9997-7d0fd5901303.png)

![image](https://user-images.githubusercontent.com/5803001/48763759-e5fe0d80-ece8-11e8-98c0-5771e550bf29.png)

## 字体

```css
font-family: "Chinese Quote", -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", "Helvetica Neue", Helvetica, Arial, sans-serif,"Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
```

| **平台**  | **中文**        | **西文**      |
| --------- | --------------- | ------------- |
| MacOS/iOS | PingFang SC     | San Francisco |
| Android   | Noto Sans CJK   | Roboto        |
| Windows   | Microsoft YaHei | Segoe UI      |

| 类型 | 字号 | 行高 | 字重         | 描述    | 适用范围                                 |
| ---- | ---- | ---- | ------------ | ------- | ---------------------------------------- |
| T1   | 12   | 20   | Regular 400  | Caption | 辅助文本                                 |
| T2   | 14   | 22   | Regular 400  | Body    | 正文                                     |
| T3   | 16   | 24   | Regular 400  | Title   | 视图信息、数据图表卡片标题、左侧导航菜单 |
| T4   | 20   | 28   | Regular 400  | H3      | 页面标题                                 |
| T5   | 30   | 38   | Semibold 600 | H2      | 视图信息、数据图表重点信息文本           |
| T6   | 46   | 54   | Semibold 600 | H1      | 运营标题                                 |

![image](https://user-images.githubusercontent.com/5803001/48979121-36160f00-f0f1-11e8-81a1-fd7c299cc39b.png)

## 布局

### 图层

图层本身承载了内容和组件，它是一种用来为内容建立组织关系和联系的视觉元素。我们通过区分图层位于 Z 轴的深度来赋予它们层级关系。

| **图层** | **高度** | **投影**                        | **适用范围**       |
| -------- | -------- | ------------------------------- | ------------------ |
| 信息层   | 4        | 0 2px 4px 0 rgba(0,0,0,0.10);   | 信息卡片           |
| 覆盖层   | 8        | 0 4px 8px 0 rgba(0,0,0,0.10);   | 顶部栏、下拉选项   |
| 导航层   | 12       | 0 6px 12px 0 rgba(0,0,0,0.10);  | 侧边导航           |
| 模态层   | 20       | 0 10px 20px 0 rgba(0,0,0,0.10); | 模态弹窗、消息通知 |

![image](https://user-images.githubusercontent.com/5803001/48979124-49c17580-f0f1-11e8-97d1-78100fc21827.png)

## 动画

- 基本 style 可动画参数

| 参数名称        | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| width           | `{ width: 100 }` 元素当前宽度到 100px                        |
| maxWidth        | `{ maxWidth: 100 }` 元素当前最大宽度到 100px                 |
| minWidth        | `{ minWidth: 100 }` 元素当前最小宽度到 100px                 |
| height          | `{ height: 100 }` 元素当前高度到 100px                       |
| maxHeight       | `{ maxHeight: 100 }` 元素当前最大高度到 100px                |
| minHeight       | `{ minHeight: 100 }` 元素当前最小高度到 100px                |
| lineHeight      | `{ lineHeight: 100 }` 区块行高到 100px                       |
| opacity         | `{ opacity: 0 }` 元素当前透明度到 0                          |
| top             | `{ top: 100 }` 元素当前顶部距离到 100px, 需配合 `position: relative | absolute` |
| right           | `{ right: 100 }` 元素当前右部距离到 100px, 需配合 `position: relative | absolute` |
| bottom          | `{ bottom: 100 }` 元素当前下部距离到 100px, 需配合 `position: relative | absolute` |
| left            | `{ left: 100 }` 元素当前左部距离到 100px, 需配合 `position: relative | absolute` |
| marginTop       | `{ marginTop: 100 }` 元素当前顶部外边距离到 100px            |
| marginRight     | `{ marginRight: 100 }` 元素当前右部外边距离到 100px          |
| marginBottom    | `{ marginBottom: 100 }` 元素当前下部外边距离到 100px         |
| marginLeft      | `{ marginLeft: 100 }` 元素当前左部外边距离到 100px           |
| paddingTop      | `{ paddingTop: 100 }` 元素当前顶部内边距离到 100px           |
| paddingRight    | `{ paddingRight: 100 }` 元素当前右部内边距离到 100px         |
| paddingBottom   | `{ paddingBottom: 100 }` 元素当前下部内边距离到 100px        |
| paddingLeft     | `{ paddingLeft: 100 }` 元素当前左部内边距离到 100px          |
| color           | `{ color: '#FFFFFF' }` 元素当前文字颜色到白色                |
| backgroundColor | `{ backgroundColor: '#FFFFFF' }` 元素当前背景颜色到白色      |
| borderWidth     | `{ borderWidth: 2 }` 元素当前边框宽度到 2px，同样可用 `borderTopWidth` `borderRightWidth``borderBottomWidth` `borderLeftWidth` |
| borderRadius    | `{ borderRadius: 5 }` 元素当前圆角到 5px, 同上, 同样可用 `上 左 下 右` |
| borderColor     | `{ borderColor: '#FFFFFF' }` 元素当前边框颜色到白色          |
| boxShadow       | `{ boxShadow: '0 0 10px #000' }` 元素当前阴影模糊到 10px     |
| textShadow      | `{ textShadow: '0 0 10px #000' }` 元素当前文字内容阴影模糊到 10px |

- transform 参数

| 参数名称        | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| translateX / x  | `{ translateX: 10 } or { x: 10 } => transform: translateX(10px)`, x 方向的位置移动 10px |
| translateY / y  | `{ translateY: 10 } or { y: 10 } => transform: translateY(10px)`, y 方向的位置移动 10px |
| translateZ / z  | `{ translateZ: 10 } or { z: 10 } => transform: translateZ(10px)`, z 方向的位置移动 10px |
| rotate          | `{ rotate: 10 } => transform: rotate(10deg)` 元素以 transformOrigin 的中心点旋转 10deg |
| rotateX         | `{ rotateX: 10 } => transform: rotateX(10deg)` 元素以 transformOrigin 的中心点向 X 旋转 10deg |
| rotateY         | `{ rotateY: 10 } => transform: rotateY(10deg)` 元素以 transformOrigin 的中心点向 Y 旋转 10deg |
| scale           | `{ scale: 0 } => transform: scale(0)` 元素以 transformOrigin 的中心点缩放到 0, 不改变元素的宽高 |
| scaleX          | `{ scaleX: 0 } => transform: scaleX(0)` 元素以 transformOrigin 的中心点 X 缩放到 0, 不改变元素的宽高 |
| scaleY          | `{ scaleY: 0 } => transform: scaleY(0)` 元素以 transformOrigin 的中心点 Y 缩放到 0, 不改变元素的宽高 |
| transformOrigin | `{ transformOrigin: '50px 50px'}` 元素当前中心点到 x: 50px y: 50px; |

- filter 参数

| 参数名称   | 说明                                                   |
| ---------- | ------------------------------------------------------ |
| grayScale  | `{ grayScale: 1 }` 元素 filter 灰度到 100%;            |
| sepia      | `{ sepia: 1 }` 元素 filter 颜色到 100%;                |
| hueRotate  | `{ hueRotate: '90deg' }` 元素 filter 色相盘旋转 90 度; |
| invert     | `{ invert: 1 }` 元素 filter 色值反相到 100%            |
| brightness | `{ brightness: 2 }` 元素 filter 亮度到 200%            |
| contrast   | `{ contrast: 2 }` 对比度到 200%                        |
| saturate   | `{ saturate: 2 }` 饱和度到 200%                        |
| blur       | `{ blur: '20px' }` 模糊到 20px                         |



# Todos 

- https://zhuanlan.zhihu.com/p/50413398
