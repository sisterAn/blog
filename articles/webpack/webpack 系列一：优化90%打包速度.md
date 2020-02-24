### 前言

webpack 打包优化并没有什么固定的模式，一般我们常见的优化就是拆包、分块、压缩等，并不是对每一个项目都适用，针对于特定项目，需要不断调试不断优化。

对于 webpack4，建议从零开始配置，在项目初期，使用 webpack4 默认的配置。

接下来，本篇文章会列出所有适用于 webpack 优化打包速度的技术方案，并给出相应的限制，请在实际项目中分情况使用。如有任何疑问，请联系瓶子君。

### 一、分析打包速度

优化 webpack 构建速度的第一步是知道将精力集中在哪里。我们可以通过 `speed-measure-webpack-plugin` 测量你的 webpack 构建期间各个阶段花费的时间：

```js
// 分析打包时间
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");
const smp = new SpeedMeasurePlugin();
// ...
module.exports = smp.wrap(prodWebpackConfig)
```

特定的项目，都有自己特定的性能构建瓶颈，下面我们对打包的每一个环节进行优化。

### 二、分析影响打包速度环节

在「窥探原理：手写一个 JavaScript 打包器」中，我们已经介绍过，打包就是从入口文件开始将所有的依赖模块打包到一个文件中的过程，当然，在打包过程中涉及各种编译、优化过程。

打包过程中，常见影响构建速度的地方有哪些喃？

#### 1. 开始打包，我们需要获取所有的依赖模块

搜索所有的依赖项，这需要占用一定的时间，即搜索时间，那么我们就确定了：

**我们需要优化的第一个时间就是搜索时间。**

#### 2. 解析所有的依赖模块（解析成浏览器可运行的代码）

webpack 根据我们配置的 loader 解析相应的文件。日常开发中我们需要使用 loader 对 js ，css ，图片，字体等文件做转换操作，并且转换的文件数据量也是非常大。由于 js 单线程的特性使得这些转换操作不能并发处理文件，而是需要一个个文件进行处理。

**我们需要优化的第二个时间就是解析时间。**

#### 3. 将所有的依赖模块打包到一个文件

将所有解析完成的代码，打包到一个文件中，为了使浏览器加载的包更新（减小白屏时间），所以 webpack 会对代码进行优化。

 JS 压缩是发布编译的最后阶段，通常 webpack 需要卡好一会，这是因为压缩  JS 需要先将代码解析成 AST 语法树，然后需要根据复杂的规则去分析和处理 AST，最后将 AST 还原成  JS，这个过程涉及到大量计算，因此比较耗时，打包就容易卡住。

**我们需要优化的第三个时间就是压缩时间。**

#### 4. 二次打包

当更改项目中一个小小的文件时，我们需要重新打包，所有的文件都必须要重新打包，需要花费同初次打包相同的时间，但项目中大部分文件都没有变更，尤其是第三方库。

**我们需要优化的第四个时间就是二次打包时间。**

### 三、 优化解析时间 - 开启多线程打包

运行在 Node.js 之上的 webpack 是单线程模式的，也就是说，webpack 打包只能逐个文件处理，当 webpack 需要打包大量文件时，打包时间就会比较漫长。

#### 1. thread-loader（webpack4 官方推荐）

把这个 loader 放置在其他 loader 之前， 放置在这个 loader 之后的 loader 就会在一个单独的 worker【worker pool】 池里运行，一个worker 就是一个nodeJS 进程【node.js proces】，每个单独进程处理时间上限为600ms，各个进程的数据交换也会限制在这个时间内。

thread-loader 使用起来也非常简单，只要把 thread-loader 放置在其他 loader 之前， 那 thread-loader 之后的 loader 就会在一个单独的 worker 池(worker pool)中运行。

例如：

```js
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        // 创建一个 js worker 池
        use: [ 
          'thread-loader',
          'babel-loader'
        ] 
      },
      {
        test: /\.s?css$/,
        exclude: /node_modules/,
        // 创建一个 css worker 池
        use: [
          'style-loader',
          'thread-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true,
              localIdentName: '[name]__[local]--[hash:base64:5]',
              importLoaders: 1
            }
          },
          'postcss-loader'
        ]
      }
      // ...
    ]
    // ...
  }
  // ...
}
```

注意：thread-loader 放在了 style-loader 之后，这是因为 thread-loader 没法存取文件也没法获取 webpack 的选项设置。

官方上说每个 worker 大概都要花费 600ms ，所以官方为了防止启动 worker 时的高延迟，提供了对 worker 池的优化：**预热**

