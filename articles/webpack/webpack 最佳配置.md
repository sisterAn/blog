### 前言

对于入门选手来讲，webpack 配置项很多很重，如何快速配置一个可用于线上环境的 webpack 就是一件值得思考的事情。其实熟悉 webpack 之后会发现很简单，基础的配置可以分为以下几个方面： `entry` 、 `output` 、 `mode` 、 `resolve` 、 `module` 、 `optimization` 、 `plugin` 、 `source map` 、 `performance` 等，本文就来重点分析下这些部分。

### 一、配置入口 entry

#### 1、单入口和多入口

 将源文件加入到 webpack 构建流程，可以是单入口:

```js
module.exports = {
  entry: `./index.js`,
}
```

构建包名称 `[name] `为 `main` ；

或多入口：

```js
module.exports = {
  entry: { 
    "index": `./index.js`,
  },
}
```

`key:value` 键值对的形式：

- key：构建包名称，即 `[name]` ，在这里为 `index` 
- value：入口路径

入口决定 webapck 从哪个模块开始生成依赖关系图（构建包），每一个入口文件都对应着一个依赖关系图。

#### 2. 动态配置入口文件

##### 动态打包所有子项目

当构建项目包含多个子项目时，每次增加一个子系统都需要将入口文件写入 webpack 配置文件中，其实我们让webpack 动态获取入口文件，例如：

```js
// 使用 glob 等工具使用若干通配符，运行时获得 entry 的条目
module.exports = {
  entry: glob.sync('./project/**/index.js').reduce((acc, path) => {
    const entry = path.replace('/index.js', '')
    acc[entry] = path
    return acc
  }, {}),
}
```

则会将所有匹配 `./project/**/index.js` 的文件作为入口文件进行打包，如果你想要增加一个子项目，仅仅需要在 `project` 创建一个子项目目录，并创建一个 `index.js` 作为入口文件即可。

这种方式比较适合入口文件不集中且较多的场景。

##### 动态打包某一子项目

在构建多系统应用或组件库时，我们每次打包可能仅仅需要打包某一模块，此时，可以通过命令行的形式请求打印某一模块，例如：

```js
npm run build --project components
```

在打包的时候解析命令行参数：

```js
// 解析命令行参数
const argv = require('minimist')(process.argv.slice(2))
// 项目
const project = argv['project'] || 'index'
```

然后配置入口：

```js
module.exports = {
  entry: { 
    "index": `./${project}/index.js`,
  } 
}
```

相当于：

```js
module.exports = {
  entry: { 
    "index": `./components/index.js`,
  } 
}
```

当然，你可以传入其它参数，也可以应用于多个地方，例如 `resolve.alias` 中。

### 二、配置出口 output

用于告知 webpack 如何构建编译后的文件，可以自定义输出文件的位置和名称:

```js
module.exports = {
  output: { 
    // path 必须为绝对路径
    // 输出文件路径
    path: path.resolve(__dirname, '../../dist/build'),
    // 包名称
    filename: "[name].bundle.js",
    // 或使用函数返回名(不常用)
    // filename: (chunkData) => {
    //   return chunkData.chunk.name === 'main' ? '[name].js': '[name]/[name].js';
    // },
    // 块名，公共块名(非入口)
    chunkFilename: '[name].[chunkhash].bundle.js',
    // 打包生成的 index.html 文件里面引用资源的前缀
    // 也为发布到线上资源的 URL 前缀
    // 使用的是相对路径，默认为 ''
    publicPath: '/', 
  }
}
```

在 webpack4 开发模式下，会默认启动 `output.pathinfo` ，它会输出一些额外的注释信息，对项目调试非常有用，尤其是使用 eval devtool 时。

`filename` ：`[name]` 为 entry 配置的 `key`，除此之外，还可以是 `[id]` （内部块 id ）、 `[hash]`、`[contenthash]` 等。

#### 1. 浏览器缓存与 hash 值

对于我们开发的每一个应用，浏览器都会对静态资源进行缓存，如果我们更新了静态资源，而没有更新静态资源名称（或路径），浏览器就可能因为缓存的问题获取不到更新的资源。在我们使用 webpack 进行打包的时候，webpack 提供了 hash 的概念，所以我们可以使用 hash 来打包。

