### 一. package.json 解读

```js
{
	"name": "hello world", // 项目名称
	"version": "0.0.1", // 版本号：大版本.次要版本.小版本
	"author": "张三",
	"description": "第一个node.js程序",
	"keywords":["node.js","javascript"], // 关键词，有助于 npm search 发现
	"repository": { // 存储库，指定代码所在位置（如果git repo在GitHub上，那么该npm docs 命令将能够找到文件位置。）
		"type": "git",
		"url": "https://path/to/url"
	},
	"license":"MIT", // 指定包许可证，详细可见[SPDX许可证ID的完整列表](https://spdx.org/licenses/)
	"engines": {"node": "0.10.x"}, // 指定该模块运行的平台，可以指定 node 版本、npm 版本等
	"bugs":{"url":"http://path/to/bug","email":"bug@example.com"}, // 项目问题跟踪器的URL和应报告问题的电子邮件地址。
	"contributors":[{"name":"李四","email":"lisi@example.com"}],
    "bin": { // 指定内部命令对应的可执行文件的位置，在 scripts 中就可以简写
    	"webpack": "./bin/webpack.js"
  	},
    "main": "lib/webpack.js", // 指定加载的模块入口文件，require('moduleName')就会加载这个文件。这个字段的默认值是模块根目录下面的index.js。
    "config" : { "port" : "8080" }, // 用于添加命令行的环境变量（用户在运行 scripts 命令时，就默认在脚本文件中添加 process.env.npm_package_config_port，用户可以通过 npm config set foo:port 80 命令更改这个值）
	"scripts": { // 指定运行脚本的 npm 命令行缩写
		"start": "node index.js"
	},
    "peerDependencies": { // 指定项目安装必须一起安装的模块及其版本号，（注意：从 npm 3.0 开始，peerDependencies不会再默认安装）
    	"chai": "1.x"
  	},
	"dependencies": { // 指定项目运行所依赖的模块
		"express": "latest",
		"mongoose": "~3.8.3",
		"handlebars-runtime": "~1.0.12",
		"express3-handlebars": "~0.5.0",
		"MD5": "~1.2.0"
	},
	"devDependencies": { // 指定项目开发所需要的模块
		"bower": "~1.2.8",
		"grunt": "~0.4.1",
		"grunt-contrib-concat": "~0.3.0",
		"grunt-contrib-jshint": "~0.7.2",
		"grunt-contrib-uglify": "~0.2.7",
		"grunt-contrib-clean": "~0.5.0",
		"browserify": "2.36.1",
		"grunt-browserify": "~1.3.0",
	},
    "browser": { // 指定该模板供浏览器使用的版本
  		"tipso": "./node_modules/tipso/src/tipso.js"
	},
    "preferGlobal": true, // 表示当用户不将该模块安装为全局模块时（即不用–global参数），要不要显示警告，表示该模块的本意就是安装为全局模块。
}
```

`package.json` 在 node 和 npm 环节都要使用，node 在调用 `require` 的时候去查找模块，会按照一个次序去查找，`package.json` 会是查找中的一个环节。npm 用的就比较多，其中的 `dependencies` 字段就是本模块的依赖的模块清单。每次`npm update`的时候，npm会自动的把依赖到的模块也下载下来。当`npm install` 本模块的时候，会把这里提到的模块都一起下载下来。通过package.json,就可以管理好模块的依赖关系。

