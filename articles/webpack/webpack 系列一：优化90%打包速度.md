### 前言

当使用 webpack 进行打包，一般会出现三方面的问题：

- 打包速度过慢 => 打包时间过长 => 优化开发体验
- 包体积太大  => 影响用户体验 => 优化输出质量
- 首屏加载时间过长 => 按需加载

下面我们就会根据这三方面入手，进行优化，首先优化打包速度。

### 一、基础配置

配置 webpack 其实很简单，基础的构建可以按照以下方式来配置 webpack：

#### 1. 入口 entry

 将源文件加入到 webpack 构建流程，可以是单入口或多入口:

```js
entry: { 
  "index": ["@babel/polyfill", `./${project}/index.js`],
}
```

#### 2. 出口 output

可以自定义输出文件的位置和名称:

```js
output: { 
  // path 必须为绝对路径
  path: path.join(__dirname, '../../dist/build'),
  // 包名称，[name] 为 entry 配置的 name，除此之外，还可以是 [hash]、[id]、[contenthash]
  filename: "[name].bundle.js",
  // 块名，公共块名
  chunkFilename: '[name].[chunkhash].bundle.js',
  publicPath: '/', 
}
```

#### 3. 解析策略  resolve

自定义寻找依赖模块时的策略:

```js
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
  }
},
```

#### 4. 解析和转换文件的策略 module

通常是配置 `module.rules` 里的 Loader:

```js
module: {
  noParse: /jquery/,
  rules: [
    {
      // 这里编译 js、jsx
      // 注意：如果项目源码中没有 jsx 文件就不要写 /\.jsx?$/，提升正则表达式性能
      test: /\.(js|jsx)$/,
      // 指定要用什么 loader 及其相关 loader 配置
      use: ["happypack/loader?id=happy-babel"],
      // 排除 node_modules 目录下的文件
      // node_modules 目录下的文件都是采用的 ES5 语法，没必要再通过 Babel 去转换
      exclude: /node_modules/
      // 也可以配置 include：需要引入的文件
    }
  ]
}
```

#### 5. 配置 plugin

配置 Plugin 去处理及优化其它的需求：

```js
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
```

#### 6. 配置环境 process.env.NODE_ENV

针对不同的环境配置相应的打包策略：

**方法一：webpack.DefinePlugin**

```js
// webpack编译过程中设置全局变量process.env
new webpack.DefinePlugin({
  'process.env': { 
	// 或  '"production"' ，环境变量的值需要是一个由双引号包裹的字符串
	NODE_ENV: JSON.stringify('production')  
  }
}
```

**方法二：webpack 命令时， NODE_ENV=development**

在 window 中配置 `NODE_ENV=production` 可能会卡住，所以使用 cross-env：

```js
cross-env NODE_ENV=production webpack --config scripts/webpack/webpack.config.prod.js
```

**方法三：使用 new webpack.EnvironmentPlugin(['NODE_ENV'])**

**方法四：webpack4 中 mode: 'production' 已经默认配置了 process.env.NODE_ENV = 'production' ，所以 webapck4 可以不定义**

#### 7. devtool：source map

用来增强调试过程。不同的值会明显影响到构建(build)和重新构建(rebuild)的速度：

生产环境：一般不设置或设置为 `cheap-module-source-map`

开发环境：一般设置为`eval` 、 `cheap-eval-source-map` 、`cheap-module-eval-source-map` 

### 二、缩小文件搜索范围

webpack 打包时，会从配置的 `entry` 触发，解析入口文件的导入语句，再递归的解析，在遇到导入语句时 Webpack 会做两件事情：

- 根据导入语句去寻找对应的要导入的文件。例如 `require('react')` 导入语句对应的文件是 `./node_modules/react/react.js`，`require('./util')` 对应的文件是 `./util.js`。
- 根据找到的要导入文件的后缀，使用配置中的 Loader 去处理文件。例如使用 ES6 开发的 JavaScript 文件需要使用 babel-loader 去处理。

