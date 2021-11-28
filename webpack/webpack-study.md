# webpack 学习

- [webpack 学习](#webpack-学习)
  - [概念](#概念)
  - [entry](#entry)
  - [output](#output)
  - [loader](#loader)
  - [plugins](#plugins)
  - [配置（Configuration）](#配置configuration)
  - [模块（Modules）](#模块modules)

## 概念

webpack 是一个现代 JavaScript 应用程序的静态模块打包器。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图，然后将应用程序依赖的的模块打包成一个或多个模块。

webpack 有四个核心概念：

- 入口 entry
- 出口 output
- loader
- 插件 plugins

这些概念将决定 webpack 从哪里开始构建依赖图、转换哪些类型的模块、如何处理这些模块，最终将打包后的 bundles 输出到哪里。

## entry

entry 将指示 webpack 应该使用哪个模块，来作为构建依赖图的起点。

可以在 webpack 配置文件中配置 `entry` 选项，来指定一个或者多个入口起点，默认值为 `'./src'`。

用法：`entry: string | Array<string> | {[entryChunkName: string]: string | Array<string>}`

```javascript
// webpack.config.js

// entry: string
module.exports = {
  entry: './src/file.js'
}

// entry: Array<string>
// 向 entry 属性传入数组将创建“多个”主入口。
module.exports = {
  entry: ['./src/file1.js', './src/file2.js']
}

// entry: {[entryChunkName: string]: string | Array<string>}
// 对象语法是最可扩展的定义入口的方式，可以将我们的关注点从环境、构建目标、运行时中分离。
// 我们可以分离应用程序和第三方库入口，也可以为多页面程序的每个页面提供一个入口
module.exports = {
  entry: {
    main: './src/index.js',
    vendor: ['./src/vendors1/js', './src/vendors2.js']
  }
}
```

## output

output 将指示 webpack 将打包后的 bundles 输出到哪里，默认值为 './dist'。

output 选项的值需要是一个对象，至少包含两个字段：

- filename，决定输出文件的名称
- path，决定输出文件的绝对路径

```javascript
const path = require('path');

// 此配置将在 ./dist 目录下输出 bundle.js 文件
module.exports = {
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, './dist') // 项目等同于 __dirname + './dist/
  }
}

// 如果配置创建了多个 chunk，则需要使用占位符来确保每个文件具有唯一的名称
// 输出文件： ./dist/main.js   ./dist/search.js
module.exports = {
  entry: {
    main: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, './dist')
  }
}
```

可使用占位符如下：

模版|描述
-|-
[name]|模块名称
[id]|模块标识符
[hash]|模块标识符的 hash
[chunkhash]|chunk 内容的 hash
[query]|模块的 query，例如，文件名 ? 后面的字符串

## loader

webpack 只能理解 JavaScript 和 JSON 文件。loader 向 webpack 描述了如何处理非原生模块，可以使我们在 `import` 或 `load` 模块时预处理其他类型的文件，并将其转换为有效模块，以供应用程序使用，以及被添加到依赖图中。

loader 一般使用配置文件的方式使用：

```javascript
// webpack.config.js

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          },
          { loader: 'sass-loader' }
        ]
      }
    ]
  }
}
```

`module.rules` 选项允许我们在 webpack 配置中配置多个指定的 loader，这种配置方式可以使我们清晰地了解到项目中使用的 loader。

loader 按照从右向左（从后向前）的顺序取值（evaluate）或执行（execute）。在上面的示例中，将先执行 `sass-loader`，再执行 `css-loader`，最后执行 `style-loader`。

loader 有如下特性：

- 链式调用。一组链式的 loader 将按照从后向前的顺序执行，前一个 loader 的结果将传递给下一个 loader。链中最后一个 loader 将返回 webpack 期望的 JavaScript
- 可同步执行，也可异步执行
- 可通过 `options` 对象配置？？？？

loader 遵循标准模块解析规则。多数情况下，loader 将从模块路径加载（通常是从 `npm install`，`mode_modules` 中进行加载）。

## plugins

loader 用于转换某些类型的模块，而插件则可以用于解决 loader 无法实现的其他事。

插件是一个具有 `apply` 方法的 JavaScript 对象。`apply` 方法会被 `webpack compiler` 调用，并且在**整个**编译生命周期都可以访问 `compiler` 对象。

```javascript
// ConsoleLogOnBuildWebpackPlugin.js

const pluginName = 'ConsoleLogOnBuildWebpackPlugin';

class ConsoleLogOnBuildWebpackPlugin {
  apply(compiler) {
    compiler.hooks.run.tap(pluginName, (compilation) => {
      console.log('webpack 构建正在启动！');
    });
  }
}

module.exports = ConsoleLogOnBuildWebpackPlugin;
```

使用插件时必须在 webpack 配置文件中，向 `plugins` 属性传入一个 `new` 实例。取决于使用的插件，还可以传递一些参数/选项。

```javascript
// webpack.config.js

const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack');
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader'
      }
    ]
  },
  plugins: [
    new webpack.ProgressPlugin(),
    new HtmlWebpackPlugin({ template: './src/index.html' })
  ]
}
```

`ProgressPlugin` 插件用于自定义编译过程中的进度报告，`HtmlWebpackPlugin` 插件将生成一个 HTML 文件，并在其中使用 `script` 引入一个名为 `index.bunlde.js` 的 JS 文件。

## 配置（Configuration）

**webpack 的配置文件是一个 JS 文件，文件内导出了一个 webpack 配置的对象**。webpack 会根据该配置定义的属性进行处理。

webpack 遵循 CommonJS 模块规范，因此我们可以在配置中使用：

- 通过 `require(...)` 引入其他文件
- 通过 `require(...)` 使用 npm 下载的工具函数
- 使用 JavaScript 控制流表达式
- 对 value 使用常量或变量赋值
- 编写并执行函数，生成部分配置

## 模块（Modules）

webpack 默认支持一下模块类型：

- ECMAScript 模块
- CommonJS 模块
- AMD 模块
- Assets 资源模块
- WebAssembly 模块

通过 loader 可以使 webpack 支持多种语言和预处理器语法编写的模块。
