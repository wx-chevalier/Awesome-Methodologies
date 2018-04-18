# Web 构建与打包工具盘点

不同的构建工具有其不同的适用场景，

Webpack 是非常优秀的构建与打包工具，但是其提供了基础且复杂的功能支持，使得并不适用于全部的场景。

笔者在本文中列举讨论的仅是日常工作中会使用的工具，更多的 [Browserify](https://github.com/browserify/browserify)、[Fusebox](https://github.com/fuse-box/fuse-box) 等等构建工具查看 [Web 构建与打包工具资料索引](https://parg.co/Uss)。

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
  }
}
```

# Webpack

# Backpack
