配置 webpack 其实很简单，基础的构建可以按照以下方式来配置 webpack：

- 想让源文件加入到构建流程中去被 Webpack 控制，配置 `entry`。
- 想自定义输出文件的位置和名称，配置 `output`。
- 想自定义寻找依赖模块时的策略，配置 `resolve`。
- 想自定义解析和转换文件的策略，配置 `module`，通常是配置 `module.rules` 里的 Loader。
- 其它的大部分需求可能要通过 Plugin 去实现，配置 `plugin`。

基础的配置，这里就不再做过多的介绍，在文章结尾会给出完整配置：

当使用 webpack 进行打包，一般会出现两方面的问题：

- 打包速度过慢 => 打包时间过长 => 优化开发体验
- 包体积太大  => 影响用户体验 => 优化输出质量

## 一、优化开发体验

### 1. 优化构建速度

#### - 缩小文件搜索范围

webpack 打包时，会从配置的 `entry` 触发，解析入口文件的导入语句，再递归的解析，在遇到导入语句时 Webpack 会做两件事情：

- 根据导入语句去寻找对应的要导入的文件。例如 `require('react')` 导入语句对应的文件是 `./node_modules/react/react.js`，`require('./util')` 对应的文件是 `./util.js`。
- 根据找到的要导入文件的后缀，使用配置中的 Loader 去处理文件。例如使用 ES6 开发的 JavaScript 文件需要使用 babel-loader 去处理。

以上两件事情虽然对于处理一个文件非常快，但是当项目大了以后文件量会变的非常多，这时候构建速度慢的问题就会暴露出来。 虽然以上两件事情无法避免，但需要尽量减少以上两件事情的发生，以提高速度。

接下来一一介绍可以优化它们的途径。

- **优化 loader 配置**

  使用 Loader 时可以通过 `test` 、 `include` 、 `exclude` 三个配置项来命中 Loader 要应用规则的文件

- **优化 resolve.module 配置**

  `resolve.modules` 用于配置 Webpack 去哪些目录下寻找第三方模块，`resolve.modules` 的默认值是 `['node_modules']` ，含义是先去当前目录下的 `./node_modules` 目录下去找想找的模块，如果没找到就去上一级目录 `../node_modules` 中找，再没有就去 `../../node_modules` 中找，以此类推。

- **优化 resolve.alias 配置**

  `resolve.alias` 配置项通过别名来把原导入路径映射成一个新的导入路径，

- **优化 resolve.extensions 配置**

  在导入语句没带文件后缀时，Webpack 会根据 resolve.extension 自动带上后缀后去尝试询问文件是否存在，所以在配置 `resolve.extensions` 应尽可能注意以下几点：

  - `resolve.extensions` 列表要尽可能的小，不要把项目中不可能存在的情况写到后缀尝试列表中。
  - 频率出现最高的文件后缀要优先放在最前面，以做到尽快的退出寻找过程。
  - 在源码中写导入语句时，要尽可能的带上后缀，从而可以避免寻找过程。

- **优化 resolve.mainFields 配置**

  有一些第三方模块会针对不同环境提供几分代码。 例如分别提供采用 ES5 和 ES6 的2份代码，这2份代码的位置写在 `package.json` 文件里，如下：

  ```
  {
    "jsnext:main": "es/index.js",// 采用 ES6 语法的代码入口文件
    "main": "lib/index.js" // 采用 ES5 语法的代码入口文件
  }
  ```

  Webpack 会根据 `mainFields` 的配置去决定优先采用那份代码，`mainFields` 默认如下：

  ```
  mainFields: ['browser', 'main']
  ```

  Webpack 会按照数组里的顺序去`package.json` 文件里寻找，只会使用找到的第一个。

  假如你想优先采用 ES6 的那份代码，可以这样配置：

  ```
  mainFields: ['jsnext:main', 'browser', 'main']
  ```

- **优化 module.noParse 配置**

  `module.noParse` 配置项可以让 Webpack 忽略对部分没采用模块化的文件的递归解析处理，这样做的好处是能提高构建性能。 原因是一些库，例如 jQuery 、ChartJS， 它们庞大又没有采用模块化标准，让 Webpack 去解析这些文件耗时又没有意义。

```js
// 编译代码的基础配置
const clientWebpackConfig = {
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
      '@client': path.resolve(`${project}/client`),
      '@h5_commonr': path.resolve('h5_commonr'),
      '@noAnyDoor': path.resolve(`h5_commonr/noAnyDoor`),
      '@dllAliasMap': path.resolve(`${dllConfig.buildPath}/dllAliasMap`),
      '@utils': path.resolve('h5_commonr/utils/importDll'),
      '@importDll': path.resolve('h5_commonr/utils/importDll'),
      '@swiper': path.resolve('node_modules/swiper/dist/js/swiper.js')
    }
  },
};
module.exports = {
  client: clientWebpackConfig
}
```

以上就是所有和缩小文件搜索范围相关的构建性能优化了，在根据自己项目的需要去按照以上方法改造后，你的构建速度一定会有所提升。

#### - 使用 HardSourceWebpackPlugin

`HardSourceWebpackPlugin` 是 webpack 的插件，为模块提供中间缓存步骤。为了查看结果，你需要使用此插件运行 webpack 打包至少两次：

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