在定义包名称（例如 `chunkFilename` 、 `filename`），我们一般会用到哈希值，不同的哈希值使用的场景不同：

##### hash

build-specific， 哈希值对应每一次构建（ `Compilation` ），即每次编译都不同，即使文件内容都没有改变，并且所有的资源都共享这一个哈希值，此时，浏览器缓存就没有用了，**可以用在开发环境，生产环境不适用。**

##### chunkhash

chunk-specific， 哈希值对应于 webpack 每个入口点，每个入口都有自己的哈希值。如果在某一入口文件创建的关系依赖图上存在文件内容发生了变化，那么相应入口文件的 `chunkhash` 才会发生变化，**适用于生产环境**

##### contenthash

content-specific，根据包内容计算出的哈希值，只要包内容不变，`contenthash` 就不变，**适用于生产环境**

但我们会发现，有时内容没有变更，打包时 ` [contenthash]` 反而变更了的问题，

webpack 也允许哈希的切片。如果你写 `[hash:8]` ，那么它会获取哈希值的前 8 位。

##### 注意：

- 尽量在生产环境使用哈希
- 按需加载的块不受 `filename` 影响，受 `chunkFilename` 影响
- 使用 `hash/chunkhash/contenthash` 一般会配合 `html-webpack-plugin` （创建 html ，并捆绑相应的打包文件） 、`clean-webpack-plugin` （清除原有打包文件） 一起使用。

#### 2. 打包成库

当使用 webapck 构建一个可以被其它模块引用的库时：

```js
module.exports = {
  output: { 
    // path 必须为绝对路径
    // 输出文件路径
    path: path.resolve(__dirname, '../../dist/build'),
    // 包名称
    filename: "[name].bundle.js",
    // 块名，公共块名(非入口)
    chunkFilename: '[name].[chunkhash].bundle.js',
    // 打包生成的 index.html 文件里面引用资源的前缀
    // 也为发布到线上资源的 URL 前缀
    // 使用的是相对路径，默认为 ''
    publicPath: '/', 
    // 一旦设置后该 bundle 将被处理为 library
    library: 'webpackNumbers',
    // export 的 library 的规范，有支持 var, this, commonjs,commonjs2,amd,umd
    libraryTarget: 'umd',
  }
}
```

### 三、配置模式 mode（webpack4）

设置 `mode` ，可以让 webpack 自动调起相应的内置优化。

```js
module.exports = {
  // 可以是 none、development、production
  // 默认为 production
  mode: 'production'
}
```

或在命令行里配置：

```shell
"build:prod": "webpack --config config/webpack.prod.config.js --mode production"
```

在设置了 `mode` 之后，webpack4 会同步配置 `process.env.NODE_ENV` 为 `development` 或 `production` 。

webpack4 最引人注目的主要是：

- 减小编译时间

  打包时间减小了超过 60%

- 零配置

  我们可以在没有任何配置文件的情况下将 webpack 用于各种项目

webpack4 支持零配置使用，这里的零配置就是指，`mode` 以及 `entry` （默认为 `src/index.js`）都可以通过入口文件指定，并且 webpack4 针对对不同的 `mode` 内置相应的优化策略。

#### 1. production

配置：

```js
// webpack.prod.config.js
module.exports = {
  mode: 'production',
}
```

相当于默认内置了：

