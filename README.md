# Webpack

本质：静态模块打包工具

原理：内部构建依赖图，映射项目所需模块，并生成 bundle

## 为什么使用 Webpack？

> 在浏览器中运行 JavaScript 有两种方法。
>
> 第一种方式，引用一些脚本来存放每个功能；此解决方案很难扩展，因为加载太多脚本会导致网络瓶颈。
>
> 第二种方式，使用一个包含所有项目代码的大型 .js 文件，但是这会导致作用域、文件大小、可读性和可维护性方面的问题。

### 立即调用函数表达式 - Immediately invoked function expressions

IIFE 解决大型项目的作用域问题，当脚本文件被封装在 IIFE 内部时，你可以安全地拼接或安全地组合所有文件，而不必担心作用域冲突。

但是，修改一个文件意味着必须重新构建整个文件。拼接可以做到很容易地跨文件重用脚本，但是却使构建结果的优化变得更加困难。

### Node.js && CommonJS

Node.js 是一个 JavaScript 运行时，可以在浏览器环境之外的计算机和服务器中使用。

webpack 运行在 Node.js 中。

CommonJS 问世并引入了 require 机制，它允许你在当前文件中加载和使用某个模块。导入需要的每个模块，这一开箱即用的功能，帮助我们解决了作用域问题。

CommonJS 是 Node.js 项目的绝佳解决方案，但浏览器不支持模块，CommonJS 没有浏览器支持。因而创建了 Browserify, RequireJS 和 SystemJS 等打包工具，允许我们编写能够在浏览器中运行的 CommonJS 模块。

### ESM

模块正在成为 ECMAScript 标准的官方功能。然而，浏览器支持不完整。

### Webpack

这就是 webpack 存在的原因。它是一个工具，可以打包你的 JavaScript 应用程序（支持 ESM 和 CommonJS），可以扩展为支持许多不同的资产，例如：images, fonts 和 stylesheets。 w webpack 关心性能和加载时间；它始终在改进或添加新功能，例如：异步地加载 chunk 和预取，以便为你的项目和用户提供最佳体验。

---

核心概念：

- 入口 entry

  构建内部依赖图的起点

- 输出 output

  告诉 webpack 在哪里输出它所创建的 bundle，以及如何命名这些文件

- 转译器 loader

  让 webpack 能够去处理其他类型的文件，并将它们转换为有效模块

- 插件 plugins

  用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量。插件目的在于解决 loader 无法实现的其他事。

- 模式 mode

  设置 mode，启用 webpack 内置在相应环境下的优化

- 浏览器兼容性

```js
// nodejs 核心模块
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

module.exports = {
  // 通过选择 development, production 或 none 之中的一个，来设置 mode 参数，你可以启用 webpack 内置在相应环境下的优化。其默认值为 production。
  mode: 'production',
  // 入口
  // https://v4.webpack.docschina.org/concepts/entry-points/
  entry: './path/to/my/entry/file.js',
  // entry: {
  //   main: './path/to/my/entry/file.js',
  // },

  // 告诉 webpack 在哪里输出它所创建的 bundle，以及如何命名这些文件
  // https://v4.webpack.docschina.org/configuration/output
  output: {
    // path 指定 bundle 生成(emit)到哪里
    path: path.resolve(__dirname, 'dist'),
    // path: '/home/proj/cdn/assets/[hash]',

    // 在编译时不知道输出地址，可在运行时通过入口起点文件中的 __webpack_public_path__ 动态设置
    // __webpack_public_path__ = myRuntimePublicPath;
    // publicPath: 'http://cdn.example.com/assets/[hash]/'

    // webpack bundle 的名称
    filename: 'my-first-webpack.bundle.js',

    // 占位符，确保每个文件具有唯一的名称
    // https://v4.webpack.docschina.org/configuration/output#output-filename
    // filename: '[name].js',
  },
  /*
  “嘿，webpack 编译器，当你碰到「在 require()/import 语句中被解析为 '.txt' 的路径」时，在你对它打包之前，先 使用 raw-loader 转换一下。”
  */
  // https://v4.webpack.docschina.org/concepts/loaders
  module: {
    // test-用于标识出应该被对应的 loader 进行转换的某个或某些文件
    // 使用正则表达式匹配文件时，你不要为它添加引号
    // use-表示进行转换时，应该使用哪个 loader
    rules: [
      { test: /\.txt$/, use: 'raw-loader' },
      { test: /\.ts$/, use: 'ts-loader' },
      // 多loader，从右向左执行
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              modules: true,
            },
          },
          { loader: 'sass-loader' },
        ],
      },
    ],
  },
  // 插件列表 https://v4.webpack.docschina.org/plugins
  plugins: [
    // html-webpack-plugin 为应用程序生成 HTML 一个文件，并自动注入所有生成的 bundle。
    new HtmlWebpackPlugin({ template: './src/index.html' }),
  ],
};
```

