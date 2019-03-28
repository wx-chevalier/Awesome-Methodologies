[![返回目录](https://parg.co/UCb)](https://github.com/wx-chevalier/Awesome-CheatSheets)

![1_sso_vplej49wti_ubptvgq](https://user-images.githubusercontent.com/5803001/40587814-0000e1f2-6207-11e8-9e38-2e478a645c31.png)

> 📌 本文侧重于盘点 TypeScript 中类型声明与校验规则相关的知识点，对于与 ECMAScript 语法使用重合的部分建议阅读 [JavaScript CheatSheet](https://parg.co/Yha) 或者 [ECMAScript CheatSheet](https://parg.co/YhW)，对于 TypeScript 在 React/Redux 中的实践可以参阅 [React CheatSheet/TypeScript]()。需要声明的是，本文参考了 [TypeScript Links]() 中列举的很多文章或书籍，特别是官方的 [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/basic-types.html) 很值得仔细阅读。

# TypeScript CheatSheet | TypeScript 语法实践速览与实践清单

TypeScript 是由 MicroSoft 出品的 JavaScript 超集，它在兼容 JavaScript 的所有特性的基础上，附带了静态类型的支持；TypeScript 还允许我们使用尚未正式发布的 ECMAScript 的语言特性，在编译时进行类似于 Babel 这样的降级转化。

JavaScript 本身乃动态类型的语言，即是在运行时才进行类型校验；该特性赋予了其快速原型化的能力，却在构建大型 JavaScript 应用时力有不逮，其无法在编译时帮助规避可能的类型错误，也无法利用自动补全、自动重构等工具特性。TypeScript 的静态类型特性则帮助我们在编译时尽可能规避类型错误，并且 TypeScript 会尽可能地从上下文信息中进行类型推导，以避免像 Java 等静态类型语言中过于冗余的麻烦。

可以参考 [fe-boilerplat/\*-ts]() 或者 [Backend-Boilerplate/node]()，如果想了解 TypeScript 在 React 中的应用，可以参考 [React CheatSheet/TypeScript 节]()。们可以通过 npm 安装 TypeScript 的依赖包：

```sh
# 全局安装
$ npm install -g typescript

# 检测是否安装成功
$ tsc -v
Version 2.8.3
```

TypeScript 源文件一般使用 `.ts` 或者 `.tsx` 为后缀，其并不能直接运行在浏览器中而需要进行编译转化，TypeScript 的官方提供了 `tsc` 命令来进行文件编译：

```sh
$ tsc main.ts

# 同时编译多个文件
$ tsc main.ts worker.ts

# 编译当前目录下的全部 ts 文件，并不会递归编译
$ tsc *.ts

# 启动后台常驻编译程序
$ tsc main.ts --watch
```

在实际的项目中，我们也往往会在项目根目录配置 tsconfig.json 文件，来个性化配置 TypeScript 的编译参数：

```json
{
  "compilerOptions": {
    "outDir": "./dist/es",
    "declarationDir": "./dist/types",
    "target": "es5",
    "module": "commonjs",
    "jsx": "react",
    "downlevelIteration": true,
    "moduleResolution": "node",
    "allowUnreachableCode": true,
    "declaration": true,
    "experimentalDecorators": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "pretty": true,
    "skipLibCheck": true,
    "sourceMap": true,
    "strictNullChecks": true,
    "suppressImplicitAnyIndexErrors": true,
    "lib": ["dom", "es2015"],
    "baseUrl": "src"
  },
  "include": ["src/**/*", "typings/**/*"]
}
```

也可以使用 [ts-node](https://github.com/TypeStrong/ts-node) 快速地直接运行 TypeScript 文件。

# 类型机制

# 基础类型

# 函数与类

# Advanced Types | 进阶类型
