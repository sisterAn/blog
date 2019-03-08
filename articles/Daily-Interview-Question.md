## 1. npm 模块安装机制，为什么输入 npm install 就可以自动安装对应的模块？

#### 1. npm 模块安装机制：

- 发出`npm install`命令
- 查询node_modules目录之中是否已经存在指定模块
  - 若存在，不再重新安装
  - 若不存在
    - npm 向 registry 查询模块压缩包的网址
    - 下载压缩包，存放在根目录下的`.npm`目录里
    - 解压压缩包到当前项目的`node_modules`目录

#### 2. npm 实现原理

输入 npm install 命令并敲下回车后，会经历如下几个阶段（以 npm 5.5.1 为例）：

1. **执行工程自身 preinstall**

当前 npm 工程如果定义了 preinstall 钩子此时会被执行。

2. **确定首层依赖模块**

首先需要做的是确定工程中的首层依赖，也就是 dependencies 和 devDependencies 属性中直接指定的模块（假设此时没有添加 npm install 参数）。

工程本身是整棵依赖树的根节点，每个首层依赖模块都是根节点下面的一棵子树，npm 会开启多进程从每个首层依赖模块开始逐步寻找更深层级的节点。

3. **获取模块**

获取模块是一个递归的过程，分为以下几步：

- 获取模块信息。在下载一个模块之前，首先要确定其版本，这是因为 package.json 中往往是 semantic version（semver，语义化版本）。此时如果版本描述文件（npm-shrinkwrap.json 或 package-lock.json）中有该模块信息直接拿即可，如果没有则从仓库获取。如 packaeg.json 中某个包的版本是 ^1.1.0，npm 就会去仓库中获取符合 1.x.x 形式的最新版本。
- 获取模块实体。上一步会获取到模块的压缩包地址（resolved 字段），npm 会用此地址检查本地缓存，缓存中有就直接拿，如果没有则从仓库下载。
- 查找该模块依赖，如果有依赖则回到第1步，如果没有则停止。

4. **模块扁平化（dedupe）**

上一步获取到的是一棵完整的依赖树，其中可能包含大量重复模块。比如 A 模块依赖于 loadsh，B 模块同样依赖于 lodash。在 npm3 以前会严格按照依赖树的结构进行安装，因此会造成模块冗余。

从 npm3 开始默认加入了一个 dedupe 的过程。它会遍历所有节点，逐个将模块放在根节点下面，也就是 node-modules 的第一层。当发现有**重复模块**时，则将其丢弃。

这里需要对**重复模块**进行一个定义，它指的是**模块名相同**且 **semver 兼容。**每个 semver 都对应一段版本允许范围，如果两个模块的版本允许范围存在交集，那么就可以得到一个**兼容**版本，而不必版本号完全一致，这可以使更多冗余模块在 dedupe 过程中被去掉。

比如 node-modules 下 foo 模块依赖 lodash@^1.0.0，bar 模块依赖 lodash@^1.1.0，则 **^1.1.0** 为兼容版本。

而当 foo 依赖 lodash@^2.0.0，bar 依赖 lodash@^1.1.0，则依据 semver 的规则，二者不存在兼容版本。会将一个版本放在 node_modules 中，另一个仍保留在依赖树里。

举个例子，假设一个依赖树原本是这样：

node_modules
-- foo
---- lodash@version1

-- bar
---- lodash@version2

假设 version1 和 version2 是兼容版本，则经过 dedupe 会成为下面的形式：

node_modules
-- foo

-- bar

-- lodash（保留的版本为兼容版本）

假设 version1 和 version2 为非兼容版本，则后面的版本保留在依赖树中：

node_modules
-- foo
-- lodash@version1

-- bar
---- lodash@version2

5. **安装模块**

这一步将会更新工程中的 node_modules，并执行模块中的生命周期函数（按照 preinstall、install、postinstall 的顺序）。

6. **执行工程自身生命周期**

