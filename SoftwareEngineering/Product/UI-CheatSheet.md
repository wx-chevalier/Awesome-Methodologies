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

# Todos 

- https://zhuanlan.zhihu.com/p/50413398