## 入口 entry

entry: string|Array\<string>

### 当你向 entry 传入一个数组时会发生什么？

向 entry 属性传入文件路径数组，将创建出一个 多主入口(multi-main entry)。在你想要一次注入多个依赖文件，并且将它们的依赖导向(graph)到一个 chunk 时，这种方式就很有用。

entry: {[entryChunkName: string]: string|Array\<string>}

---

## 输出 output

控制 webpack 如何向硬盘写入编译文件。
即使可以存在多个 entry 起点，但只指定一个 output 配置。

如果配置创建了多个单独的 "chunk"（例如，使用多个入口起点或使用像 CommonsChunkPlugin 这样的插件），则应该使用 占位符(substitutions) 来确保每个文件具有唯一的名称。

---

## 模式 mode

none, development 或 production（默认）

提供 mode 配置选项，告知 webpack 使用相应环境的内置优化。

```bash
  webpack --mode=production
```

记住，设置 NODE_ENV 并不会自动地设置 mode。

---

## 转译器 loader

通过 loader，webpack 可以支持以各种语言和预处理器语法编写的模块。loader 描述了 webpack 如何处理 非 JavaScript _模块_，并且在 bundle 中引入这些*依赖*。

1. 安装相应的 loader
2. 配置 rules，对应类型使用指定 loader 处理

使用 loader 的三种方式：

- 配置
- 内联

  `import Styles from 'style-loader!css-loader?modules!./styles.css';`

- cli

  `webpack --module-bind jade-loader --module-bind 'css=style-loader!css-loader'`

配置多个 loader，从右向左执行

### 特性

- loader 支持链式传递
- loader 可以是同步的，也可以是异步的
- loader 运行在 Node.js 中，并且能够执行任何 Node.js 能做到的操作
- loader 可以通过 options 对象配置
- loader 能够产生额外的任意文件
- 插件(plugin)可以为 loader 带来更多特性

---

## 插件 plugin

_tapable_

webpack 插件是一个具有 apply 方法的 JavaScript 对象。apply 方法会被 webpack compiler 调用，并且 compiler 对象可在整个编译生命周期访问。

ConsoleLogOnBuildWebpackPlugin.js

```js
const pluginName = 'ConsoleLogOnBuildWebpackPlugin';

class ConsoleLogOnBuildWebpackPlugin {
  apply(compiler) {
    compiler.hooks.run.tap(pluginName, (compilation) => {
      console.log('webpack 构建过程开始！');
    });
  }
}
```

compiler hook 的 tap 方法的第一个参数，应该是驼峰式命名的插件名称。建议为此使用一个常量，以便它可以在所有 hook 中复用。

### 用法

由于插件可以携带参数/选项，你必须在 webpack 配置中，向 plugins 属性传入 new 实例。

```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); //通过 npm 安装
const webpack = require('webpack'); //访问内置的插件
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    filename: 'my-first-webpack.bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader',
      },
    ],
  },
  plugins: [
    new webpack.ProgressPlugin(),
    new HtmlWebpackPlugin({ template: './src/index.html' }),
  ],
};
```

---

## 模块 module

将程序分解为功能离散的 chunk(discrete chunks of functionality)，并称之为*模块*

Node.js 从最一开始就支持模块化编程。
