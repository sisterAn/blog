在开发过程中，修改代码是最常见的事情，我们不可能每次修改代码都去重新打包，刷新浏览器，所以需要配置一套自动化的解决方案，每次修改代码，都能自动化的重新打包（或增量打包），刷新浏览器。

### 一、使用自动刷新

#### 1. 步骤一：监听文件变更

**有两种方式：**

**方式一：**

在配置文件 `webpack.config.js` 中设置 `watch: true`

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
**方式二：**

在执行启动 Webpack 命令时，带上 `--watch` 参数，完整命令是 `webpack --watch`

#### 2. 步骤二：控制浏览器自动刷新

##### 方式一：webpack-dev-server

在使用 webpack-dev-server 模块去启动 webpack 模块时，webpack 模块的监听模式默认会被开启。 webpack 模块会在文件发生变化时告诉 webpack-dev-server 模块。

devServer 默认（默认开启 devServer.inline ）采用往开发的网页中注入代理客户端代码，通过代理客户端去刷新整个页面的原理实现网页的自动刷新。但它会向每一个 Chunk 包都注入一个代理服务器，当项目需要输出的 Chunk 有很多个时，这会导致构建缓慢，所以，另一方面：

我们可以关闭 inline 模式，只注入一个代理客户端，此时，devServer 会采用将开发的网页装进一个 iframe 中，通过刷新 iframe 去看到最新效果的原理实现网页的自动刷新。

##### 方式二：koa + webpack-dev-middleware + webpack-hot-middleware 前后端同构

https://segmentfault.com/a/1190000004883199



### 二、开启模块热替换

通过前面的介绍，项目每次更新代码，都会刷新整个网页，需要开发人员等待网页刷新，反应不灵敏，那么如何实现在不刷新整个网页的情况下做到超灵敏的实时预览。

#### 模块热替换与自动刷新

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