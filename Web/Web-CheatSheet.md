[![返回目录](https://parg.co/UCb)](https://github.com/wx-chevalier/Awesome-CheatSheets)

# Web 应用运行机制盘点

[![web](https://user-images.githubusercontent.com/5803001/38910164-a8e97c1a-42fa-11e8-8500-a737833e80cc.png)](https://www.processon.com/mindmap/59a26552e4b0afafe7a7606c)

# Rendering: 渲染

## Layout: 布局

After the DOM (Document Object Model) is created and the styles are recalculated, the browser takes a moment to figure out how much space each visible HTML node is about to take and where it is going to be placed. This phase is called “Layout”, and at this point elements are only represented as vector boxes.

## Paint: 绘制

Once that part is done, the browser takes these vector boxes and rasterizes them (exchanges vectors to pixels) in a “Paint” step. The rasterized elements are put on “layers” (by default only one layer, unless there is a reason to move them away — more about that later).

At this point, for example in Chrome, the new layers are created while:

- Using 3D or perspective transforms properties
- Using animated 2D transforms or opacity properties
- An element is on top or a child of a compositing layer
- Using accelerated CSS filters
- Embedding <video>, <canvas>, plugins like Silverlight or Flash (in special cases)

## Compositing: 组合

The layers are placed together and finally shown on the screen.
All of this work happens when we want to show just one frame to the user. But if any change is introduced to the interface (e.g. scrolling, triggering an animation), the browser needs to create a series of frames to represent that change.
