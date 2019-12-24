虽然前面介绍了很多优化方法，但具体项目如何进行优化，这就要分析它的输出结果，来决定如何进行优化。

最直观的方式是分析它的输出文件，但 webpack 打包后的文件非常大且可读性差，下面就介绍几种可视化的分析工具，帮助你快速定位问题所在。

### 一、webpack-bundle-analyzer

它能方便的让你知道：

- 打包出的文件中都包含了什么；
- 每个文件的尺寸在总体中的占比，一眼看出哪些文件尺寸大；
- 模块之间的包含关系；
- 每个文件的 Gzip 后的大小。

接入 webpack-bundle-analyzer 的方法很简单，步骤如下：

- 安装 webpack-bundle-analyzer，执行命令 `npm install webpack-bundle-analyzer --save-dev`；

- 配置 webpack：

  ```js
  var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
  module.exports = {
    plugins: [
      // 开启 BundleAnalyzerPlugin
      new BundleAnalyzerPlugin(),
    ],
  };
  ```



### 二、官方可视化分析工具 Webapck Analyse

[Webapck Analyse](http://webpack.github.io/analyse/) 是一个在线 Web 应用。

**步骤一：生成构建文件**

在 webpack 启动时带入参数：`--profile --json > stats.json` ：

- `--profile`：记录下构建过程中的耗时信息
- `--json`：以 JSON 的格式输出构建结果，最后只输出一个 `.json` 文件，这个文件（stats.json）中包括所有构建相关的信息
- `> stats.json` ：是 UNIX/Linux 系统中的管道命令、含义是把 `webpack --profile --json` 输出的内容通过管道输出到 `stats.json` 文件中

启动后，会在项目中多出一个 `stats.json` 文件

**步骤二：Webpack Analyse**

打开 [Webapck Analyse](http://webpack.github.io/analyse/) ，上传 `stats.json`：

<img width="622" alt="upload-stats" src="https://user-images.githubusercontent.com/19721451/70489446-c5dcfe80-1b36-11ea-9f92-728e962978d4.png">

Webpack Analyse 不会把你选择的 `stats.json` 文件发达到服务器，而是在浏览器本地解析，你不用担心自己的代码为此而泄露。 选择文件后，你马上就能如下的效果图：
<img width="1344" alt="stats-analyse" src="https://user-images.githubusercontent.com/19721451/70489466-d55c4780-1b36-11ea-9a96-f77a97e6befe.png">

- **Modules**：展示所有的模块，每个模块对应一个文件。并且还包含所有模块之间的依赖关系图、模块路径、模块ID、模块所属 Chunk、模块大小；
- **Chunks**：展示所有的代码块，一个代码块中包含多个模块。并且还包含代码块的ID、名称、大小、每个代码块包含的模块数量，以及代码块之间的依赖关系图；
- **Assets**：展示所有输出的文件资源，包括 `.js`、`.css`、图片等。并且还包括文件名称、大小、该文件来自哪个代码块；
- **Warnings**：展示构建过程中出现的所有警告信息；
- **Errors**：展示构建过程中出现的所有错误信息；
- **Hints**：展示处理每个模块的过程中的耗时。



### 三、speed-measure-webpack-plugin 

优化 Webpack 构建速度的第一步是知道将精力集中在哪里。

这个插件可以测量您的webpack构建速度，

步骤一：

安装：`npm install speed-measure-webpack-plugin --save-dev`

步骤二：

```js
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");
const smp = new SpeedMeasurePlugin();
module.exports = smp.wrap({
  // ...
});
```