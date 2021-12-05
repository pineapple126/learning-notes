# 内部原理

打包，是指处理某些文件并将其输出为其他文件的能力。

在输入和输出之间，还包括有模块，entry，chunk，chunk 组和许多其他中间部分。

## 主要部分

项目中使用的每个文件都是一个模块

```javascript
// ./index.js

import app from './app.js';


// ./app.js

export default 'the app';
```

通过相互引用，这些模块会形成一个依赖图。在打包过程中，模块会被合并为 chunk，chunk 合并成 chunk 组，并形成一个通过模块互相连接的图。在其内部，创建一个只有一个 chunk 的 chunk 组作为一个入口起点。

```javascript
// ./webpack.config.js

module.exports = {
  entry: './index.js'
};
```

这会创建一个名为 `main` 的 chunk 组（`main` 是入口起点的默认名称）。此 chunk 组包含 `./index.js` 模块。随着 parser 处理 `./index.js` 内部的 import 时，新模块就会被添加到此 chunk 中。

另一个示例：

```javascript
// ./webpack.config.js

module.exports = {
  entry: {
    home: './home.js',
    about: './about.js'
  }
};
```

这将会创建两个名为 `home` 和 `about` 的 chunk 组。每个 chunk 组都有一个包含一个模块的 chunk：`./home.js` 对应 `home`，`./about.js` 对应 `about`。

## chunk

chunk 有两种形式：

- `initial（初始化）` 是入口起点的 main chunk。此 chunk 包含为入口起点指定的所有模块及其依赖项。
- `non-initial` 是可以延迟加载的块。可能会出现在使用*动态导入*时

每个 chunk 都有对应的 **asset（资源，指输出文件）**。

```javascript
// webpack.config.js

module.exports = {
  entry: './src/index.jsx'
};


// ./src/index.jsx

import React from "react";
import ReactDOM from "react-dom";

import('./app.jsx').then((App) => {
  ReactDOM.render(<App />, root);
});
```

这将会创建一个名为 `main` 的 initial chunk。其中包含：

- `./src/index.jsx`
- `React`
- `ReactDOM`
- 除 `./app.jsx` 外的所有依赖

然后会为 `./app.jsx` 创建 non-initial chunk，这是因为 `./app.jsx` 是动态导入的。

**Output**：

- `/dist/main.js` - 一个 initial chunk
- `/dist/394.js` - 一个 non-initial chunk

默认情况下，non-initial chunk 都没有名称，因此会使用唯一 ID 来替代名称。在使用动态导入时，可以通过使用 magic comment（魔术注释）来显式指定 chunk 名称：

```javascript
import(
  /* webpackChunkName: "app" */
  './app.jsx'
).then((App) => {
  ReactDOM.render(<App />, root);
});
```

**Output**：

- `/dist/main.js` - 一个 initial chunk
- `/dist/app.js` - 一个 non-initial chunk

## output

输出文件的名称会受配置中的两个字段影响：

- `output.filename` - 用于 initial chunk
- `ouput.chunkFilename` 用于 non-initial chunk
