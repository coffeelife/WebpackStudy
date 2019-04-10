# webpack使用学习

官网链接(中文文档)：https://www.webpackjs.com

![5cac4aeae47e4](https://i.loli.net/2019/04/09/5cac4aeae47e4.png)

## 概念

本质上，*webpack*是一个现代 JavaScript 应用程序的*静态模块打包器(module bundler)*。当 webpack 处理应用程序时，它会递归地构建一个*依赖关系图(dependency graph)*，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个*bundle*。

### 四个核心概念

-   入口(entry)
    
-   输出(output)
    
-   loader
    
-   插件(plugins)
    

## 入口(entry)

**入口起点(entry point)**指示 webpack 应该使用哪个模块，来作为构建其内部*依赖图*的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。可以通过在webpack 配置中配置`entry`属性，来指定一个入口起点（或多个入口起点）。默认值为`./src`。

### 单入口模式

用法：`entry: string|Array<string>`

**webpack.config.js**

```
//简写
module.exports = {
  entry: './path/to/my/entry/file.js'
};

module.exports = {
  entry: {
    main: './path/to/my/entry/file.js'
  }
};
```

### 多入口模式

用法：`entry: {[entryChunkName: string]: string|Array<string>}`

**webpack.config.js**

```
module.exports = {
  entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
  }
};
```

**根据经验：每个 HTML 文档只使用一个入口起点。**

## 出口(output)

### 用法(Usage)

**output**属性告诉 webpack 在哪里输出它所创建的*bundles*，以及如何命名这些文件，默认值为`./dist`。基本上，整个应用程序结构，都会被编译到你指定的输出路径的文件夹中。你可以通过在配置中指定一个`output`字段，来配置这些处理过程。

在 webpack 中配置`output`属性的最低要求是，将它的值设置为一个对象，包括以下两点：

-   `filename`用于输出文件的文件名。
    
-   目标输出目录`path`的绝对路径。
    

### 单入口起点

**webpack.config.js**

```
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```

我们通过`output.filename`和`output.path`属性，来告诉 webpack bundle 的名称，以及我们想要 bundle 生成(emit)到哪里。

### 多入口起点

**webpack.config.js**

```
{
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: '[name].js',
    path: __dirname + '/dist'
  }
}
//写入到硬盘：./dist/app.js, ./dist/search.js
```

如果配置创建了多个单独的 "chunk"（例如，使用多个入口起点或使用像 CommonsChunkPlugin 这样的插件），则应该使用占位符(substitutions)来确保每个文件具有唯一的名称。

## 模式(mode)

### 用法

只在配置中提供`mode`选项：

```
module.exports = {
  mode: 'production'
};
```

或者从CLI参数中传递：

```
webpack --mode=production
```

| 选项 | 描述 |
| --- | --- |
| development | 会将`process.env.NODE_ENV`的值设为`development`。启用`NamedChunksPlugin`和`NamedModulesPlugin`。 |
| production | 会将`process.env.NODE_ENV`的值设为`production`。启用`FlagDependencyUsagePlugin`,`FlagIncludedChunksPlugin`,`ModuleConcatenationPlugin`,`NoEmitOnErrorsPlugin`,`OccurrenceOrderPlugin`,`SideEffectsFlagPlugin`和`UglifyJsPlugin`. |

**mode: development**

```
// webpack.development.config.js
module.exports = {
+ mode: 'development'
- plugins: [
-   new webpack.NamedModulesPlugin(),
-   new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("development") }),
- ]
}
```

**mode: production**

```
// webpack.production.config.js
module.exports = {
+  mode: 'production',
-  plugins: [
-    new UglifyJsPlugin(/* ... */),
-    new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("production") }),
-    new webpack.optimize.ModuleConcatenationPlugin(),
-    new webpack.NoEmitOnErrorsPlugin()
-  ]
}
```

## loader

loader 用于对模块的源代码进行转换。loader 可以使你在`import`或"加载"模块时预处理文件。因此，loader 类似于其他构建工具中“任务(task)”，并提供了处理前端构建步骤的强大方法。loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript，或将内联图像转换为 data URL。loader 甚至允许你直接在 JavaScript 模块中`import`CSS文件！

### 安装

```
npm install --save-dev css-loader
npm install --save-dev ts-loader
```

**webpack.config.js**

```
module.exports = {
  module: {
    rules: [
      { test: /\.css$/, use: 'css-loader' },
      { test: /\.ts$/, use: 'ts-loader' }
    ]
  }
};
```

指示 webpack 对每个`.css`使用[`css-loader`](https://www.webpackjs.com/loaders/css-loader)，以及对所有`.ts`文件使用[`ts-loader`](https://github.com/TypeStrong/ts-loader)：

### 使用loader

在你的应用程序中，有三种使用 loader 的方式：

-   配置（推荐）：在**webpack.config.js**文件中指定 loader。
    
-   内联：在每个`import`语句中显式指定 loader。
    
-   CLI：在 shell 命令中指定它们。
    

### 配置(Configuration)

[`module.rules`](https://www.webpackjs.com/configuration/module/#module-rules)允许你在 webpack 配置中指定多个 loader。 这是展示 loader 的一种简明方式，并且有助于使代码变得简洁。同时让你对各个 loader 有个全局概览：

```
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
          }
        ]
      }
    ]
  }
```

### 内联

可以在`import`语句或任何等效于 "import" 的方式中指定 loader。使用`!`将资源中的 loader 分开。分开的每个部分都相对于当前目录解析。

```
import Styles from 'style-loader!css-loader?modules!./styles.css';
```

通过前置所有规则及使用`!`，可以对应覆盖到配置中的任意 loader。

选项可以传递查询参数，例如`?key=value&foo=bar`，或者一个 JSON 对象，例如`?{"key":"value","foo":"bar"}`。

> 尽可能使用`module.rules`，因为这样可以减少源码中的代码量，并且可以在出错时，更快地调试和定位 loader 中的问题。

### CLI

你也可以通过 CLI 使用 loader：

```
webpack --module-bind jade-loader --module-bind 'css=style-loader!css-loader'
```

这会对`.jade`文件使用`jade-loader`，对`.css`文件使用[`style-loader`](https://www.webpackjs.com/loaders/style-loader)和[`css-loader`](https://www.webpackjs.com/loaders/css-loader)。

### loader特性

loader 通过（loader）预处理函数，为 JavaScript 生态系统提供了更多能力。 用户现在可以更加灵活地引入细粒度逻辑，例如压缩、打包、语言翻译和其他更多。

-   loader 支持链式传递。能够对资源使用流水线(pipeline)。一组链式的 loader 将按照相反的顺序执行。loader 链中的第一个 loader 返回值给下一个 loader。在最后一个 loader，返回 webpack 所预期的 JavaScript。
    
-   loader 可以是同步的，也可以是异步的。
    
-   loader 运行在 Node.js 中，并且能够执行任何可能的操作。
    
-   loader 接收查询参数。用于对 loader 传递配置。
    
-   loader 也能够使用`options`对象进行配置。
    
-   除了使用`package.json`常见的`main`属性，还可以将普通的 npm 模块导出为 loader，做法是在`package.json`里定义一个`loader`字段。
    
-   插件(plugin)可以为 loader 带来更多特性。
    
-   loader 能够产生额外的任意文件。
    

## 插件(plugins)

插件是 webpack 的支柱功能。webpack 自身也是构建于，你在 webpack 配置中用到的**相同的插件系统**之上！插件目的在于解决loader无法实现的**其他事**。

### 剖析

webpack**插件**是一个具有[`apply`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)属性的 JavaScript 对象。`apply`属性会被 webpack compiler 调用，并且 compiler 对象可在**整个**编译生命周期访问。

**ConsoleLogOnBuildWebpackPlugin.js**

```
const pluginName = 'ConsoleLogOnBuildWebpackPlugin';

class ConsoleLogOnBuildWebpackPlugin {
    apply(compiler) {
        compiler.hooks.run.tap(pluginName, compilation => {
            console.log("webpack 构建过程开始！");
        });
    }
}
```

compiler hook 的 tap 方法的第一个参数，应该是驼峰式命名的插件名称。建议为此使用一个常量，以便它可以在所有 hook 中复用。

### 用法

由于**插件**可以携带参数/选项，你必须在 webpack 配置中，向`plugins`属性传入`new`实例。

根据你的 webpack 用法，这里有多种方式使用插件。

### 配置

**webpack.config.js**

```
const HtmlWebpackPlugin = require('html-webpack-plugin'); //通过 npm 安装
const webpack = require('webpack'); //访问内置的插件
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    filename: 'my-first-webpack.bundle.js',
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
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```