#### - 使用 DllPlugin

由于公司老项目的坑爹性，HardSourceWebpackPlugin 无法发挥作用，所以这里介绍一下 DllPlugin 的配置。

使用过 windows 电脑的人一般会看到 `.dll` 为后缀的文件，这些文件称为 **动态链接库** ，在 webpack 打包时，我们也可以构建项目的动态链接库，将项目中的网页基础模块、公共模块抽离出来，打包到一个个动态链接库中，当 webpack 打包时，需要导入的模块存在动态链接库中时，这个模块就不需要再被打包，直接去动态链接库中取就可以。 **注意：所有的动态链接库需要被加载** 。

webpack 已经内置了对动态链接库的支持，主要通过两个内置的插件接入：

- DllPlugin 插件：用于打包出一个个单独的动态链接库文件。
- DllReferencePlugin 插件：用于在主要配置文件中去引入 DllPlugin 插件打包好的动态链接库文件。

**步骤一：配置 webpack.dll.conf.js，构建出动态链接库文件：**

```js
/**
 * 打包Dll的Webpack配置文件
 *
 */
const path = require('path');
const CleanWebpackPlugin = require('clean-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const webpack = require('webpack');
const merge = require('webpack-merge');

// 编译代码的基础配置
const baseWebpackConfig = require('./webpack.base.conf');

// dll 配置文件（在下面）
const config = require('../../config');

// 这里把 styleLoaders 配置也单独独立出去了
const utils = require('../tools/utils');

const dlls = config.dlls;
// dll 版本号
const dllVersion = dlls.version; 
// dll 默认配置（你也可以配置可控）
const dllConfig = dlls.dllPlugin.defaults;
// dll 入口文件
const dllEntry = dlls.dllPlugin.entry;
// dll 包出口路径
const outputPath = path.resolve(dllConfig.buildPath);
// dll 生成映射表json文件地址
const outputPathMap = path.resolve(dllConfig.buildPath, '[name].json');

const isDev = process.env.NODE_ENV === 'development';


const plugins = [
  // 接入 DllPlugin
  new webpack.DllPlugin({
    // 动态链接库的全局变量名称，需要和 output.library 中保持一致
    // 该字段的值也就是输出的 json 文件 中 name 字段的值
    name: '[name]', // json文件名
    // 描述动态链接库的 json 文件输出时的文件名称
    path: outputPathMap //生成映射表json文件地址
  }),
  // 删除文件
  new CleanWebpackPlugin([outputPath], {
    root: process.cwd(), // 根目录
    verbose: true, // 开启在控制台输出信息
    dry: false // 启用删除文件
  }),
]

if (isDev) {
  // 将webpack生成的映射表中的数字id，替换为路径id  1---->./nodu_module/react/dist/react.js
  plugins.push(new webpack.NamedModulesPlugin());
} else {
  // 抽离css
  plugins.push(new ExtractTextPlugin({
    filename: `pa_${dllVersion}` + '.dll.css',
    disable: false,
    allChunks: true,
  }));
}

// module.exports
module.exports = merge(baseWebpackConfig.client, {
  // 上下文
  context: process.cwd(),
  // 执行入口文件
  entry: dllEntry(),
  // 调试模式
  devtool: isDev ? 'eval' : false,
  // 打包文件输出
  output: {
    // 输出的动态链接库的文件名称，[name]_${dllVersion} 代表当前动态链接库的名称，
    filename: `[name]_${dllVersion}.dll.js`,
    // 输出的文件都放到 outputPath 目录下
    path: outputPath,
    // 存放动态链接库的全局变量名称
    library: '[name]',
  },
  // loaders
  module: {
    rules: utils.styleLoaders({
      extract: !isDev
    })
  },
  // webpack插件
  plugins: plugins,
  // 性能提示
  performance: {
    hints: false,
  },
})

// 打包完，生成 dllAliasMap.json
process.on('exit', (err) => {
  generateDllMap(outputPath, err);
});
```

**注意：在 `webpack.dll.conf.js` 文件中，DllPlugin 中的 name 参数必须和 output.library 中保持一致。 原因在于 DllPlugin 中的 name 参数会影响输出的 json 文件中 name 字段的值， 而在 `webpack.config.js` 文件中 DllReferencePlugin 会去 json 文件读取 name 字段的值，并把值的内容作为在从全局变量中获取动态链接库中内容时的全局变量名。**

在这里，我把配置单独放在配置文件 `config/dll.conf.js` 中：

```js
const path = require('path');
const utils = require('../scripts/tools/utils');
const pullAll = require('lodash/pullAll'); //数组除值
const uniq = require('lodash/uniq'); //数组去重
const glob = require('glob'); //Match files

const reactDll = {
  version: '1.0.0',
  dllPlugin: {
    defaults: {
      exclude: [

      ],
      include: [
        "babel-polyfill",
        'react',
        'react-dom',
        'react-redux',
        'redux',
        'react-router',
        'better-scroll'
      ],
      // 针对开发本地调试用devPath，针对各种环境打包时用buildPath
      devPath: 'h5_commonr/@react_dll/dev_dll',
      buildPath: process.env.NODE_ENV === 'development' ? 'h5_commonr/@react_dll/dev_dll' : 'h5_commonr/@react_dll/prd_dll',
    },

    entry() {
      let dependencyNames = [];
      const exclude = reactDll.dllPlugin.defaults.exclude;
      const include = reactDll.dllPlugin.defaults.include;
      const includeDependencies = uniq([...include, ...dependencyNames]);
      return {
        paReactDll: pullAll(includeDependencies, exclude),
        paChartDll: ['highcharts'],
        paSwiperDll: ['@swiper']
      };
    },
  },
};

module.exports = reactDll;
```