关于更多规范，请看官方[npm-package.json](https://docs.npmjs.com/files/package.json.html)

### 二. 版本号规范

> - **指定版本**：比如`1.2.2`，遵循“大版本.次要版本.小版本”的格式规定，安装时只安装指定版本。
> - **波浪号（tilde）+指定版本**：比如`~1.2.2`，表示安装1.2.x的最新版本（不低于1.2.2），但是不安装1.3.x，也就是说安装时不改变大版本号和次要版本号。
> - **插入号（caret）+指定版本**：比如ˆ1.2.2，表示安装1.x.x的最新版本（不低于1.2.2），但是不安装2.x.x，也就是说安装时不改变大版本号。**需要注意的是，如果大版本号为0，则插入号的行为与波浪号相同，这是因为此时处于开发阶段，即使是次要版本号变动，也可能带来程序的不兼容。**
> - **latest**：安装最新版本。

### 三. npm install 与 **npm update**

- 如果本地 `node_modules` 已安装，再次执行 `install` 不会更新包版本, 执行 `update` 才会更新; 而如果本地 `node_modules` 为空时，执行 `install/update` 都会直接安装更新包;
- `npm update` 总是会把包更新到符合 `package.json` 中指定的 semver(语义化版本) 的**最新**版本号——本例中符合 `^1.8.0` 的最新版本为 `1.15.0`
- 一旦给定 `package.json`, 无论后面执行 `npm install` 还是 `update`, `package.json` 中的 webpack 版本一直顽固地保持 一开始的 `^1.8.0` 岿然不动

### 四. npm i 与 **npm install**

实际使用的区别点主要如下： 

- 用`npm i`安装的模块无法用`npm uninstall`删除，用`npm un`才卸载掉 
- `npm i`会帮助检测与当前 node 版本最匹配的 npm 包版本号，并匹配出来相互依赖的 npm 包应该提升的版本号 
- 部分 npm 包在当前 node 版本下无法使用，必须使用建议版本 
- 安装报错时 install 肯定会出现 `npm-debug.log`  文件，`npm i`不一定

### 五. npm devDependencies 与 dependencies

`--save-dev`

或

`—save`

首先需要说明的是 Dependencies一词的中文意思是依赖和附属的意思，而dev则是 develop（开发）的简写。

所以它们的区别在 package.json 文件里面体现出来的就是，使用 `--save-dev` 安装的 插件，被写入到 devDependencies 域里面去，而使用 `—save` 安装的插件，则是被写入到 dependencies 区块里面去。

那 package.json 文件里面的 devDependencies  和 dependencies 对象有什么**区别**呢？

**devDependencies  里面的插件只用于开发环境，不用于生产环境，而 dependencies  是需要发布到生产环境的**。

比如我们写一个项目要依赖于jQuery，没有这个包的依赖运行就会报错，这时候就把这个依赖写入dependencies

### 六. 全局安装与本地安装

通过 `-g` 来安装的包，将包安装成全局可用的可执行命令。

#### 1. 全局安装， 将包安装成全局可用的可执行命令

```js
// 全局安装 babel-cli
babel app.js
```
#### 2. 本地安装

```Js
// 本地安装 babel-cli
node_modules/.bin/babel app.js
```
#### 3. 修改全局安装默认路径

- 设置自定义的全局安装路径

  ```js
  npm config set prefix "/usr/local/npm" // 自定义的全局安装路径
  npm config set cache "/usr/local/npm" // 自定义的全局安装路径
  ```

- 设置环境变量

  ```js
  PATH="/usr/local/npm/bin" // 将 /usr/local/npm/bin 追加到 PATH 变量中
  NODE_PATH="/usr/local/npm/lib/node_modules" // 指定 NODE_PATH 变量
  ```

  操作系统中都会有一个`PATH`环境变量，想必大家都知道，当系统调用一个命令的时候，就会在PATH变量中注册的路径中寻找，如果注册的路径中有就调用，否则就提示命令没找到。

  而 `NODE_PATH` 就是`NODE`中用来 **寻找模块所提供的路径注册环境变量** 。我们可以使用上面的方法指定`NODE_PATH` 环境变量。

  使用 `npm config list` 查看配置

### 七. npm 包命令

```js
npm list -g --depth 0 // 查看全局安装过的包 -g:全局的安装包 list：已安装的node包 –depth 0：深度0
npm view <packageName> // 查看npm服务器中包版本号 
npm info <packageName> // npm服务器更多信息，更多版本号
npm ls <packageName> // 本地包
npm ls <packageName> -g // 全局安装包
npm docs // 打开包git目录

// 注意：npm build 与 npm start 是项目中常用的命令，注意它们有什么不同
npm start [--<args>] // 在 package.json 文件中定义的 "scripts" 对象中查找 "start" 属性，执行该属性定义的命令，如果没有定义，默认执行 node server.js 命令
npm build [<package-folder>] // 其中，<package-folder> 为其根目录中包含一个 package.json 文件的文件夹，这是由 npm link 命令和 npm install 命令组成的管道命令，通常在安装过程中被调用。如果想要直接运行它，则运行 npm run build
```

还有其他的 钩子命令，具体项目中我还没用到，你可以自行了解。

package.json 中 scripts 常用命令：

```js
// 删除目录
"clean": "rimraf dist/*",

// 本地搭建一个 HTTP 服务
"serve": "http-server -p 9090 dist/",

// 打开浏览器
"open:dev": "opener http://localhost:9090",

// 实时刷新
 "livereload": "live-reload --port 9091 dist/",

// 构建 HTML 文件
"build:html": "jade index.jade > dist/index.html",

// 只要 CSS 文件有变动，就重新执行构建
"watch:css": "watch 'npm run build:css' assets/styles/",

// 只要 HTML 文件有变动，就重新执行构建
"watch:html": "watch 'npm run build:html' assets/html",

// 部署到 Amazon S3
"deploy:prod": "s3-cli sync ./dist/ s3://example-com/prod-site/",

// 构建 favicon
"build:favicon": "node scripts/favicon.js",
```

### 八. 简写形式

```js
npm start   // 是 npm run start 的简写
npm stop    // 是 npm run stop 的简写
npm test    // 是 npm run test 的简写
npm restart // 是 npm run stop && npm run restart && npm run start 的简写
```
### 九. process

我们可以通过环境变量`process.env`对象，拿到 npm 所有的配置变量。其中 npm 脚本可以通过`npm_config_`前缀，拿到 npm 的配置变量。通过`npm_package_`前缀，拿到`package.json`里面的字段。

```js
console.log(process.env.npm_package_name); // chejianer
console.log(process.env.npm_package_version); // 1.0.0
console.log(process.env); // ... 
```

对于 **全局模式安装的包（通过 -g 来安装的包，将包安装成全局可用的可执行命令，并不意味着任何地方都可以通过 require() 来引用它）**：它会通过 bin 字段配置，将实际脚本链接到 Node 可执行目录下，例如

```js
"bin": {
  "webpack": "./bin/webpack.js"
},
```

通过全局安装的包都安装到一个统一的目录下，可以通过以下方式获得:

```js
path.resolve(process.execPath, "..", "..", "lib", "node_modules") 
// 例如：/usr/local/lib/node_modules
```

### 一零. npm 发布包

- **创建一个空文件**：

```js
// lib/index.js
exports.sayHello = function () {
    return "Hello An!";
};
```

- **运行：`npm init`**

```js
package name: (module) hello-an
version: (1.0.0) 0.1.0
description: a hello-an package
entry point: (hello.js) 
test command: 
git repository: 
keywords: hello an
author: sisterAn
license: (ISC) MIT
About to write to /Users/lianran777/Study/node/chejianer_node/module/package.json:

{
  "name": "hello-an",
  "version": "1.0.0",
  "description": "a hello-an package",
  "main": "hello.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "hello",
    "an"
  ],
  "author": "sisterAn",
  "license": "MIT"
}


Is this OK? (yes) 
```

- **注册 npm 包仓库账号**

```js
npm adduser
```

- **上传包**

```js
npm publish . // package.json 所在目录
```

在这个过程中，npm 会将目录打包成一个存档文件，然后上传到官方源仓库中

- **管理包权限**

```js
npm owner add <user> [<@scope>/]<pkg>
npm owner rm <user> [<@scope>/]<pkg>
npm owner ls [<@scope>/]<pkg>
```

在自己的项目中安装包 `npm install`，通过 `npm ls` 分析模块路径找到的所有包，并生成依赖树。