```js
// webpack.prod.config.js
module.exports = {
  performance: {
    // 性能设置,文件打包过大时，会报警告
    hints: 'warning'
  },
  output: {
    // 打包时，在包中不包含所属模块的信息的注释
    pathinfo: false
  },
  optimization: {
    // 不使用可读的模块标识符进行调试
    namedModules: false,
    // 不使用可读的块标识符进行调试
    namedChunks: false,
    // 设置 process.env.NODE_ENV 为 production
    nodeEnv: 'production',
    // 标记块是否是其它块的子集
    // 控制加载块的大小（加载较大块时，不加载其子集）
    flagIncludedChunks: true,
    // 标记模块的加载顺序，使初始包更小
    occurrenceOrder: true,
    // 启用副作用
    sideEffects: true,
    // 确定每个模块的使用导出，
    // 不会为未使用的导出生成导出
    // 最小化的消除死代码
    // optimization.usedExports 收集的信息将被其他优化或代码生成所使用
    usedExports: true,
    // 查找模块图中可以安全的连接到其它模块的片段
    concatenateModules: true,
    // SplitChunksPlugin 配置项
    splitChunks: {
      // 默认 webpack4 只会对按需加载的代码做分割
      chunks: 'async',
      // 表示在压缩前的最小模块大小,默认值是30kb
      minSize: 30000,
      minRemainingSize: 0,
      // 旨在与HTTP/2和长期缓存一起使用 
      // 它增加了请求数量以实现更好的缓存
      // 它还可以用于减小文件大小，以加快重建速度。
      maxSize: 0,
      // 分割一个模块之前必须共享的最小块数
      minChunks: 1,
      // 按需加载时的最大并行请求数
      maxAsyncRequests: 6,
      // 入口的最大并行请求数
      maxInitialRequests: 4,
      // 界定符
      automaticNameDelimiter: '~',
      // 块名最大字符数
      automaticNameMaxLength: 30,
      cacheGroups: { // 缓存组
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    },
    // 当打包时，遇到错误编译，将不会把打包文件输出
    // 确保 webpack 不会输入任何错误的包
    noEmitOnErrors: true,
    checkWasmTypes: true,
    // 使用 optimization.minimizer || TerserPlugin 来最小化包
    minimize: true,
  },
  plugins: [
    // 使用 terser 来优化 JavaScript
    new TerserPlugin(/* ... */),
    // 定义环境变量
    new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("production") }),
    // 预编译所有模块到一个闭包中，提升代码在浏览器中的执行速度
    new webpack.optimize.ModuleConcatenationPlugin(),
    // 在编译出现错误时，使用 NoEmitOnErrorsPlugin 来跳过输出阶段。
    // 这样可以确保输出资源不会包含错误
    new webpack.NoEmitOnErrorsPlugin()
  ]
}
```

#### 2. development

配置：

```js
// webpack.dev.config.js
module.exports = {
  mode: 'development',
}
```

相当于默认内置了：

```js
// webpack.dev.config.js
module.exports = {
  devtool: 'eval',
  cache: true,
  performance: {
    // 性能设置,文件打包过大时，不报错和警告，只做提示
    hints: false
  },
  output: {
    // 打包时，在包中包含所属模块的信息的注释
    pathinfo: true
  },
  optimization: {
    // 使用可读的模块标识符进行调试
    namedModules: true,
    // 使用可读的块标识符进行调试
    namedChunks: true,
    // 设置 process.env.NODE_ENV 为 development
    nodeEnv: 'development',
    // 不标记块是否是其它块的子集
    flagIncludedChunks: false,
    // 不标记模块的加载顺序
    occurrenceOrder: false,
    // 不启用副作用
    sideEffects: false,
    usedExports: false,
    concatenateModules: false,
    splitChunks: {
      hidePathInfo: false,
      minSize: 10000,
      maxAsyncRequests: Infinity,
      maxInitialRequests: Infinity,
    },
    // 当打包时，遇到错误编译，仍把打包文件输出
    noEmitOnErrors: false,
    checkWasmTypes: false,
    // 不使用 optimization.minimizer || TerserPlugin 来最小化包
    minimize: false,
    removeAvailableModules: false
  },
  plugins: [
    // 当启用 HMR 时，使用该插件会显示模块的相对路径
    // 建议用于开发环境
    new webpack.NamedModulesPlugin(),
    // webpack 内部维护了一个自增的 id，每个 chunk 都有一个 id。
    // 所以当增加 entry 或者其他类型 chunk 的时候，id 就会变化，
    // 导致内容没有变化的 chunk 的 id 也发生了变化
    // NamedChunksPlugin 将内部 chunk id 映射成一个字符串标识符（模块的相对路径）
    // 这样 chunk id 就稳定了下来
    new webpack.NamedChunksPlugin(),
    // 定义环境变量
    new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("development") }),
  ]
}
```

#### 3. none

不进行任何默认优化选项。

配置：

```js
// webpack.com.config.js
module.exports = {
  mode: 'none',
}
```

相当于默认内置了：