style 配置文件 tools/utils.js：

```js
const path = require('path')
const ExtractTextPlugin = require('extract-text-webpack-plugin')

exports.cssLoaders = (options = {}) => {
  // generate loader string to be used with extract text plugin
  const generateLoaders = loaders => {
    const sourceLoader = loaders.map(loader => {
      let extraParamChar
      if (/\?/.test(loader)) {
        loader = loader.replace(/\?/, '-loader?')
        extraParamChar = '&'
      } else {
        loader = loader + '-loader'
        extraParamChar = '?'
      }
      return loader + (options.sourceMap ? extraParamChar + 'sourceMap' : '')
    }).join('!')

    // Extract CSS when that option is specified
    if (options.extract) {
      if (options.publicPath) {
        return ExtractTextPlugin.extract({
          use: sourceLoader,
          fallback: 'style-loader',
          publicPath: options.publicPath
        })
      } else {
        return ExtractTextPlugin.extract({
          use: sourceLoader,
          fallback: 'style-loader',
        })
      }
    } else {
      return ['style-loader', sourceLoader].join('!')
    }
  }

  return {
    css: generateLoaders(['css', 'postcss']),
    postcss: generateLoaders(['css']),
    less: generateLoaders(['css', 'less']),
    sass: generateLoaders(['css', 'postcss', 'sass?indentedSyntax']),
    scss: generateLoaders(['css', 'postcss', 'sass']),
    stylus: generateLoaders(['css', 'stylus']),
    styl: generateLoaders(['css', 'stylus'])
  }
}

// Generate loaders for standalone style files
exports.styleLoaders = options => {
  const output = []
  const loaders = exports.cssLoaders(options)

  for (const extension of Object.keys(loaders)) {
    const loader = loaders[extension]
    output.push({
      test: new RegExp('\\.' + extension + '$'),
      loader: loader
    })
  }
  return output
}
```

**步骤二：使用动态链接库文件**

**webapck 打生产或测试包时，引入 DllPlugin 插件打包好的动态链接库文件**

```js
// 引入所有的 dll 动态链接库
const manifests = glob.sync(path.resolve(`${dllPath}/pa*Dll.json`));
manifests.forEach(item => {
  plugins.push(new webpack.DllReferencePlugin({
    context: process.cwd(),
    manifest: item,
  }))
})
// 对 js
glob.sync(`${dllConfig.buildPath}/paReactDll*.dll.js`).forEach((dllPath) => {
  plugins.push(
    new AddAssetHtmlPlugin({
      filepath: dllPath,
      includeSourcemap: false,
      publicPath: './js',
      context: process.cwd(),
      outputPath: 'js',
      typeOfAsset: 'js'
    })
  );
});

// 对 css
glob.sync(`${dllConfig.buildPath}` + '/*.dll.css').forEach((dllPath) => {
  plugins.push(
    new AddAssetHtmlPlugin({
      filepath: dllPath,
      includeSourcemap: false,
      publicPath: './stylesheet',
      context: process.cwd(),
      outputPath: 'stylesheet',
      typeOfAsset: 'css'
    })
  );
});

module.exports = merge(clientWebpackConfig, {
  // ...
  plugins: plugins
})
```

**步骤三：执行构建，并在页面中被引入**

在配置好以上的 webpack 配置文件后，需要执行构建出所有配置好的动态链接库（ `webpack --display-chunks --color  --progress --config scripts/webpack/webpack.dll.conf.js` ），因为 webpack 生产或测试配置文件中定义的 DllReferencePlugin 依赖这些文件。

你可以在 html 中引入，也可以在 router 中引入：

Html:

```js
<html>
<head>
  <meta charset="UTF-8">
</head>
<body>
<div id="app"></div>
<!--导入依赖的动态链接库文件-->
<script src="./dist/XX.dll.js"></script>
<script src="./dist/XXX.dll.js"></script>
<!--导入执行入口文件-->
<script src="./dist/main.js"></script>
</body>
</html>
```

或 router 中：

```js
module.exports = {
  path: '/',
  component: require('../app/App'),
  childRoutes: [
    {
      path: 'ProductDetailPage',
      getComponent(location, cb) {
        require.ensure([], (require) => {
          Promise.all([importDll('paChartDll')]).then(function() {
            cb(null, require('../pages/ProductDetailPage'));
          })
        }, 'ProductDetailPage');
      },
      onEnter: () => {
        setTitle({'title': productName, barBgColor: backgroundColor, textColor: '#fff'});
        showNavBar();
      }
    }
  ],
}
```

#### - 使用 HappyPack

运行在 Node.js 之上的 Webpack 是单线程模式的，也就是说，webpack 打包只能逐个文件处理，当 webpack 需要打包大量文件时，打包时间就会比较漫长。