```js
// ...
const threadLoader = require('thread-loader');

const jsWorkerPool = {
  // options
  
  // 产生的 worker 的数量，默认是 (cpu 核心数 - 1)
  // 当 require('os').cpus() 是 undefined 时，则为 1
  workers: 2,
  
  // 闲置时定时删除 worker 进程
  // 默认为 500ms
  // 可以设置为无穷大， 这样在监视模式(--watch)下可以保持 worker 持续存在
  poolTimeout: 2000
};

const cssWorkerPool = {
  // 一个 worker 进程中并行执行工作的数量
  // 默认为 20
  workerParallelJobs: 2,
  poolTimeout: 2000
};

threadLoader.warmup(jsWorkerPool, ['babel-loader']);
threadLoader.warmup(cssWorkerPool, ['css-loader', 'postcss-loader']);


module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'thread-loader',
            options: jsWorkerPool
          },
          'babel-loader'
        ]
      },
      {
        test: /\.s?css$/,
        exclude: /node_modules/,
        use: [
          'style-loader',
          {
            loader: 'thread-loader',
            options: cssWorkerPool
          },
          {
            loader: 'css-loader',
            options: {
              modules: true,
              localIdentName: '[name]__[local]--[hash:base64:5]',
              importLoaders: 1
            }
          },
          'postcss-loader'
        ]
      }
      // ...
    ]
    // ...
  }
  // ...
}
```

**注意：请仅在耗时的 loader 上使用。**

#### 2. HappyPack

在webpack构建过程中，实际上耗费时间大多数用在 loader 解析转换以及代码的压缩中，HappyPack 可利用多线程对文件进行打包(默认cpu核数-1)，对多核cpu利用率更高。HappyPack 可以让 Webpack 同一时间处理多个任务，发挥多核 CPU 的能力，将任务分解给多个子进程去并发的执行，子进程处理完后，再把结果发送给主进程。

