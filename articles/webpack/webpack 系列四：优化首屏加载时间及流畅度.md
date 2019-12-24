### 一、区分环境

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



### 二、CDN 加速

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



### 三、使用 Tree Shaking

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

### 四 、提取公共代码

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
  - priority 属性，是指，当一个文件满足两个以上的组的时候，那个组的priority 越大那么它的优先级越高，就会判断进那个组。
  - reuseExistingChunk 属性，是指，当一个模块已经被打包过，那么它在其他文件中需要再次引入时，可以复用之前打包的文件。

### 五、按需加载

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