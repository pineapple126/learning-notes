# webpack 学习

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