HappyPack 可利用多线程对文件进行打包(默认cpu核数-1)，对多核cpu利用率更高。HappyPack 可以让 Webpack 同一时间处理多个任务，发挥多核 CPU 的能力，将任务分解给多个子进程去并发的执行，子进程处理完后，再把结果发送给主进程。

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

**注意，项目较小时，多线程打包反而会使打包速度变慢。**

#### - 使用 ParallelUglifyPlugin

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
        compress: {
          // 在UglifyJs删除没有用到的代码时不输出警告
          warnings: false,
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



### 2. 优化开发体验

在开发过程中，修改代码是最常见的事情，我们不可能每次修改代码都去重新打包，刷新浏览器，所以需要配置一套自动化的解决方案，每次修改代码，都能自动化的重新打包（或增量打包），刷新浏览器。

#### - 使用自动刷新

##### 步骤一：监听文件变更

有两种方式：

- 在配置文件 `webpack.config.js` 中设置 `watch: true`

  ```js
  // 从配置的 Entry 文件出发，递归解析出 Entry 文件所依赖的文件，
  // 把这些依赖的文件加入到监听列表
  // 而不是直接监听项目目录下的所有文件
  module.export = {
    // 只有在开启监听模式时，watchOptions 才有意义
    // 默认为 false，也就是不开启
    watch: true,
    // 监听模式运行时的参数
    // 在开启监听模式时，才有意义
    watchOptions: {
      // 不监听的文件或文件夹，支持正则匹配
      // 默认为空
      ignored: /node_modules/,
      // 在 Webpack 中监听一个文件发生变化的原理是定时的不停的去获取文件的最后编辑时间，
      // 每次都存下最新的最后编辑时间，如果发现当前获取的和最后一次保存的最后编辑时间不一致，
      // 就认为该文件发生了变化。 
      // poll 就是用于控制定时检查的周期，具体含义是每隔多少毫秒检查一次
      // 默认每隔1000毫秒询问一次
      poll: 1000,
      // 监听到文件发生变化时，webpack 并不会立刻告诉监听者，
      // 而是先缓存起来，收集一段时间的变化后，再一次性告诉监听者
      // aggregateTimeout 就是用于配置这个等待时间，
      // 目的是防止文件更新太快导致重新编译频率太高，让程序构建卡死
      // 默认为 300ms
      aggregateTimeout: 300,
      // 不监听的 node_modules 目录下的文件
      ignored: /node_modules/,
    }
  }
  ```

- 在执行启动 Webpack 命令时，带上 `--watch` 参数，完整命令是 `webpack --watch`

##### 步骤二：控制浏览器自动刷新

###### 方式一：webpack-dev-server

在使用 webpack-dev-server 模块去启动 webpack 模块时，webpack 模块的监听模式默认会被开启。 webpack 模块会在文件发生变化时告诉 webpack-dev-server 模块。

devServer 默认（默认开启 devServer.inline ）采用往开发的网页中注入代理客户端代码，通过代理客户端去刷新整个页面的原理实现网页的自动刷新。但它会向每一个 Chunk 包都注入一个代理服务器，当项目需要输出的 Chunk 有很多个时，这会导致构建缓慢，所以，另一方面：

我们可以关闭 inline 模式，只注入一个代理客户端，此时，devServer 会采用将开发的网页装进一个 iframe 中，通过刷新 iframe 去看到最新效果的原理实现网页的自动刷新。



###### 方式二：koa + webpack-dev-middleware + webpack-hot-middleware 前后端同构

https://segmentfault.com/a/1190000004883199



#### - 开启模块热替换

通过前面的介绍，项目每次更新代码，都会刷新整个网页，需要开发人员等待网页刷新，反应不灵敏，那么如何实现在不刷新整个网页的情况下做到超灵敏的实时预览。

##### 模块热替换与自动刷新

- 相同点：往网页注入一个代理服务器的方式，连接 devServer 及 网页
- 不同点：
  - 自动刷新：网页自动刷新
  - 模块热替换：模块替换机制

它同样有两种配置方式：

- `webpack-dev-server —hot`

- 在配置文件 `webpack.config.js` 中:

  ```js
  const HotModuleReplacementPlugin = require('webpack/lib/HotModuleReplacementPlugin');
  
  module.exports = {
    entry:{
      // 为每个入口都注入代理客户端
      main:['webpack-dev-server/client?http://localhost:8080/', 'webpack/hot/dev-server','./src/main.js'],
    },
    plugins: [
      // 该插件的作用就是实现模块热替换，实际上当启动时带上 `--hot` 参数，会注入该插件，生成 .hot-update.json 文件。
      new HotModuleReplacementPlugin(),
    ],
    devServer:{
      // 告诉 DevServer 要开启模块热替换模式
      hot: true,      
    }  
  };
  ```

Webpack 为了让使用者在使用了模块热替换功能时能灵活地控制老模块被替换时的逻辑，可以在源码中定义一些代码去做相应的处理。所以你需要配置入口文件：

```js
import React from 'react';
import { render } from 'react-dom';
import { AppComponent } from './AppComponent';
import './main.css';

render(<AppComponent/>, window.document.getElementById('app'));

// 只有当开启了模块热替换时 module.hot 才存在
if (module.hot) {
  // accept 函数的第一个参数指出当前文件接受哪些子模块的替换，这里表示只接受 ./AppComponent 这个子模块
  // 第2个参数用于在新的子模块加载完毕后需要执行的逻辑
  module.hot.accept(['./AppComponent'], () => {
    // 新的 AppComponent 加载成功后重新执行下组建渲染逻辑
    render(<AppComponent/>, window.document.getElementById('app'));
  });
}
```

当子模块发生更新时，更新事件会一层层往上传递，也就是从 `AppComponent.js` 文件传递到 `main.js`文件， 直到有某层的文件接受了当前变化的模块，也就是 `main.js` 文件中定义的 `module.hot.accept(['./AppComponent'], callback)`， 这时就会调用 `callback` 函数去执行自定义逻辑。如果事件一直往上抛到最外层都没有文件接受它，就会直接刷新网页。

那为什么没有地方接受过 `.css` 文件，但是修改所有的 `.css` 文件都会触发模块热替换呢？ 原因在于 `style-loader` 会注入用于接受 CSS 的代码。

> 请不要把模块热替换技术用于线上环境，它是专门为提升开发效率生的。

 关闭默认的 inline 模式手动注入代理客户端的优化方法不能用于在使用模块热替换的情况下， 原因在于模块热替换的运行依赖在每个 Chunk 中都包含代理客户端的代码。

## 二、优化输出质量

### 1. 优化首屏加载时间

#### - 区分环境

针对不同的环境，我们应对打包的要求也不一样：

- 开发环境：侧重功能的调试
- 生产环境：侧重用户的体验

同一套源码如何打包成适用不同环境的包，webpack 通过配置环境变量的值，帮我们实现了这一点：

##### 配置很简单：

**方法一：**

```js
module.exports = {
  plugins: [
    new webpack.DefinePlugin({
      // 定义 NODE_ENV 环境变量为 production
      'process.env': {
        NODE_ENV: JSON.stringify('production')
       }
    }),
  ],
};
```

注意这里使用 `JSON.stringify('production')` ，而不直接使用 `'production'` ，原因是环境变量的值需要是一个由双引号包裹的字符串，而 `JSON.stringify('production')` 的值正好等于 `'"production"'` 。

或：

**方法二：**

```js
// 使用 NODE_ENV=production webpack ... 启动
module.exports = {
  plugins: [
    new webpack.EnvironmentPlugin(['NODE_ENV'])
  ],
};
```

##### 使用也很简单：

在被打包的文件中可以测试一下：

```js
if (process.env.NODE_ENV === 'production') {
  console.log('你正在使用线上环境');
} else {
  console.log('你正在使用开发环境');
}
```

> 注意：
>
> - 只用在项目中使用到 `process` 时，webpack 才会将 process 模块的代码打包进来，没使用则不会。
>
> - 并且打包成功的源码中 
>
>   ```js
>   if (process.env.NODE_ENV === 'production') {
>     console.log('你正在使用线上环境');
>   } else {
>     console.log('你正在使用开发环境');
>   }
>   ```
>
>   被直接替换成了
>
>   ```js
>   if (true) {
>     console.log('你正在使用线上环境');
>   } else {}
>   // 这里 console.log('你正在使用开发环境') 是死代码，在 UglifyJS 压缩时被去除了
>   ```
>
>   此时访问 process 的语句被替换了而没有了，Webpack 也不会打包进 process 模块
>
> - 定义的环境变量只对 Webpack 需要处理的代码有效，而不会影响 Node.js 运行时的环境变量的值。

`process.env.NODE_ENV !== 'production'` 中的 `NODE_ENV` 和 `'production'` 两个值是社区普遍的约定，很多第三方库（React）都针对此做了环境区分的优化，我们也开始使用这条判断语句在开发时区分开发环境和线上环境。

注意：在 webpack4 中 `mode: 'production'` 已经默认配置了`process.env.NODE_ENV = 'production'`

#### - 压缩代码

用户通过浏览器访问网页（JS、CSS资源），当文件体积越大时，网页的加载时间就越长，使用的流量就越大，用户的体验就越糟糕，所以我们需要对代码进行压缩。

目前比较成熟的压缩工具有两种：

- UglifyJsPlugin：通过封装 UglifyJS 实现压缩
- ParallelUglifyPlugin：多进程并行处理压缩

他们都会分析 JavaScript 代码语法树，理解代码含义，从而能做到诸如去掉无效代码、去掉日志输出代码、缩短变量名等优化，但相对于 UglifyJsPlugin 是单线程， ParallelUglifyPlugin 插件实现了多线程压缩，ParallelUglifyPlugin 会开启多个子进程，把对多个文件的压缩工作分配给多个子进程去完成，每个子进程其实还是通过 UglifyJS 去压缩代码，但是变成了并行执行。 所以 ParallelUglifyPlugin 能更快的完成对多个文件的压缩工作。

##### 压缩 Js

###### UglifyJsPlugin

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

###### webpack4 中 `webpack.optimize.UglifyJsPlugin` 已被废弃

```js
module.exports = {
    optimization: {
        minimize: true,
    },
}
```



##### 压缩 css

仅仅需要在 `css-loader` 时，加上 `'css-loader?minimize'`

#### - CDN 加速

优化用户体验不光要压缩代码文件，还要提高网络的传输速度，通过 CDN 可以实现。

> CDN：内容分发网络，通过把资源部署到世界各地，用户在访问时按照就近原则从离用户最近的服务器获取资源，从而加速资源的获取速度。 CDN 其实是通过优化物理链路层传输过程中的网速有限、丢包等问题来提升网速的，其大致原理可以如下：

![4-9cdn-arch](https://user-images.githubusercontent.com/19721451/70135650-6e1d3e00-16c5-11ea-9c0f-8667c76a27e3.png)

因为 CDN 都有缓存，所以为了避免 CDN 缓存导致用户加载到老版本的问题，需要遵循以下规则：

- 针对 HTML 文件：不开启任何缓存，不放入 CDN
- 针对静态 JS 、CSS 、图片等文件：开启 CDN 和缓存，放入 CDN 服务器，并且给每一个文件名带入 Hash 值，避免文件重名导致访问到同名缓存废弃文件的问题
- 介于浏览器对同一时刻、同一域名的请求个数有限制的状况，请求资源过多的话，可能导致加载文件被阻塞。所以，当同一时间加载资源过多时，我们可以针对不同的文件类型放入不同的 CDN 上

所以针对以上，我们在构建中注意：

- 静态资源的导入 URL 应设置为指向 CDN 服务器的绝对路径 URL
- 静态资源的文件要加入根据内容计算出的 Hash 值，防止被缓存
- 不同资源放入不同域名的 CDN 服务器上，防止资源并并行加载导致的阻塞

```js
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const {WebPlugin} = require('web-webpack-plugin');

module.exports = {
  // 省略 entry 配置...
  output: {
    // 给输出的 JavaScript 文件名称加上 Hash 值
    filename: '[name]_[chunkhash:8].js',
    path: path.resolve(__dirname, './dist'),
    // 指定存放 JavaScript 文件的 CDN 目录 URL
    publicPath: '//js.cdn.com/id/',
  },
  module: {
    rules: [
      {
        // 增加对 CSS 文件的支持
        test: /\.css$/,
        // 提取出 Chunk 中的 CSS 代码到单独的文件中
        use: ExtractTextPlugin.extract({
          // 压缩 CSS 代码
          use: ['css-loader?minimize'],
          // 指定存放 CSS 中导入的资源（例如图片）的 CDN 目录 URL
          publicPath: '//img.cdn.com/id/'
        }),
      },
      {
        // 增加对 PNG 文件的支持
        test: /\.png$/,
        // 给输出的 PNG 文件名称加上 Hash 值
        use: ['file-loader?name=[name]_[hash:8].[ext]'],
      },
      // 省略其它 Loader 配置...
    ]
  },
  plugins: [
    // 使用 WebPlugin 自动生成 HTML
    new WebPlugin({
      // HTML 模版文件所在的文件路径
      template: './template.html',
      // 输出的 HTML 的文件名称
      filename: 'index.html',
      // 指定存放 CSS 文件的 CDN 目录 URL
      stylePublicPath: '//css.cdn.com/id/',
    }),
    new ExtractTextPlugin({
      // 给输出的 CSS 文件名称加上 Hash 值
      filename: `[name]_[contenthash:8].css`,
    }),
    // 省略代码压缩插件配置...
  ],
};
```

注意：

- 但多个 CDN 域名会增加域名解析时间，是否采用多域名解析需要根据具体的情况而定。
- 我们可以在 HTML HEAD 中加入 `<link rel="dns-prefetch" href="//js.cdn.com">` 去预解析域名，降低域名解析带来的延迟
- `//cdn.com/` 的URL省略掉了前面的 `http:` 或者` https:` 前缀，在具体数据请求时，它会根据当前 HTML 的 URL 加载模式去确定是采用 HTTP 还是 HTTPS 模式。

#### - 使用 Tree Shaking

Tree Shaking 可以用来剔除 JavaScript 中用不上的死代码。但仅仅针对的是 ES6 语法的代码。这是因为 ES6 模块化语法是静态的（`import x from './util';` ：导入导出的都是静态的字符串），webpack 可以简单的分析出哪些被 `import` 或 `export` 了，如果采用 ES5 （ `require(x+y)` ），webpack 则无法分析出具体哪些可以剔除。

> 目前的 Tree Shaking 还有些的局限性，经实验发现：
>
> 1. 不会对entry入口文件做 Tree Shaking。
> 2. 不会对[异步分割](https://webpack.wuhaolin.cn/4%E4%BC%98%E5%8C%96/4-12%E6%8C%89%E9%9C%80%E5%8A%A0%E8%BD%BD.html)出去的代码做 Tree Shaking。

步骤一：配置 `.babelrc`

```js
{
  "presets": [
    [
      "env",
      {
        // 关闭 Babel 的模块转换功能，保留原本的 ES6 模块化语法
        "modules": false
      }
    ]
  ]
}
```

并且在启动 Webpack 时带上 `--display-used-exports` 参数，以方便追踪 Tree Shaking 的工作。

注意：Webpack 只是指出了哪些函数用没用上，要剔除用不上的代码还得经过 UglifyJS 去处理一遍

步骤二：配合 UglifyJS

这里可以简单操作一下：在启动 Webpack 时带上 `--optimize-minimize` 参数，

具体的操作可以看一下 UglifyJS 模块。

#### - 提取公共代码

为什么要提取公共代码？

很多大型项目都是由多页面构成，并且所有的页面都采用同一套技术及基础库，如果每个页面单独打包、加载，就会导致每个包都包含大量的公共部分、基础库。影响用户体验：

- 相同的资源被重复加载，浪费用户的流量与服务器的成本
- 每个页面需要加载的资源过大，页面首屏加载过慢，影响用户体验

所以我们需要提取公共代码，单独打包，当用户加载多页面应用时，第一次访问的时候，公共代码将会被浏览器缓存起来，当加载其它页面时，用户不需要再重复加载公共模块，直接从缓存中获取即可。

怎么提取喃？

##### 第一步：使用 DllPlugin 来提取基础模块库，预构建依赖包

前面我们提到我们可以使用 DllPlugin 来提取基础模块库，这里不再赘述。

##### 第二步：使用 CommonsChunkPlugin 或 SplitChunksPlugin（webpack4以上） 对公共模块打包

**方式一：CommonsChunkPlugin **

Webpack 内置了专门用于提取多个 Chunk 中公共部分的插件 CommonsChunkPlugin：

```js
new webpack.optimize.CommonsChunkPlugin({
  // 从哪些 Chunk 中提取
  // chunks: ['a', 'b'], 不填则默认会从所有已知的 Chunk 中提取
  // name: 提取出的公共部分形成一个新的 Chunk，这个新 Chunk 的名称
  name: "index",
  // 在传入  公共chunk(commons chunk) 之前所需要包含的最少数量的 chunks 。
  // 数量必须大于等于2，或者少于等于 chunks的数量
  // 传入 `Infinity` 会马上生成 公共chunk，但里面没有模块。
  // 你可以传入一个 `function` ，以添加定制的逻辑（默认是 chunk 的数量）
  minChunks: 2,
  // 如果设置为 `true`，所有公共 chunk 的子模块都会被选择
  children: true,
  // 如果设置为 `true`，所有公共 chunk 的后代模块都会被选择
  deepChildren: true,
})
```



**方式二：SplitChunksPlugin（webpack4以上）**

```js
new webpack.optimize.SplitChunksPlugin({
  chunks: "all",
  minSize: 30000,
  minChunks: 1,
  maxAsyncRequests: 5,
  maxInitialRequests: 3,
  automaticNameDelimiter: '~',
  name: true,
  cacheGroups: {
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
}),
```

- chunks: 表示哪些代码需要优化，有三个可选值：initial(初始块)、async(按需加载块)、all(全部块)，默认为async
- minSize: 表示在压缩前的最小模块大小，默认为30000
- minChunks: 表示被引用次数，默认为1
- maxAsyncRequests: 按需加载时候最大的并行请求数，默认为5
- maxInitialRequests: 一个入口最大的并行请求数，默认为3
- automaticNameDelimiter: 命名连接符
- name: 拆分出来块的名字，默认由块名和hash值自动生成

  - test: 用于控制哪些模块被这个缓存组匹配到
  - priority: 缓存组打包的先后优先级
  - reuseExistingChunk: 如果当前代码块包含的模块已经有了，就不在产生一个新的代码块
- cacheGroups，缓存组。当我们不配置这个属性时，当我们import 多个库，比如jquery, lodash 时，进行代码分割，是会分别放入两个文件中的。但我们配置了cacheGroups 时，如上。它就会在打包时，先将要分割的代码缓存到vendor 组或者default 组，再一起打包成一个文件。
  -  priority 属性，是指，当一个文件满足两个以上的组的时候，那个组的priority 越大那么它的优先级越高，就会判断进那个组。
  -  reuseExistingChunk 属性，是指，当一个模块已经被打包过，那么它在其他文件中需要再次引入时，可以复用之前打包的文件。

#### - 按需加载

针对单页面应用首屏加载过慢，我们也可以采用懒加载、按需加载的方式控制首屏加载文件大小。

- 将网站划分为几个大的功能模块
- 每一块为一个 chunk，除首页 chunk 直接加载外，按需加载其余的 chunk
- 对于依赖代码量特别大的功能，也可以进行按需加载

我们知道在 ES6 中，我们使用 `import` 、`export` 静态的加载、导出文件，这里以ES6 `import()` 为例，更多可查看 [Module Methods](https://webpack.js.org/api/module-methods/#import)。

**步骤一：按需加载**

```js
export default class Routes extends React.Component {
  render() {
    return (
      <Router path={href} history={browserHistory}>
        <Switch>
          <Route exact path='/' component={Splash} />
          <Route path='/login' component={Login} />
          <Route path='/main/' component={getAsyncComponent(
              // 异步加载函数，异步地加载 main 组件
              () => import(/* webpackChunkName: 'page-main' */'./page/main')
            )}
          />
        </Switch>
      </Router>
    )
  }
}

/**
 * 异步加载组件
 * @param load 组件加载函数，load 函数会返回一个 Promise，在文件加载完成时 resolve
 * @returns {AsyncComponent} 返回一个高阶组件用于封装需要异步加载的组件
 */
function getAsyncComponent(load) {
  return class AsyncComponent extends React.PureComponent {

    componentDidMount() {
      // 在高阶组件 DidMount 时才去执行网络加载步骤
      load().then(({default: component}) => {
        // 代码加载成功，获取到了代码导出的值，调用 setState 通知高阶组件重新渲染子组件
        this.setState({
          component,
        })
      });
    }

    render() {
      const {component} = this.state || {};
      // component 是 React.Component 类型，需要通过 React.createElement 生产一个组件实例
      return component ? React.createElement(component) : null;
    }
  }
}
```



**步骤二：支持 import()**

你可能遇到 `import()` 报错 `Support for the experimental syntax 'dynamicImport' isn't currently enabled` ，这是因为 `import()` 还没有被加入到 ECMAScript 标准中去，所以我们需要安装 `npm install babel-plugin-syntax-dynamic-import --save-dev` ，并且配置  ：

```js
// .babelrc 文件
{
  "presets": [
    // ...
  ],
  "plugins": [
    "syntax-dynamic-import"
    // ...
  ]
}
```

并且可以在 webapck 中配置为动态加载的 Chunk 配置输出文件的名称 `chunkFilename` 。

### 2. 提升流畅度

前面分块、按需加载，都是网络加载层面的优化，下面我们主要介绍代码运行时的优化。

#### - 使用 Prepack

我们在之前 **区分环境** 中介绍过，`process.env.NODE_ENV === 'production'` 在打包时被直接替换成 `true` ，优化了源代码的运行逻辑，输出了性能更高的 JavaScript 代码。

Prepack 就是这样一个部分求值器，编译代码时，提前将运行结果放到编译后的代码中，而不是在代码运行时再去求值。

Prepack 的工作原理及流程大致如下：

- 通过 Babel 把 JavaScript 源码解析成抽象语法树（AST），以方便更细粒度地分析源码
- Prepack 实现了一个 JavaScript 解释器，用于执行源码。借助这个解释器 Prepack 才能掌握源码具体是如何执行的，并把执行过程中的结果返回到输出中

在 UglifyJS 压缩代码时，也会对这些代码进行优化，所以是否使用依据具体的项目而定，这里不做过多介绍。

#### - 开启 Scope Hoisting

Scope Hoisting（作用域提升） 可以让 webapck 打包出来的文件更小、运行更快。

假如现在有两个文件分别是 `util.js`:

```js
export default 'Hello,Webpack';
```

和入口文件 `main.js`:

```js
import str from './util.js';
console.log(str);
```

以上源码用 Webpack 打包后输出中的部分代码如下：

```js
[
  (function (module, __webpack_exports__, __webpack_require__) {
    var __WEBPACK_IMPORTED_MODULE_0__util_js__ = __webpack_require__(1);
    console.log(__WEBPACK_IMPORTED_MODULE_0__util_js__["a"]);
  }),
  (function (module, __webpack_exports__, __webpack_require__) {
    __webpack_exports__["a"] = ('Hello,Webpack');
  })
]
```

在开启 Scope Hoisting 后，同样的源码输出的部分代码如下：

```js
[
  (function (module, __webpack_exports__, __webpack_require__) {
    var util = ('Hello,Webpack');
    console.log(util);
  })
]
```

从中可以看出开启 Scope Hoisting 后，函数申明由两个变成了一个，`util.js` 中定义的内容被直接注入到了 `main.js` 对应的模块中。 这样做的好处是：

- 代码体积更小，因为函数申明语句会产生大量代码；
- 代码在运行时因为创建的函数作用域更少了，内存开销也随之变小。

> Scope Hoisting 的实现原理其实很简单：分析出模块之间的依赖关系，尽可能的把打散的模块合并到一个函数中去，但前提是不能造成代码冗余。 因此只有那些被引用了一次的模块才能被合并。

**注意：Scope Hoisting 需要分析出模块之间的依赖关系，因此源码必须采用 ES6 模块化语句，对于采用了非 ES6 模块化语法的代码，Webpack 会降级处理不使用 Scope Hoisting 优化。**

Webpack 中使用 Scope Hoisting 很简单，只需要配置：

```js
module.exports = {
  plugins: [
    // 开启 Scope Hoisting
    new webpack.optimize.ModuleConcatenationPlugin(),
  ],
  resolve: {
    // 针对 Npm 中的第三方模块优先采用 jsnext:main 中指向的 ES6 模块化语法的文件
    mainFields: ['jsnext:main', 'browser', 'main']
  },
};
```

可以在启动 Webpack 时带上 `--display-optimization-bailout` ，就可以知道 Webpack 对哪些代码做了降级处理。



### 3. 输出分析

虽然前面介绍了很多优化方法，但具体项目如何进行优化，这就要分析它的输出结果，来决定如何进行优化。

最直观的方式是分析它的输出文件，但 webpack 打包后的文件非常大且可读性差，下面就介绍几种可视化的分析工具，帮助你快速定位问题所在。

#### - webpack-bundle-analyzer

它能方便的让你知道：

- 打包出的文件中都包含了什么；
- 每个文件的尺寸在总体中的占比，一眼看出哪些文件尺寸大；
- 模块之间的包含关系；
- 每个文件的 Gzip 后的大小。

接入 webpack-bundle-analyzer 的方法很简单，步骤如下：

- 安装 webpack-bundle-analyzer 到全局，执行命令 `npm i -g webpack-bundle-analyzer`；

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

#### - 官方可视化分析工具 Webapck Analyse

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
