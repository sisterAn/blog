用户通过浏览器访问网页（JS、CSS资源），当文件体积越大时，网页的加载时间就越长，使用的流量就越大，用户的体验就越糟糕，所以我们需要对代码进行压缩。

目前比较成熟的压缩工具有两种：

- UglifyJsPlugin：通过封装 UglifyJS 实现压缩
- ParallelUglifyPlugin：多进程并行处理压缩

他们都会分析 JavaScript 代码语法树，理解代码含义，从而能做到诸如去掉无效代码、去掉日志输出代码、缩短变量名等优化，但相对于 UglifyJsPlugin 是单线程， ParallelUglifyPlugin 插件实现了多线程压缩，ParallelUglifyPlugin 会开启多个子进程，把对多个文件的压缩工作分配给多个子进程去完成，每个子进程其实还是通过 UglifyJS 去压缩代码，但是变成了并行执行。 所以 ParallelUglifyPlugin 能更快的完成对多个文件的压缩工作。

### 一、压缩 Js

#### UglifyJsPlugin

```js
module.exports = {
  plugins: [
    // 压缩输出的 JS 代码
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        // 在UglifyJs删除没有用到的代码时不输出警告
        warnings: false,
        // 删除所有的 `console` 语句，可以兼容ie浏览器
        drop_console: true,
        // 内嵌定义了但是只用到一次的变量
        collapse_vars: true,
        // 提取出出现多次但是没有定义成变量去引用的静态值
        reduce_vars: true,
      },
      output: {
        // 最紧凑的输出
        beautify: false,
        // 删除所有的注释
        comments: false,
      }
    }),
  ],
};
```

或在启动打包时加上 `--optimize-minimize` ，这样 Webpack 会自动为你注入一个带有默认配置的 UglifyJSPlugin 。

**注意：webpack4 中 `webpack.optimize.UglifyJsPlugin` 已被废弃**

```js
module.exports = {
    optimization: {
        minimize: true,
    },
}
```



#### 使用 ParallelUglifyPlugin

前面介绍了如何利用缓存机制（最优配置、HardSourceWebpackPlugin 或 DllPlugin）提高 webpack 构建速度， 除此之外，我们知道 webpack 构建代码时，不仅仅是从入口递归解析打包，还包括压缩代码这一环节。

由于压缩 JavaScript 代码需要先把代码解析成用 Object 抽象表示的 AST 语法树，再去应用各种规则分析和处理 AST，导致这个过程计算量巨大，耗时非常多。

老版本 webpack 利用 webpack.optimize.UglifyJsPlugin 进行文件压缩，但是此插件是单线程，可利用 ParallelUglifyPlugin 插件实现多线程压缩，ParallelUglifyPlugin 则会开启多个子进程，把对多个文件的压缩工作分配给多个子进程去完成，每个子进程其实还是通过 UglifyJS 去压缩代码，但是变成了并行执行。 所以 ParallelUglifyPlugin 能更快的完成对多个文件的压缩工作。

使用 ParallelUglifyPlugin 也非常简单，把原来 Webpack 配置文件中内置的 UglifyJsPlugin 去掉后，再替换成 ParallelUglifyPlugin，相关代码如下：

```js
// ...
const ParallelUglifyPlugin = require('webpack-parallel-uglify-plugin');

module.exports = {
  // ...
  plugins: [
    // ...
    // 使用 ParallelUglifyPlugin 并行压缩输出的 JS 代码
    new ParallelUglifyPlugin({
      // 传递给 UglifyJS 的参数
      uglifyJS: {
        // 在UglifyJs删除没有用到的代码时不输出警告
        warnings: false,
        compress: {
          drop_debugger: true,
          // 删除所有的 `console` 语句，可以兼容ie浏览器
          drop_console: true,
          // 内嵌定义了但是只用到一次的变量
          collapse_vars: true,
          // 提取出出现多次但是没有定义成变量去引用的静态值
          reduce_vars: true,
        },
        output: {
          // 最紧凑的输出
          beautify: false,
          // 删除所有的注释
          comments: false
        }
      }
    }),    
  ]
}
```



### 二、提取 css

在使用 webapck 打包时，我们通常会把 css 提取到单独的 css 样式文件中，不再内联到 JS 包中，在项目运行的时候，速度更快，因为 CSS 包和 JS 包是并行加载的，其中提取 css 的插件有：

- extract-text-webpack-plugin： webpack3 及之前版本
- mini-css-extract-plugin：webapck4 及之后版本

根据具体项目中使用的 webpack 版本选择相应的插件。

#### extract-text-webpack-plugin

webpack3 及之前：

```js
const ExtractTextPlugin = require('extract-text-webpack-plugin')
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        // ExtractTextPlugin.extract 从现有加载器创建提取加载器
        use: ExtractTextPlugin.extract({
          fallback: "style-loader",
          use: "css-loader"
        })
      }
    ]
  },
  plugins: [
    new ExtractTextPlugin({
      // 被提取css文件名称
      filename: 'stylesheet/[name].[chunkhash:8].css',
      // 是否禁用
      disable: false,
      // 是否从所有块中提取，默认只从初始块中提取
      // 注意在使用 CommonsChunkPlugin 提取公共块时，
      // 若需要从公共块中提取css，必须设置allChunks为true
      allChunks: true,
    })
  ]
}
```

#### mini-css-extract-plugin

webpack4 或之后，该插件将 CSS 提取到单独的文件中。 它为每个包含 CSS 的 JS 文件创建一个 CSS 文件。 它支持 CSS 和 SourceMap 的按需加载。

该插件通常在没有 style-loader 的 production 环境中使用：

```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const devMode = process.env.NODE_ENV !== 'production';

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          devMode ? 'style-loader' ? MiniCssExtractPlugin.loader, 
          'css-loader'
        ],
      },
    ],
  },
  plugins: [
    new MiniCssExtractPlugin({
	  filename: devMode ? '[name].css' : '[name].[hash:8].css',
	  chunkFilename: devMode ? '[id].css' : '[id].[hash:8].css',
	  disable: false,
	  allChunks: true,
	}),
  ],
};
```



### 三、压缩 css

#### optimize-css-assets-webpack-plugin

在 webpack 构建过程中搜索 CSS ，并优化、最小化 CSS。

```js
var OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');

module.exports = {
  plugins: [
    new OptimizeCssAssetsPlugin({
      // ExtractTextPlugin 或 MiniCssExtractPlugin导出的文件名
      assetNameRegExp: /\.css$/g,
      // 用于优化/最小化CSS的CSS处理器，默认为cssnano
      cssProcessor: require('cssnano'),
      cssProcessorPluginOptions: {
        preset: ['default', { discardComments: { removeAll: true } }],
      },
      // 控制插件是否可以将消息打印到控制台，默认为 true
      canPrint: true
    })
  ]
};
```