happypack 的处理思路是将原有的 webpack 对 loader 的执行过程从单一进程的形式扩展多进程模式，原本的流程保持不变。使用 HappyPack 也有一些限制，它只兼容部分主流的 loader，具体可以查看官方给出的 [兼容性列表](https://github.com/amireh/happypack/wiki/Loader-Compatibility-List)。

**注意：Ahmad Amireh 推荐使用 thread-loader，并宣布将不再继续维护 happypack，所以不推荐使用它**

```js
const path = require('path')
const webpack = require("webpack");
const HappyPack = require('happypack'); // 多线程loader
// node 提供的系统操作模块
const os = require('os');
//  构造出共享进程池，根据系统的内核数量，指定线程池个数，也可以其他数量
const happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length });
const createHappyPlugin = (id, loaders) => new HappyPack({
  // 用唯一的标识符 id 来代表当前的 HappyPack 是用来处理一类特定的文件
  id: id,
  // 如何处理 .js 文件，用法和 Loader 配置中一样
  loaders: loaders,
  // 其它配置项(可选)
  // 代表共享进程池，即多个 HappyPack 实例都使用同一个共享进程池中的子进程去处理任务，以防止资源占用过多
  threadPool: happyThreadPool,
  // 是否允许 HappyPack 输出日志，默认是 true
  verbose: true
  // threads：代表开启几个子进程去处理这一类型的文件，默认是3个，类型必须是整数
});

const clientWebpackConfig = {
  // ...
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        // 把对 .js .jsx 文件的处理转交给 id 为 happy-babel 的 HappyPack 实例
        use: ["happypack/loader?id=happy-babel"],
        // 排除 node_modules 目录下的文件
        // node_modules 目录下的文件都是采用的 ES5 语法，没必要再通过 Babel 去转换
        exclude: /node_modules/,
      }
    ]
  },
  // ...
  plugins: [
    createHappyPlugin('happy-babel', [{
      loader: 'babel-loader',
      options: {
        presets: ['@babel/preset-env', "@babel/preset-react"],
        plugins: [
          ["import", { "libraryName": "antd", "style": true }],
          ['@babel/plugin-proposal-class-properties',{loose:true}]
        ],
        cacheDirectory: true,
        // Save disk space when time isn't as important
        cacheCompression: true,
        compact: true,
      }
    }]),
    // ...
  ]
}
```

**注意，当项目较小时，多线程打包反而会使打包速度变慢。**

### 四、合理利用缓存（缩短连续构建时间，增加初始构建时间）

使用 webpack 缓存的方法有几种，例如使用 `cache-loader`，`HardSourceWebpackPlugin` 或 `babel-loader` 的 `cacheDirectory` 标志。 所有这些缓存方法都有启动的开销。 重新运行期间在本地节省的时间很大，但是初始（冷）运行实际上会更慢。

如果你的项目生产版本每次都必须进行初始构建的话，缓存会增加构建时间，减慢你的速度。如果不是，那它们就会大大缩减你二次构建的时间。

#### 1. cache-loader

cache-loader 和 thread-loader 一样，使用起来也很简单，仅仅需要在一些性能开销较大的 loader 之前添加此 loader，以将结果缓存到磁盘里，显著提升二次构建速度。

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.ext$/,
        use: ['cache-loader', ...loaders],
        include: path.resolve('src'),
      },
    ],
  },
};
```

**⚠️ 请注意，保存和读取这些缓存文件会有一些时间开销，所以请只对性能开销较大的 loader 使用此 loader。**

#### 2. HardSourceWebpackPlugin

- 第一次构建将花费正常的时间
- 第二次构建将显着加快（大概提升90%的构建速度）。

```js
const HardSourceWebpackPlugin = require('hard-source-webpack-plugin')
const clientWebpackConfig = {
  // ...
  plugins: [
    new HardSourceWebpackPlugin({
      // cacheDirectory是在高速缓存写入。默认情况下，将缓存存储在node_modules下的目录中
      // 'node_modules/.cache/hard-source/[confighash]'
      cacheDirectory: path.join(__dirname, './lib/.cache/hard-source/[confighash]'),
      // configHash在启动webpack实例时转换webpack配置，
      // 并用于cacheDirectory为不同的webpack配置构建不同的缓存
      configHash: function(webpackConfig) {
        // node-object-hash on npm can be used to build this.
        return require('node-object-hash')({sort: false}).hash(webpackConfig);
      },
      // 当加载器、插件、其他构建时脚本或其他动态依赖项发生更改时，
      // hard-source需要替换缓存以确保输出正确。
      // environmentHash被用来确定这一点。如果散列与先前的构建不同，则将使用新的缓存
      environmentHash: {
        root: process.cwd(),
        directories: [],
        files: ['package-lock.json', 'yarn.lock'],
      },
      // An object. 控制来源
      info: {
        // 'none' or 'test'.
        mode: 'none',
        // 'debug', 'log', 'info', 'warn', or 'error'.
        level: 'debug',
      },
      // Clean up large, old caches automatically.
      cachePrune: {
        // Caches younger than `maxAge` are not considered for deletion. They must
        // be at least this (default: 2 days) old in milliseconds.
        maxAge: 2 * 24 * 60 * 60 * 1000,
        // All caches together must be larger than `sizeThreshold` before any
        // caches will be deleted. Together they must be at least this
        // (default: 50 MB) big in bytes.
        sizeThreshold: 50 * 1024 * 1024
      },
    }),
    new HardSourceWebpackPlugin.ExcludeModulePlugin([
      {
        test: /.*\.DS_Store/
      }
    ]),
  ]
}
```

### 五、优化压缩时间

#### 1. webpack3 

webpack3 启动打包时加上 `--optimize-minimize` ，这样 Webpack 会自动为你注入一个带有默认配置的 UglifyJSPlugin 。

或：

```js
module.exports = {
    optimization: {
        minimize: true,
    },
}
```

压缩 JavaScript 代码需要先把代码解析成用 Object 抽象表示的 AST 语法树，再去应用各种规则分析和处理 AST，导致这个过程计算量巨大，耗时非常多。但 `UglifyJsPlugin` 是单线程，所以我们可以使用  `ParallelUglifyPlugin` 。

 `ParallelUglifyPlugin` 插件实现了多线程压缩，`ParallelUglifyPlugin` 会开启多个子进程，把对多个文件的压缩工作分配给多个子进程去完成，每个子进程其实还是通过 `UglifyJS` 去压缩代码，但是变成了并行执行。 所以 `ParallelUglifyPlugin` 能更快的完成对多个文件的压缩工作。

#### 2. webpack4

webpack4 中 `webpack.optimize.UglifyJsPlugin` 已被废弃。

也不推荐使用 ParallelUglifyPlugin，项目基本处于没人维护的阶段，issue 没人处理，pr没人合并。

webpack4 默认内置使用 `terser-webpack-plugin` 插件压缩优化代码，而该插件使用 `terser` 来缩小  `JavaScript` 。

##### terser 是什么？

所谓 terser，官方给出的定义是：

> 用于 ES6+ 的 JavaScript 解析器、mangler/compressor（压缩器）工具包。

为什么 webpack 选择 terser？

> 不再维护 uglify-es ，并且 uglify-js 不支持 ES6 +。
>
> terser 是 uglify-es 的一个分支，主要保留了与 uglify-es 和 uglify-js@3 的 API 和 CLI 兼容性。

##### terser 启动多线程

使用多进程并行运行来提高构建速度。并发运行的默认数量为 `os.cpus().length - 1` 。

```js
module.exports = {
  optimization: {
    minimizer: [
      new TerserPlugin({
        parallel: true,
      }),
    ],
  },
};
```

**可以显著加快构建速度，因此强烈推荐开启多线程**

### 六、优化搜索时间- 缩小文件搜索范围 减小不必要的编译工作

webpack 打包时，会从配置的 `entry` 触发，解析入口文件的导入语句，再递归的解析，在遇到导入语句时 webpack 会做两件事情：

- 根据导入语句去寻找对应的要导入的文件。例如 `require('react')` 导入语句对应的文件是 `./node_modules/react/react.js`，`require('./util')` 对应的文件是 `./util.js`。
- 根据找到的要导入文件的后缀，使用配置中的 Loader 去处理文件。例如使用 ES6 开发的 JavaScript 文件需要使用 babel-loader 去处理。

以上两件事情虽然对于处理一个文件非常快，但是当项目大了以后文件量会变的非常多，这时候构建速度慢的问题就会暴露出来。 虽然以上两件事情无法避免，但需要尽量减少以上两件事情的发生，以提高速度。

接下来一一介绍可以优化它们的途径。

#### 1. 优化 loader 配置

使用 Loader 时可以通过 `test` 、 `include` 、 `exclude` 三个配置项来命中 Loader 要应用规则的文件

#### 2. 优化 resolve.module 配置

`resolve.modules` 用于配置 webpack 去哪些目录下寻找第三方模块，`resolve.modules` 的默认值是 `['node_modules']` ，含义是先去当前目录下的 `./node_modules` 目录下去找想找的模块，如果没找到就去上一级目录 `../node_modules` 中找，再没有就去 `../../node_modules` 中找，以此类推。

#### 3. 优化 resolve.alias 配置

`resolve.alias` 配置项通过别名来把原导入路径映射成一个新的导入路径，减少耗时的递归解析操作。

#### 4. 优化 resolve.extensions 配置

在导入语句没带文件后缀时，webpack 会根据 resolve.extension 自动带上后缀后去尝试询问文件是否存在，所以在配置 `resolve.extensions` 应尽可能注意以下几点：

- `resolve.extensions` 列表要尽可能的小，不要把项目中不可能存在的情况写到后缀尝试列表中。
- 频率出现最高的文件后缀要优先放在最前面，以做到尽快的退出寻找过程。
- 在源码中写导入语句时，要尽可能的带上后缀，从而可以避免寻找过程。

#### 5. 优化 resolve.mainFields 配置

有一些第三方模块会针对不同环境提供几分代码。 例如分别提供采用 ES5 和 ES6 的2份代码，这2份代码的位置写在 `package.json` 文件里，如下：

```js
{
  "jsnext:main": "es/index.js",// 采用 ES6 语法的代码入口文件
  "main": "lib/index.js" // 采用 ES5 语法的代码入口文件
}
```

webpack 会根据 `mainFields` 的配置去决定优先采用那份代码，`mainFields` 默认如下：

```js
mainFields: ['browser', 'main']
```

webpack 会按照数组里的顺序去 `package.json` 文件里寻找，只会使用找到的第一个。

假如你想优先采用 ES6 的那份代码，可以这样配置：

```js
mainFields: ['jsnext:main', 'browser', 'main']
```

#### 6. 优化 module.noParse 配置

`module.noParse` 配置项可以让 Webpack 忽略对部分没采用模块化的文件的递归解析处理，这样做的好处是能提高构建性能。 原因是一些库，例如 jQuery 、ChartJS， 它们庞大又没有采用模块化标准，让 Webpack 去解析这些文件耗时又没有意义。

#### 7. 详细配置

```js
// 编译代码的基础配置
module.exports = {
  // ...
  module: {
    // 项目中使用的 jquery 并没有采用模块化标准，webpack 忽略它
    noParse: /jquery/,
    rules: [
      {
        // 这里编译 js、jsx
        // 注意：如果项目源码中没有 jsx 文件就不要写 /\.jsx?$/，提升正则表达式性能
        test: /\.(js|jsx)$/,
        // babel-loader 支持缓存转换出的结果，通过 cacheDirectory 选项开启
        use: ['babel-loader?cacheDirectory'],
        // 排除 node_modules 目录下的文件
        // node_modules 目录下的文件都是采用的 ES5 语法，没必要再通过 Babel 去转换
        exclude: /node_modules/,
      },
    ]
  },
  resolve: {
    // 设置模块导入规则，import/require时会直接在这些目录找文件
    // 可以指明存放第三方模块的绝对路径，以减少寻找
    modules: [
      path.resolve(`${project}/client/components`), 
      path.resolve('h5_commonr/components'), 
      'node_modules'
    ],
    // import导入时省略后缀
    // 注意：尽可能的减少后缀尝试的可能性
    extensions: ['.js', '.jsx', '.react.js', '.css', '.json'],
    // import导入时别名，减少耗时的递归解析操作
    alias: {
      '@compontents': path.resolve(`${project}/compontents`),
    }
  },
};
```

以上就是所有和缩小文件搜索范围相关的构建性能优化了，在根据自己项目的需要去按照以上方法改造后，你的构建速度一定会有所提升。