以上两件事情虽然对于处理一个文件非常快，但是当项目大了以后文件量会变的非常多，这时候构建速度慢的问题就会暴露出来。 虽然以上两件事情无法避免，但需要尽量减少以上两件事情的发生，以提高速度。

接下来一一介绍可以优化它们的途径。

#### 1. 优化 loader 配置

使用 Loader 时可以通过 `test` 、 `include` 、 `exclude` 三个配置项来命中 Loader 要应用规则的文件

#### 2. 优化 resolve.module 配置

`resolve.modules` 用于配置 Webpack 去哪些目录下寻找第三方模块，`resolve.modules` 的默认值是 `['node_modules']` ，含义是先去当前目录下的 `./node_modules` 目录下去找想找的模块，如果没找到就去上一级目录 `../node_modules` 中找，再没有就去 `../../node_modules` 中找，以此类推。

#### 3. 优化 resolve.alias 配置

`resolve.alias` 配置项通过别名来把原导入路径映射成一个新的导入路径，

#### 4. 优化 resolve.extensions 配置

在导入语句没带文件后缀时，Webpack 会根据 resolve.extension 自动带上后缀后去尝试询问文件是否存在，所以在配置 `resolve.extensions` 应尽可能注意以下几点：

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

Webpack 会根据 `mainFields` 的配置去决定优先采用那份代码，`mainFields` 默认如下：

```js
mainFields: ['browser', 'main']
```

Webpack 会按照数组里的顺序去`package.json` 文件里寻找，只会使用找到的第一个。

假如你想优先采用 ES6 的那份代码，可以这样配置：

```js
mainFields: ['jsnext:main', 'browser', 'main']
```
#### 6. 优化 module.noParse 配置

`module.noParse` 配置项可以让 Webpack 忽略对部分没采用模块化的文件的递归解析处理，这样做的好处是能提高构建性能。 原因是一些库，例如 jQuery 、ChartJS， 它们庞大又没有采用模块化标准，让 Webpack 去解析这些文件耗时又没有意义。

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



### 三、使用 HardSourceWebpackPlugin

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



### 四、使用 DllPlugin

由于公司老项目的坑爹性，HardSourceWebpackPlugin 无法发挥作用，所以这里介绍一下 DllPlugin 的配置。

使用过 windows 电脑的人一般会看到 `.dll` 为后缀的文件，这些文件称为 **动态链接库** ，在 webpack 打包时，我们也可以构建项目的动态链接库，将项目中的网页基础模块、公共模块抽离出来，打包到一个个动态链接库中，当 webpack 打包时，需要导入的模块存在动态链接库中时，这个模块就不需要再被打包，直接去动态链接库中取就可以。 **注意：所有的动态链接库需要被加载** 。

webpack 已经内置了对动态链接库的支持，主要通过两个内置的插件接入：

- DllPlugin 插件：用于打包出一个个单独的动态链接库文件。
- DllReferencePlugin 插件：用于在主要配置文件中去引入 DllPlugin 插件打包好的动态链接库文件。

**步骤一：配置 webpack.config.dll.js，构建出动态链接库文件：**

```js
/**
 * 打包Dll的Webpack配置文件
 */
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
// 抽离 css
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const webpack = require('webpack');

// dll 配置文件（在下面）
const config = require('../../config');

// 这里把 styleLoaders 配置也单独独立出去了
const utils = require('../tools/utils');
const generateDllMap = require('../tools/generateDllMap')

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
  // 清除原有打包文件
  new CleanWebpackPlugin({
    root: process.cwd(), // 根目录
    verbose: true, // 开启在控制台输出信息
    dry: false // 启用删除文件
  }),
]

if (isDev) {
  // 将webpack生成的映射表中的数字id，替换为路径id  1---->./nodu_module/react/dist/react.js
  plugins.push(new webpack.NamedModulesPlugin());
} else {
  // 抽离 css
  plugins.push(new MiniCssExtractPlugin({
	filename: `[name]_${dllVersion}` + '.dll.css',
	chunkFilename: `[id]_${dllVersion}` + '.dll.css',
	disable: false, //是否禁用此插件
	allChunks: true,
  }));
}

// module.exports
module.exports = {
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
}

// 打包完，生成 dllAliasMap.json
process.on('exit', (err) => {
  generateDllMap(outputPath, err);
});
```

