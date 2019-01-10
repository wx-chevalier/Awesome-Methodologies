[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Canvas & WebGL CheatSheet

SVG 只是一种矢量图形文件格式， 不仅现在的浏览器都支持，很多主流的系统也都支持。可以代替一些图片,多用于图标,以及图表上,优势在于拥有 HTML 的 event 事件,交互起来很方便。

Canvas 是 HTML5 新增的一个元素对象，名副其实就是一个画布，浏览器 js 配有相应的操作 api，可以不再依赖其他的 API 或组件而直接绘图，相当于 2D 的 API。一般用于绘制比较复杂的动画,做游戏之类的,由于 canvas 是 HTML5 带的,所以不支持低版本浏览器,特别是 IE,canvas 只是一个画布,绘制上去的东西,例如图片,都是转换成像素点绘制上去的,所以没有 event 事件,如果需要添加交互事件,需要自己手动计算绘制的对象所在坐标以及层级。

WebGL 是以 OpenGL ES 2.0 为基础的一套 浏览器 3D 图形 API （HTML5），在编程概念上与 OpenGL ES 2.0 几乎是完全通用的，同样采用可编程渲染管线，也就是每个顶点的处理受到一小段 Vertex Shader 代码的控制，每个像素的绘制过程也受到一小段 Fragment Shader 代码的控制。WebGL 主要是 3D 为主， 不过 2D 的绘图要求也可以变通来实现。WebGL 无论如何都需要一个显示对象来呈现，这个对象就是 Canvas，仅此而已，WebGL 不对 Canvas 有任何附加的操作 API， 那部分属于浏览器 js 支持的范畴。 可看作能在浏览器上运行的 OpenGL,WebGL 的 HTML 节点名称用的也是 canvas,但是他的渲染则和 canvas 不同,他可以支持硬件加速,支持 3D,可用于 3D 游戏的开发,目前很少有 3D 的 HTML5 游戏,现在你能看到很多酷炫的图形交互的 3D 图表,大多用 WebGL 来渲染的。WebGL 也继承 OpenGL ES 2.0 的兼容性支持能力， 在不同的设备上做有限的支持，需要运行时查询。

Three.js、Babylon.js、Blender4Web 等是几种知名的 WebGL 开发框架，对 WebGL 基础操作做了大量的封装，可以拿来就用，即使不了解 WebGL 规范的细节。

# Canvas

canvas（翻译为画布）是 HTML5 的一个标签，canvas 可以使用 JavaScript 在网页上绘制图像，例如下面的代码就使用 canvas 绘制一个简单的矩形。

```html
<canvas id="container" width="1280px" height="720px"></canvas>
```

```js
const canvas = document.getElementById('container');
const context = canvas.getContext('2d');
context.fillStyle = 'rgba(0, 0, 255, 1.0)';
context.fillRect(120, 10, 150, 150);
```

canvas 只支持一些简单的 2d 绘制，不支持 3d，更重要的是性能有限，WebGL 弥补了这两方便的不足。

# WebGL

```js
const canvas = document.getElementById('container');
const gl =
  canvas.getContext('webgl') || canvas.getContext('experimental-webgl');
gl.clearColor(0.0, 0.0, 0.0, 1.0); // 指定清空canvas的颜色
gl.clear(gl.COLOR_BUFFER_BIT); // 清空canvas
```

使用 WebGL 绘制，依赖于着色器（shader）；

- 顶点着色器（Vertex shader）: 绘制每个定点都会调用一次；
- 片段着色器（Fragment shader）: 每个片源（可以简单的理解为像素）都会调用一次；

```js
/**
 * 使用WebGL画点
 * xu.lidong@qq.com
 * */

// 顶点着色器源码
var vertexShaderSrc = `
void main(){
    gl_Position = vec4(0.0, 0.0, 0.0, 1.0);// gl_Position 内置变量，表示点的位置，必须赋值
    gl_PointSize = 10.0;// gl_PointSize 内置变量，表示点的大小（单位像素），可以不赋值，默认为1.0，，绘制单个点时才生效
}`;

// 片段着色器源码
var fragmentShaderSrc = `
void main(){
    gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);// 内存变量，表示片元颜色RGBA
}`;

// 初始化使用的shader
function initShader(gl) {
  var vertexShader = gl.createShader(gl.VERTEX_SHADER); // 创建顶点着色器
  gl.shaderSource(vertexShader, vertexShaderSrc); // 绑定顶点着色器源码
  gl.compileShader(vertexShader); // 编译定点着色器

  var fragmentShader = gl.createShader(gl.FRAGMENT_SHADER); // 创建片段着色器
  gl.shaderSource(fragmentShader, fragmentShaderSrc); // 绑定片段着色器源码
  gl.compileShader(fragmentShader); // 编译片段着色器

  var shaderProgram = gl.createProgram(); // 创建着色器程序
  gl.attachShader(shaderProgram, vertexShader); // 指定顶点着色器
  gl.attachShader(shaderProgram, fragmentShader); // 指定片段着色色器
  gl.linkProgram(shaderProgram); // 链接程序
  gl.useProgram(shaderProgram); //使用着色器
}

function main() {
  var canvas = document.getElementById('container');
  var gl =
    canvas.getContext('webgl') || canvas.getContext('experimental-webgl');
  initShader(gl); // 初始化着色器
  gl.clearColor(0.0, 0.0, 0.0, 1.0); // 指定清空canvas的颜色
  gl.clear(gl.COLOR_BUFFER_BIT); // 清空canvas
  gl.drawArrays(gl.POINTS, 0, 1); // 画点
}
```

向 shader 中传值有两种方式：

- attribute 变量，传递与顶点相关的数组，只能在顶点着色器中使用；
- uniform 变量，传递与顶点无关的数据；

前面的代码将点的位置和大小都直接写在了顶点着色器中，现在将其改为由外面的程序传入。首先修改顶点着色器：

```js
const vertexShaderSrc = `
    attribute vec4 a_Position;// 接收传入位置坐标，必须声明为全局
    attribute float a_PointSize;// 接收传入位置坐标，必须声明为全局
    void main(){
        gl_Position = a_Position;// gl_Position 内置变量，表示点的位置，必须赋值
        gl_PointSize = a_PointSize;// gl_PointSize 内置变量，表示点的大小（单位像素），可以不赋值，默认为1.0
    }
`;
```

然后在 initShader 的最后给这两个变量赋值：

```js
// 获取shader中的a_Position变量
vaconstr a_Position = gl.getAttribLocation(shaderProgram, 'a_Position');
// 给变量a_Position赋值
gl.vertexAttrib4f(a_Position, 0.0, 0.0, 0.0, 1.0);

// 获取shader中的a_PointSize变量
vaconstr a_PointSize = gl.getAttribLocation(shaderProgram, 'a_PointSize');
// a_PointSize
gl.vertexAttrib1f(a_PointSize, 10.0);
```
