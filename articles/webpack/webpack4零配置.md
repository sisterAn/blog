都2020年了，你还在用 webpack3 吗？😱😱😱！webapck5 都要出了，webapck3 已经过时了！！！

👇，让我们开始走进 webapck4 吧！

相对于 webpack3，webapck4 调整了什么喃？

### 一、Web 性能

#### 1. optimization.splitChunks

webapck4 新增了 `optimization.splitChunks` 特性，它是对老版本 `optimization.CommonsChunkPlugin` 的一种优化改进。

##### 1.1 splitChunks 与 CommonsChunkPlugin

老版本的块图：

- 在  `CommonsChunkPlugin` 中，`chunk` 之间是通过父子关系连接，并且 `chunk` 包含 `module`。所以当子 `chunk` 有父 `chunk` 时，当加载这个子 `chunk` 时，一定已经加载了至少一个父 `chunk`，当子块有某一个 `module` 时，并且它的所有父 `chunk` 都有这个 `module`，此时子 `chunk` 中的这个 `module` 是可以被删除的。
- `CommonsChunkPlugin` 中，在入口文件或某一异步节点都会有一个 `chunk` 列表，这些 `chunk` 列表是被并行加载的。

这种块依赖关系图使得很难将块分裂开来，例如：我从一个或多个 `chunks` 中抽取一个 `module`，单独封装成一个 `chunk`，那我们如何将这个 `chunk` 加入到 `chunk graph` ，作为父块，或子块，`CommonsChunkPlugin` 会将其添加为父块，但这在技术上是错误的，并且会对其它优化产生消极影响。

 `splitChunks` 进入了一个新的对象：块组（ `ChunkGroup` ），一个 `ChunkGroup` 包含很多块，在入口文件或某一异步节点都有相应的一个 `ChunkGroup` ，`ChunkGroup` 中的`chunk` 是并行加载的，并且一个 `chunk` 可以被不同的 `ChunkGroup` 引用。这就解决了上述问题。

**注意： `chunk` 之间是不存在父子关系， `ChunkGroup` 之间仍然存在父子关系。但这种父子关系对分块来说，已经不具有影响力。**

另一方面，`splitChunks` 与 `CommonsChunkPlugin` 对块的划分也不一致：

- `CommonsChunkPlugin` ：由 `minChunks` （模块在块中重复出现次数，超过 `minChunks` 默认生成新 `chunk`）去确定是否生成新块
- `splitChunks` ：使用模块重复计数和模块类别（例如 `node_modules`），通过启发式方法自动识别应按块划分的模块。 并分割大块…

所以，`splitChunks` 相对于 `CommonsChunkPlugin` ：

 `CommonsChunkPlugin` ：

- 可能加载不需要的代码
- 对异步支持不友好
- 内部实现难以理解
- 很难使用

`splitChunks` ：

- 仅仅加载需要的模块
- 对异步支持友好
- 大多数都是自动化的
- 使用起来很简单

##### 1.2 splitChunks 新增配置项

先看一下 `CommonsChunkPlugin` 的配置：

```js
{
  name: string, // or
  names: string[], 
  // 公共 chunk ( commnons chunk ) 的名称，如果是已经存在的 chunk 名，则就把公共模块代码合并到这个 chunk 上，否则创建名为 name 的 chunk 进行合并
  
  filename: string, 
  // 指定公共 chunk 的文件名模版
      
  chunks: string[],
  // 指定从哪些 chunk 当中去找公共模块，省略该选项的时候，默认就是 entry chunks

  minChunks: number|Infinity|function(module, count) => boolean,
  // number：至少在 minChunks 个块中存在的模块
  // Infinity：只有当入口文件（entry chunks） >= 3 才生效，用来在第三方库中分离自定义的公共模块
  // function：可以在函数内进行你规定好的逻辑来决定某个模块是否提取成为commons chunk   
  // 才被提取到公共 chunk 中

  children: boolean,
  // 如果设置为 `true`，source chunks 是通过 entry chunks（入口文件）进行 code split 出来的children chunks
  // 它会把 entry chunk 创建的 children chunks 的共用模块合并到自身，但这会导致初始加载时间较长
  // children 和 chunks 不能同时设置，因为它们都是指定 source chunks 的

  deepChildren: boolean,
  // 如果设置为 `true`，所有公共 chunk 的后代模块都会被选择

  async: boolean|string,
  // 解决 children:true 时，children chunks 合并到 entry chunks 自身时，初始加载时间过长的问题
  // async 设为 true 时，commons chunk 将不会合并到自身，而是使用一个新的异步的 commons chunk
  // 当这个 children chunk 被下载时，自动并行下载该 commons chunk

  minSize: number,
  // 在 公共chunk 被创建立之前，所有 公共模块 (common module) 的最小大小
}
```