```js
// webpack.com.config.js
module.exports = {
  performance: {
   // 性能设置,文件打包过大时，不报错和警告，只做提示
   hints: false
  },
  optimization: {
    // 不标记块是否是其它块的子集
    flagIncludedChunks: false,
    // 不标记模块的加载顺序
    occurrenceOrder: false,
    // 不启用副作用
    sideEffects: false,
    usedExports: false,
    concatenateModules: false,
    splitChunks: {
      hidePathInfo: false,
      minSize: 10000,
      maxAsyncRequests: Infinity,
      maxInitialRequests: Infinity,
    },
    // 当打包时，遇到错误编译，仍把打包文件输出
    noEmitOnErrors: false,
    checkWasmTypes: false,
    // 不使用 optimization.minimizer || TerserPlugin 来最小化包
    minimize: false,
  },
  plugins: []
}
```

#### 4. production、 development、none 总结

![](http://resource.muyiy.cn/image/20200102230808.png)
![](http://resource.muyiy.cn/image/20200102230825.png)
![](http://resource.muyiy.cn/image/20200102230835.png)
![](http://resource.muyiy.cn/image/20200102230846.png)

**production 模式下给你更好的用户体验：**

- 较小的输出包体积
- 浏览器中更快的代码执行速度
- 忽略开发中的代码
- 不公开源代码或文件路径
- 易于使用的输出资产

**development 模式会给予你最好的开发体验：**

- 浏览器调试工具
- 快速增量编译可加快开发周期
- 运行时提供有用的错误消息

尽管 webpack4 在尽力让零配置做到更多，但仍然是有限度的，大多数情况下还是需要一个配置文件。我们可以在项目的初期使用零配置，在后期业务复杂的时候再配置。

#### 5. 环境变量 process.env.NODE_ENV

第三方框架或库，以及我们的业务代码，都会针对不同的环境配置，执行不同的逻辑代码，例如：

我们可以通过以下方式定义环境变量：

**方法一：webpack4 中 mode: 'production' 已经默认配置了 process.env.NODE_ENV = 'production' ，所以 webapck4 可以不定义**

尽管 webpack4 中定义 `mode` 会自动配置 `process.env.NODE_ENV` ，那么我们就不需要手动配置环境变量了吗？

其实不然，`mode` 只可以定义成 `development` 或 `production` ，而在项目中，我们不仅仅只有开发或生产环境，很多情况下需要配置不同的环境（例如测试环境），此时我们就需要手动配置其它环境变量（例如测试环境，就需要定义 `process.env.NODE_ENV` 为 `'test'` ），你可以采取以下方式：

**方法二：webpack.DefinePlugin**

```js
// webpack编译过程中设置全局变量process.env
new webpack.DefinePlugin({
  'process.env': require('../config/dev.env.js')
}
```

`config/prod.env.js` ：

```js
module.exports ={
  // 或  '"production"' ，环境变量的值需要是一个由双引号包裹的字符串
  NODE_ENV: JSON.stringify('production') 
}
```

**方法三：webpack 命令时， NODE_ENV=development**

在 window 中配置 `NODE_ENV=production` 可能会卡住，所以使用 cross-env：

```js
cross-env NODE_ENV=production webpack --config webpack.config.prod.js
```

**方法四：使用 `new webpack.EnvironmentPlugin(['NODE_ENV'])`**

`EnvironmentPlugin` 是一个通过 `webpack.DefinePlugin` 来设置 `process.env` 环境变量的快捷方式。

```js
new webpack.EnvironmentPlugin({
  NODE_ENV: 'production',
});
```

注意：上面其实是给 `NODE_ENV` 设置一个默认值 `'production'` ，如果其它地方有定义 `process.env.NODE_ENV` ，则该默认值无效。

### 四、配置解析策略  resolve

自定义寻找依赖模块时的策略（例如 `import _ from 'lodash'`）:

```js
module.exports = {
  resolve: {
    // 设置模块导入规则，import/require时会直接在这些目录找文件
    // 可以指明存放第三方模块的绝对路径，以减少寻找，
    // 默认 node_modules
    modules: [path.resolve(`${project}/components`), 'node_modules'],
    // import导入时省略后缀
    // 注意：尽可能的减少后缀尝试的可能性
    extensions: ['.js', '.jsx', '.react.js', '.css', '.json'],
    // import导入时别名，减少耗时的递归解析操作
    alias: {
      '@components': path.resolve(`${project}/components`),
      '@style': path.resolve('asset/style'),
    },
    // 很多第三方库会针对不同的环境提供几份代码
    // webpack 会根据 mainFields 的配置去决定优先采用那份代码
    // 它会根据 webpack 配置中指定的 target 不同，默认值也会有所不同
    mainFields: ['browser', 'module', 'main'],
  },
}
```



### 五、配置解析和转换文件的策略 module

决定如何处理项目中不同类型的模块，通常是配置 module.rules 里的 Loader:

```js
module.exports = {
  module: {
    // 指明 webpack 不去解析某些内容，该方式有助于提升 webpack 的构建性能
    noParse: /jquery/,
    rules: [
      {
        // 这里编译 js、jsx
        // 注意：如果项目源码中没有 jsx 文件就不要写 /\.jsx?$/，提升正则表达式性能
        test: /\.(js|jsx)$/,
        // 指定要用什么 loader 及其相关 loader 配置
        use: {
          loader: "babel-loader",
          options: {
            // babel-loader 支持缓存转换出的结果，通过 cacheDirectory 选项开启
            // 使用 cacheDirectory 选项将 babel-loader 的速度提高2倍
      		cacheDirectory: true,
      		// Save disk space when time isn't as important
      		cacheCompression: true,
      		compact: true,     
          }
        },
        // 排除 node_modules 目录下的文件
        // node_modules 目录下的文件都是采用的 ES5 语法，没必要再通过 Babel 去转换
        exclude: /node_modules/
        // 也可以配置 include：需要引入的文件
      }
    ]
  }
}
```

#### 1. noParse

指明 webpack 不去解析某些内容，该方式有助于提升 webpack 的构建性能。

#### 2. rules

常见的 loader 有：

- `babel-loader`：解析 `.js` 和 `.jsx` 文件

  ```js
  // 配置 .babelrc
  {
    "presets": [
      [
        "@babel/preset-env",
      ],
      "@babel/preset-react"
    ],
    "plugins": [
      [
        "@babel/plugin-proposal-class-properties",
        {
          "loose": true
        }
      ],
      [
        "@babel/plugin-transform-runtime",
        {
          "absoluteRuntime": false,
          "corejs": false,
          "helpers": true,
          "regenerator": true,
          "useESModules": false
        }
      ],
    ]
  }
  ```

- `tsx-loader`：处理 ts 文件

- `less-loader`：处理 less 文件，并将其编译为 css

- `sass-loader`：处理 sass、scss 文件，并将其编译为 css

- `postcss-loader`：

  ```js
  // postcss.config.js
  module.exports = { // 解析CSS文件并且添加浏览器前缀到 CSS 内容里
  	plugins: [require('autoprefixer')],
  };
  ```

- `css-loader`：处理 css 文件

- `style-loader`：将 css 注入到 DOM

- `file-loader`：将文件上的` import`  /  `require` 解析为 `url`，并将该文件输出到输出目录中

- `url-loader`：用于将文件转换成 base64 uri 的 webpack 加载程序

- `html-loader`：将 HTML 导出为字符串， 当编译器要求时，将 HTML 最小化

更多 loaders 可查看 [LOADERS](https://webpack.js.org/loaders/) 。

### 六、配置优化 optimization（webpack4）

webapck4 会根据你所选择的 `mode` 进行优化，你可以手动配置，它将会覆盖自动优化，详细配置请见 [Optimization](https://webpack.js.org/configuration/optimization/#optimizationminimizer) 。

主要涉及两方面的优化：

- 最小化包
- 拆包

#### 1. 最小化包

- 使用 `optimization.removeAvailableModules` 删除已可用模块
- 使用 `optimization.removeEmptyChunks` 删除空模块
- 使用 `optimization.occurrenceOrder` 标记模块的加载顺序，使初始包更小
- 使用 `optimization.providedExports` 、 `optimization.usedExports` 、`concatenateModules` 、`optimization.sideEffects` 删除死代码
- 使用 `optimization.splitChunks` 提取公共包
- 使用 `optimization.minimizer` || `TerserPlugin` 来最小化包

#### 2. 拆包

当包过大时，如果我们更新一小部分的包内容，那么整个包都需要重新加载，如果我们把这个包拆分，那么我们仅仅需要重新加载发生内容变更的包，而不是所有包，有效的利用了缓存。

##### 拆分 node_modules

很多情况下，我们不需要手动拆分包，可以使用 `optimization.splitChunks` ：

```js
const path = require('path');
module.exports = {
  entry: path.resolve(__dirname, 'src/index.js'),
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
  },
  optimization: {
    splitChunks: {
      // 对所有的包进行拆分
      chunks: 'all',
    },
  },
};
```

我们不必制定拆包策略，`chunks: all` 会自动将 `node_modules` 中的所有内容放入一个名为 `vendors〜main.js` 的文件中。

##### 拆分业务代码

```js
module.exports = {
  entry: {
    main: path.resolve(__dirname, 'src/index.js'),
    ProductList: path.resolve(__dirname, 'src/ProductList/ProductList.js'),
    ProductPage: path.resolve(__dirname, 'src/ProductPage/ProductPage.js'),
    Icon: path.resolve(__dirname, 'src/Icon/Icon.js'),
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash:8].js',
  },
};
```

采用多入口的方式，当有业务代码更新时，更新相应的包即可

##### 拆分第三方库

```js
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: path.resolve(__dirname, 'src/index.js'),
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
  },
  optimization: {
    runtimeChunk: 'single',
    splitChunks: {
      chunks: 'all',
      maxInitialRequests: Infinity,
      minSize: 0,
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name(module) {
            // 获取第三方包名
            const packageName = module.context.match(/[\\/]node_modules[\\/](.*?)([\\/]|$)/)[1];

            // npm 软件包名称是 URL 安全的，但是某些服务器不喜欢@符号
            return `npm.${packageName.replace('@', '')}`;
          },
        },
      },
    },
  },
};
```

当第三方包更新时，仅更新相应的包即可。

注意，当包太多时，浏览器会发起更多的请求，并且当文件过小时，对代码压缩也有影响。

##### 动态加载

现在我们已经对包拆分的很彻底了，但以上的拆分仅仅是对浏览器缓存方面的优化，减小首屏加载时间，实际上我们也可以使用按需加载的方式来进一步拆分，减小首屏加载时间：

```js
import React, { useState, useEffect } from 'react';
import './index.scss'

function Main() {
  const [NeighborPage, setNeighborPage] = useState(null)

  useEffect(() => {
    import('../neighbor').then(({ default: component }) => {
      setNeighborPage(React.createElement(component))
    });
  }, [])

  return NeighborPage
    ? NeighborPage
    : <div>Loading...</div>;
}

export default Main
```



### 七、配置 plugin

配置 Plugin 去处理及优化其它的需求，

```js
module.exports = {
  plugins: [
    // 优化 require
    new webpack.ContextReplacementPlugin(/moment[\/\\]locale$/, /en|zh/),
    // 用于提升构建速度
    createHappyPlugin('happy-babel', [{
      loader: 'babel-loader',
      options: {
        presets: ['@babel/preset-env', "@babel/preset-react"],
        plugins: [
          ['@babel/plugin-proposal-class-properties', {
            loose: true
          }]
        ],
        // babel-loader 支持缓存转换出的结果，通过 cacheDirectory 选项开启
        cacheDirectory: true,
        // Save disk space when time isn't as important
        cacheCompression: true,
        compact: true,
      }
    }])
  ]
}
```

常用 plugins：

- `html-webpack-plugin`：生成 html 文件，并将包添加到 html 中
- `webpack-parallel-uglify-plugin`：压缩 js（多进程并行处理压缩）
- `happypack`：多线程loader，用于提升构建速度
- `hard-source-webpack-plugin`：为模块提供中间缓存步骤，显著提高打包速度
- `webpack-merge`：合并 webpack 配置
- `mini-css-extract-plugin`：抽离 css
- `optimize-css-assets-webpack-plugin`：压缩 css
- `add-asset-html-webpack-plugin`：将 JavaScript 或 CSS 资产添加到 html-webpack-plugin 生成的 HTML 中

更多插件可见：[plugins](https://webpack.js.org/plugins/)

### 八、配置devtool：source map

配置 webpack 如何生成 Source Map，用来增强调试过程。不同的值会明显影响到构建(build)和重新构建(rebuild)的速度：

生产环境：默认为 `null` ，一般不设置（ `none` ）或 `nosources-source-map`

开发环境：默认为 `eval` ，一般设置为 `eval` 、 `cheap-eval-source-map` 、`cheap-module-eval-source-map` 

**策略为：**

- **使用 cheap 模式可以大幅提高 souremap 生成的效率。** 没有列信息（会映射到转换后的代码，而不是映射到原始代码），通常我们调试并不关心列信息，而且就算 source map 没有列，有些浏览器引擎（例如 v8） 也会给出列信息。
- **使用 eval 方式可大幅提高持续构建效率。**参考官方文档提供的速度对比表格可以看到 eval 模式的编译速度很快。
- **使用 module 可支持 babel 这种预编译工具**（在 webpack 里做为 loader 使用）。

如果默认的 webpack `minimizer` 已经被重定义(例如 `terser-webpack-plugin` )，你必须提供 `sourceMap：true` 选项来启用 source map 支持。

更多可查看：[devtool](https://webpack.js.org/configuration/devtool/#devtool)

###九、配置性能 performance

当打包是出现超过特定文件限制的资产和入口点，`performance` 控制 webpack 如何通知：

```js
module.exports = {
  // 配置如何显示性能提示
  performance: {
    // 可选 warning、error、false
    // false：性能设置,文件打包过大时，不报错和警告，只做提示
    // warning：显示警告，建议用在开发环境
    // error：显示错误，建议用在生产环境，防止部署太大的生产包，从而影响网页性能
    hints: false
  }
}
```

### 十、配置其它 

#### 1. watch 与 watchOptions

##### watch

监视文件更新，并在文件更新时重新编译：

```js
module.export = {
  // 启用监听模式
  watch: true,
}
```

在 `webpack-dev-server` 和 `webpack-dev-middleware` 中，默认启用了监视模式。

或者我们可以在命令行里启动监听（ `--watch` ）：

```shell
webpack --watch --config webpack.config.dev.js
```

##### watchOptions

```js
module.export = {
  watch: true,
  // 自定义监视模式
  watchOptions: {
    // 排除监听
    ignored: /node_modules/,
    // 监听到变化发生后，延迟 300ms（默认） 再去执行动作，
    // 防止文件更新太快导致重新编译频率太高
    aggregateTimeout: 300,
    // 判断文件是否发生变化是通过不停的去询问系统指定文件有没有变化实现的
    // 默认 1000ms 询问一次
    poll: 1000
  }
}
```

#### 2. externals 

排除打包时的依赖项，不纳入打包范围内，例如你项目中使用了 `jquery` ，并且你在 html 中引入了它，那么在打包时就不需要再把它打包进去：

```js
<script
  src="https://code.jquery.com/jquery-3.1.0.js"
  integrity="sha256-slogkvB1K3VOkzAI8QITxV3VzpOnkeNVsKvtkYLMjfk="
  crossorigin="anonymous">
</script>
```

配置：

```js
module.exports = {
  // 打包时排除 jquery 模块
  externals: {
    jquery: 'jQuery'
  }
};
```

#### 3.target

构建目标，用于为 webpack 指定一个环境：

```js
module.exports = {
  // 编译为类浏览器环境里可用（默认）
  target: 'web'
};
```

#### 4. cache

缓存生成的 webpack 模块和块以提高构建速度。在开发模式中，缓存设置为 `type: 'memory'` ，在生产模式中禁用。`cache: true` 是 `cache: {type: 'memory'}` 的别名。要禁用缓存传递 `false` ：

```js
module.exports = {
  cache: false
}
```

在内存中，缓存仅在监视模式下有用，并且我们假设你在开发中使用监视模式。 在不进行缓存的情况下，内存占用空间较小。

#### 5. name

配置的名称，用于加载多个配置：

```js
module.exports = {
  name: 'admin-app'
};
```

### 十一、总结

本文仅仅是列出一些常用的配置项，所有的配置文件架构可见：[WebpackOptions.json](https://github.com/webpack/webpack/blob/master/schemas/WebpackOptions.json)，你也可以进入 webpack 官网了解更多。 

参考：

[webpack](https://webpack.js.org/)

[The 100% correct way to split your chunks with Webpack](https://medium.com/hackernoon/the-100-correct-way-to-split-your-chunks-with-webpack-f8a9df5b7758)

[webpack 4: mode and optimization](https://medium.com/webpack/webpack-4-mode-and-optimization-5423a6bc597a)
