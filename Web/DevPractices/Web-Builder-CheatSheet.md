# Web 构建与打包工具盘点

不同的构建工具有其不同的适用场景，

Webpack 是非常优秀的构建与打包工具，但是其提供了基础且复杂的功能支持，使得并不适用于全部的场景。

![webpack](https://user-images.githubusercontent.com/5803001/39966649-02e751a6-56e2-11e8-8af1-bbbd47aa7dbb.png)

笔者在本文中列举讨论的仅是日常工作中会使用的工具，更多的 [Browserify](https://github.com/browserify/browserify)、[Fusebox](https://github.com/fuse-box/fuse-box) 等等构建工具查看 [Web 构建与打包工具资料索引](https://parg.co/Uss)。

Grunt、Glup 属于 Task Runner，即任务执行器； 实际上，npm package.json 中定义的脚本也可以看做 Task Runner，而 Rollup，Parcel 以及 Webpack 则是属于 Bundler，即打包工具。

# Parcel

```sh
npm install -g parcel-bundler
```

[iReactPack](https://github.com/wxyyxc1992/iReactPack)

# Rollup + Microbundle

```js
import { rollup } from 'rollup';
import babel from 'rollup-plugin-babel';

rollup({
  entry: 'main.js',
  plugins: [
    babel({
      exclude: 'node_modules/**'
    })
  ]
}).then(...)
```

```js
import typescript from 'rollup-plugin-typescript';

export default {
  entry: './main.ts',

  plugins: [typescript()]
};
```

[Microbundle](https://github.com/developit/microbundle) 则是 Developit 基于 Rollup 封装的零配置的轻量级打包工具，其目前已经内建支持 TypeScript 与 Flow，不需要额外的配置；笔者在 [x-fetch](https://github.com/wxyyxc1992/js-swissgear/tree/master/x-fetch) 项目的打包中也使用了该工具。

```json
{
  "scripts": {
    "build": "microbundle",
    "dev": "microbundle watch"
  }\
}
```

index.js is the CommonJS module. This is module type used by Node and it looks like const myModule = require('my-module')
index.m.js is the ECMAScript Module, as defined in ES6, it looks like import MyModule from 'my-module'
index.umd.js is the UMD module
index.d.ts is TypeScript type declaration file

# Webpack

作为著名的打包工具，Webpack 允许我们指定项目的入口地址，然后自动将用到的资源，经由 Loader 与 Plugin 的转换，打包到包体文件中。[fe-boilerplate/react-webpack](https://github.com/wxyyxc1992/fe-boilerplate/blob/master/react/webpack)

![538c4af0d21e375d6d252d38cbb8a993](https://user-images.githubusercontent.com/5803001/39744493-0e21c33a-52d7-11e8-8295-1f8deb389565.png)

Webpack 目前也支持零配置运行

```js
$ npm install webpack webpack-cli webpack-dev-server --save-dev
```

```json
"scripts": {
  "start": "webpack-dev-server --mode development",
  "build": "webpack --mode production"
},
```

## 基础配置

```js
const config = {
  // 定义入口
  entry: {
    app: path.join(__dirname, 'app')
  },
  // 定义包体文件
  output: {
    // 输出目录
    path: path.join(__dirname, 'build'),

    // 输出文件名
    filename: '[name].js'
    // 使用 hash 作为文件名
    // filename: "[name].[chunkhash].js",
  },
  // 定义如何处理
  module: {
    rules: [
      {
        test: /\.js$/,
        use: 'babel-loader',
        exclude: /node_modules/
      }
    ]
  },
  // 添加额外插件操作
  plugins: [new webpack.DefinePlugin()]
};
```

Webpack 同样支持添加多个配置：

```js
module.exports = [{
  entry: './app.js',
  output: ...,
  ...
}, {
  entry: './app.js',
  output: ...,
  ...
}]
```

我们代码中的 require 与 import 解析规范，则由 resolve 模块负责，其包含了扩展、别名、模块等部分：

```js
const config = {
  resolve: {
    alias: {
      /*...*/
    },
    extensions: [
      /*...*/
    ],
    modules: [
      /*...*/
    ]
  }
};
```

## 资源加载

```js
const config = {
  module: {
    rules: [
      {
        // **Conditions**
        test: /\.js$/, // Match files
        enforce: 'pre', // "post" too

        // **Restrictions**
        include: path.join(__dirname, 'app'),
        exclude: path => path.match(/node_modules/),

        // **Actions**
        use: 'babel-loader'
      }
    ]
  }
};
```

```js
// Process foo.png through url-loader and other matches
import 'url-loader!./foo.png';

// Override possible higher level match completely
import '!!url-loader!./bar.png';
```

babel-loader 或者 awesome-typescript-loader 来处理 JavaScript 或者 TypeScript 文件

```js
/******/ (function(modules) { // webpackBootstrap
...
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony default export */ __webpack_exports__["default"] = ((text = "Hello world") => {
  const element = document.createElement("div");

  element.innerHTML = text;

  return element;
});

/***/ })
/******/ ]);
```

`use: ["style-loader", "css-loader"]` css-loader 会自动地解析 @import 与 url()，而 style-loader 则会将 CSS 注入到 DOM 中，并且实现 HMR 的特性，而对于 SASS、LESS 等 CSS 预处理器，也有专门的 sass-loader 或者 less-loader 来处理；在生产环境下，我们也常常会将 CSS 抽取到独立的样式文件中，此时就可以使用 mini-css-extract-plugin (MCEP) 等工具。

同样，我们可以使用 url-loader/file-loader 来处理图片等资源文件，

## 代码分割

代码分割是[提升 Web 性能表现](https://parg.co/lwm)的重要分割，我们常做的代码分割也分为公共代码提取与按需加载等方式。公共代码提取即是将第三方渲染模块或者库与应用本身的逻辑代码分割，或者将应用中多个模块间的公共代码提取出来，划分到独立的 Chunk 中，以方便客户端进行缓存等操作。

![cc11f7e53c579fff28de1b3ed5b9f53a](https://user-images.githubusercontent.com/5803001/39862950-c8ba51c0-5477-11e8-892c-a2b6ec619e2d.png)

不同于 Webpack 3 中需要依赖 CommonChunksPlugin 进行配置，Webpack 4 为我们提供了开箱即用的代码优化特性。

```js
module.exports = {
  /* ... */
  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendor',
          chunks: 'all'
        }
      }
    }
  }
};
```

我们也可以在代码中使用 import 语句，动态地进行块划分，实现代码的按需加载：

![c4e91fafb1a08e7733ac2688222eb65a](https://user-images.githubusercontent.com/5803001/39863036-0aaf92d4-5478-11e8-929c-9f07e8dca3b8.png)

```js
// Webpack 3 之后支持显式指定 Chunk 名
import(/* webpackChunkName: "optional-name" */ './module')
  .then(module => {
    /* ... */
  })
  .catch(error => {
    /* ... */
  });
```

```js
webpackJsonp([0], {
  KMic: function(a, b, c) {
    ...
  },
  co9Y: function(a, b, c) {
    ...
  },
});
```

如果是使用 React 进行项目开发，推荐使用 [react-loadable](https://www.npmjs.com/package/react-loadable) 进行组件的按需加载，他能够优雅地处理组件加载、服务端渲染等场景。Webpack 还内建支持基于 ES6 Module 规范的 Tree Shaking 优化，即仅从导入文件中提取出所需要的代码：

```

```

更多关于 Webpack 的使用技巧可以参阅 [Webpack CheatSheet]() 或者[现代 Web 开发基础与工程实践/Webpack]() 章节。

# Backpack