`optimization.splitChunks` 在 `CommonsChunkPlugin` 上增加了一些新的配置项：

**automaticNameDelimiter：**默认情况下，webpack会使用界定符来连接块名（例如 `vendor〜main.js` ），默认情况下为 `~` ，不过现在你可以使用 `automaticNameDelimiter` 进行更改。

**chunks：** 表示选择哪些块进行优化，仅仅支持 `"initial"` 、 `"async"` 和 `"all"` ，或函数。

**新增了 maxSize：** 将分割块控制在 `minSize` 和 `maxSize` 之间，可以更好地利用 HTTP/2 的长期缓存。

**为 import() 增加了对预取（Prefetch）及预加载（Preload）的支持：** webpack 4.6.0+增加了对预取和预加载的支持，通过 `import(/* webpackPrefetch: true */ "...")` 、 `import(/* webpackPreload: true */ "...");` 配置预取按需加载的代码，这个功能超强大。注意 Prefetch 指令与 Preload 有很多不同之处：预取的块在父块完成加载后（空闲时）启动；预加载块与父块并行加载。

**增加了对 [contenthash] 的支持：** 通过 `mini-css-extract-plugin` 使用 `CSS` 时，会缩小哈希范围。

**注意：**

- 按需加载块时并行请求的最大数量将小于或等于 6
- 初始页面加载时并行请求的最大数量将小于或等于 4

##### 1.3 splitChunks 默认配置

```js
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: 'async', 
      // 只针对异步块进行拆分
      minSize: 30000, 
      // chunk 最小体积
      minRemainingSize: 0, 
      // webpack5 中引入，限制拆分后剩余的块的最小大小，避免大小为零的模块
      // 仅仅对剩余的最后一个块有效
      // 在“开发”模式下默认为0 
      // 在其他情况下，默认值为 minSize 的值
      // 因此除极少数需要深度控制的情况外，无需手动指定它
      
      maxSize: 0,
      // 旨在与HTTP/2和长期缓存一起使用 
      // 它增加了请求数量以实现更好的缓存
      // 它还可以用于减小文件大小，以加快重建速度。
      
      minChunks: 1,
      // 分割一个模块之前必须共享的最小块数
      maxAsyncRequests: 6,
      // 按需加载时的最大并行请求数
      maxInitialRequests: 4,
      // 入口的最大并行请求数
      automaticNameDelimiter: '~',
      // 界定符
      automaticNameMaxLength: 30,
      // 块名最大字符数
      cacheGroups: { // 缓存组
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10
        },
        default: { // 可以设置为 false，禁用缓存
          minChunks: 2,
          priority: -20, // 优先级
          reuseExistingChunk: true
          // 当一个模块已经被打包过，
          //那么它在其他文件中需要再次引入时
          // 可以复用之前打包的文件
        }
      }
    }
  }
};
```



### 二、合规性



### 三、其它特性



### 四、零配置

选择了默认配置以适合Web性能最佳实践，但是项目的最佳策略可能会有所不同。 如果要更改配置，则应评估所做更改的影响，以确保确实有好处。



### 五、浏览器缓存与 hash 值

在定义包名称（例如 `chunkFilename` 、 `filename`），我们一般会用到哈希值，不同的哈希值使用的场景不同：

- **hash**：build-specific，即每次编译都不同，**适用于开发环境**
- **chunkhash**：chunk-specific，是根据每个 chunk 的内容计算出来的 hash，**适用于生产环境**