当前 npm 工程如果定义了钩子此时会被执行（按照 install、postinstall、prepublish、prepare 的顺序）。

最后一步是生成或更新版本描述文件，npm install 过程完成。

参考 [npm 模块安装机制简介](http://www.ruanyifeng.com/blog/2016/01/npm-install.html)

[详解npm的模块安装机制](https://www.bbsmax.com/A/qVdemmnEdP/)

[npm install的实现原理](https://www.zhihu.com/question/66629910)



## 2. Object.prototype.toString.call() 、 instanceof 以及 Array.isArray()

1. Object.prototype.toString.call()

   每一个继承 Object 的对象都有 `toString` 方法，如果 `toString` 方法没有重写的话，会返回 `[Object type]`，其中 type 为对象的类型。但当除了 Object 类型的对象外，其他类型直接使用  `toString` 方法时，会直接返回都是内容的字符串，所以我们需要使用call或者apply方法来改变toString方法的执行上下文。

   ```js
   const an = ['Hello','An'];
   an.toString(); // "Hello,An"
   Object.prototype.toString.call(an); // "[object Array]"
   ```

   这种方法对于所有基本的数据类型都能进行判断，即使是 null 和 undefined  。

   ```js
   Object.prototype.toString.call('An') // "[object String]"
   Object.prototype.toString.call(1) // "[object Number]"
   Object.prototype.toString.call(Symbol(1)) // "[object Symbol]"
   Object.prototype.toString.call(null) // "[object Null]"
   Object.prototype.toString.call(undefined) // "[object Undefined]"
   Object.prototype.toString.call(function(){}) // "[object Function]"
   Object.prototype.toString.call({name: 'An'}) // "[object Object]"
   ```

   `Object.prototype.toString.call()` 常用于判断浏览器内置对象时。

   更多实现可见 [谈谈 Object.prototype.toString](https://juejin.im/post/591647550ce4630069df1c4a)

2. instanceof

   `instanceof`  的内部机制是通过判断对象的原型链中是不是能找到类型的 `prototype`。

   使用 `instanceof`判断一个对象是否为数组，`instanceof`  会判断这个对象的原型链上是否会找到对应的 `Array` 的原型，找到返回 `true`，否则返回 `false`。

   ```js
   []  instanceof Array; // true
   ```

   但 `instanceof` 只能用来判断对象类型，原始类型不可以。并且所有对象类型 instanceof Object 都是 true。

   ```js
   []  instanceof Object; // true
   ```

3. Array.isArray()

   - 功能：用来判断对象是否为数组

   - instanceof 与 isArray

     当检测Array实例时，`Array.isArray` 优于 `instanceof` ，因为 `Array.isArray` 可以检测出 `iframes`

     ```js
     var iframe = document.createElement('iframe');
     document.body.appendChild(iframe);
     xArray = window.frames[window.frames.length-1].Array;
     var arr = new xArray(1,2,3); // [1,2,3]
     
     // Correctly checking for Array
     Array.isArray(arr);  // true
     Object.prototype.toString.call(arr); // true
     // Considered harmful, because doesn't work though iframes
     arr instanceof Array; // false
     ```

   - `Array.isArray()` 与 `Object.prototype.toString.call()`

     `Array.isArray()`是ES5新增的方法，当不存在 `Array.isArray()` ，可以用 `Object.prototype.toString.call()` 实现。

     ```js
     if (!Array.isArray) {
       Array.isArray = function(arg) {
         return Object.prototype.toString.call(arg) === '[object Array]';
       };
     }
     ```

     

   

## 3. 介绍下观察者模式和订阅-发布模式的区别，各自适用于什么场景



## 4. cookie 和 token 都存放在 header 中，为什么不会劫持 token？

cookie 每次请求都会带上 内容主要包括：名字，值，过期时间，路径和域,cookie是有状态的，被劫持不安全, 可以设置httpOnly 防止 cors，token可以设置失效时间, 是无状态的，被劫持后危险性要低一些, 在跨端能力上更好