**注意：在 `webpack.dll.conf.js` 文件中，DllPlugin 中的 name 参数必须和 output.library 中保持一致。 原因在于 DllPlugin 中的 name 参数会影响输出的 json 文件中 name 字段的值， 而在 `webpack.config.js` 文件中 DllReferencePlugin 会去 json 文件读取 name 字段的值，并把值的内容作为在从全局变量中获取动态链接库中内容时的全局变量名。**

在这里，我把配置单独放在配置文件 `config/dll.conf.js` 中：

```js
const path = require('path');
const pullAll = require('lodash/pullAll'); // 数组除值
const uniq = require('lodash/uniq'); // 数组去重
const pm2Config = require('./pm2.config')

const reactDll = {
  version: '1.0.0',
  dllPlugin: {
    defaults: {
      exclude: [

      ],
      include: [
        'react',
        'react-dom',
      ],
      // 针对开发本地调试用devPath，针对各种环境打包时用buildPath
      devPath: 'common/@react_dll/dev_dll',
      buildPath: process.env.NODE_ENV === 'development' ? 'common/@react_dll/dev_dll' : 'common/@react_dll/prd_dll',
    },

    entry() {
      let dependencyNames = [];
      const exclude = reactDll.dllPlugin.defaults.exclude;
      const include = reactDll.dllPlugin.defaults.include;
      const includeDependencies = uniq([...include, ...dependencyNames]);
      return {
        reactDll: pullAll(includeDependencies, exclude),
      };
    },
  },
};

module.exports = reactDll;
```

**步骤二：使用动态链接库文件**

**webapck 打生产或测试包时，引入 DllPlugin 插件打包好的动态链接库文件**

```js
// 将 JavaScript 或 CSS 资产添加到 html-webpack-plugin 生成的 HTML 中
const AddAssetHtmlPlugin = require('add-asset-html-webpack-plugin')

// ...
// 引入所有的 dll 动态链接库
const manifests = glob.sync(path.resolve(`${dllPath}/*Dll.json`));
manifests.forEach(item => {
  plugins.push(new webpack.DllReferencePlugin({
    context: process.cwd(),
    manifest: item,
  }))
})
// 对 js
glob.sync(`${dllConfig.buildPath}/reactDll*.dll.js`).forEach((dllPath) => {
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

在配置好以上的 webpack 配置文件后，需要执行构建出所有配置好的动态链接库（ `webpack --display-chunks --color  --progress --config scripts/webpack/webpack.config.dll.js` ），因为 webpack 生产或测试配置文件中定义的 DllReferencePlugin 依赖这些文件。

你可以在 html 中引入，也可以在 router 中引入：

Html：使用 AddAssetHtmlPlugin 自动导入:

```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>webpack</title>
<link href="/index_3e3df.css" rel="stylesheet"></head>
<body>
	<div id="app"></div>
<script type="text/javascript" src="./js/reactDll_1.0.0.dll.js"></script><script type="text/javascript" src="/index.bundle.js"></script></body>
</html>
```

或 router 中（按需加载）：

```js
module.exports = {
  path: '/',
  component: require('../app/App'),
  childRoutes: [
    {
      path: 'DetailPage',
      getComponent(location, cb) {
        require.ensure([], (require) => {
          Promise.all([importDll('rechatDll')]).then(function() {
            cb(null, require('../pages/DetailPage'));
          })
        }, 'DetailPage');
      },
      onEnter: () => {
        setTitle({'title': name, barBgColor: backgroundColor, textColor: '#fff'});
        showNavBar();
      }
    }
  ],
}
```



### 五、 使用 HappyPack

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

### 六、github 详细配置

https://github.com/sisterAn/webapck-template