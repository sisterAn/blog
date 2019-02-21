- 开发环境已普及
- 浏览器环境支持不好（需要开发环境编译）
- 开发环境如何使用+重点语法的掌握

问题：

- 模块化，开发环境如何打包
- Class与普通构造函数
- Promise
- 总结ES6常用功能

### 模块化与打包

1. export与import语法

   ```js
   /* util1.js*/
   export default {
       a: 1
   }
   
   /* util2.js */
   export function fun1() {}
   export function fun2() {}
   
   /* index.js */
   import util1 from './util1'
   import {fun1, fun2} from 'util2'
   ```

2. 模块化

   - 开发环境 - babel语法解析，将ES6语法解析成浏览器可识别的低层次语法
     - node环境安装

     - 创建项目工程文件:es-test

     - 在项目根目录下，npm init

     - npm install —save-dev babel-core babel-preset-env babel-preset-latest

     - 在项目根目录下，创建.babelrc文件，这是babel的配置文件

       ```js
       // .babelrc
       {
         "presets": ["env", "latest"],
         "plugins": []
       }
       ```

     - npm install —global babel-cli

     - babel —version，可以查看当前版本号

     - 创建./src/index.js

       ```js
       .
       └── src
           └── index.js
       ```

       ```js
       // index.js
       [1, 2, 3].map(item => item + 1)
       ```

     - 运行 babel ./src/index.js

       ```js
       // 控制台打印
       [1, 2, 3].map(function (item) {
         return item + 1;
       });
       ```
   - 开发环境 - webpack处理模块化
     - npm install webpack babel-loader —save-dev

     - 配置 webpack.config.js

       ```js
       module.exports = {
           entry: "./src/index.js",
           output: {
               path: __dirname,
               filename: "./build/bundle.js"
           },
           module: {
               rules: [{
                   test: /\.js?$/,
                   exclude: /(node_modules)/,
                   loader: "babel-loader"
               }]
           }
       }
       ```

     - 配置 package.json 中的 scripts

       ```js
       "scripts": {
           "start": "webpack",
           "test": "echo \"Error: no test specified\" && exit 1"
         },
       ```

     - 运行 npm start
   - 开发环境 - rollup
     - npm init
     - npm i —save-dev rollup rollup-plugin-node-resolve rollup-plugin-babel babel-plugin-external-helpers babel-preset-latest
     - 配置 .babelrc
     - 配置 rollup.config.js
   - Webpack、rollup比较
     - rollup功能单一，webpack功能强大
     - 参考设计原则和《Linux/Unix设计思想》
     - 工具尽量功能单一，可集成，可扩展

3. 总结

   - 语法： import export（注意有无 default）
   - 环境： babel 编译 ES6 语法，模块化可用webpack 和 rollup
   - 扩展：说一下自己对模块化标准统一的期望

4. 时间轴

   - 没有模块化
   - AMD 成为标准，require.js （也有CMD）
   - 前端打包工具，是nodejs模块化可以被使用
   - ES6出现，想统一现在的所有模块化标准
   - nodejs积极支持，浏览器尚未统一
   - 可以自造lib，但不能自造标准

### Class 和普通构造函数的区别

1. JS 构造函数

   ```js
   // 构造函数
   function MathHandle(x, y) {
       this.x = x
       this.y = y
   }
   // 原型扩展
   MathHandle.prototype.add = function() {
       return this.x + this.y
   }
   // 实例化
   var m = new MathHAndle(1, 2)
   console.log(m.add()) // 3
   typeof MathHandle // function
   MathHandle/prototype.constructor === MathHandle // true
   m.__proto__ === MathHandle.prototype // true
   ```

   

2. Class 基本语法

   ```react
   class Ad extends Rect.Component {
       constructor(props) {
           super(props)
           this.state = {
               data: []
           }
       }
       componentDidMount() {}
       render() {
           return (
               <div>hello world</div>
           )
       }
   }
   ```

3. 语法糖

   ```react
   class MathHandle {
       // ...
   }
   
   typeof MathHandle // function
   MathHandle/prototype.constructor === MathHandle // true
   m.__proto__ === MathHandle.prototype // true
   ```

4. 继承

   ```js
   // 动物
   function Animal() {
       this.eat = function() {
           console.log('animal eat')
       }
   }
   // 狗
   function Dog() {
       this.dark = function() {
           console.log('dog dark')
       }
   }
   
   Dog.prototype = new Animal()
   var d = new Dog()
   ```

   ```React
   class Animal {
       constructor(name) {
           this.name = name
       }
       eat() {
           console.log(`${this.name} eat`)
       }
   }
   
   class Dog extends Animal {
       constructor(name) {
           super(name)
           this.name = name
       }
       say() {
           console.log(`${this.name} say`)
       }
   }
   
   var dog = new Dog('hashiqi')
   dog.say()
   dog.eat()
   ```

5. 总结

   - class 在语法上更加贴合面向对象的写法
   - class 实现继承更加易读、易理解
   - 更易于写Java等后端语言的使用
   - 本质还是语法糖，使用 prototype

### Promise

```js
function loadImg(src, callback, fail) {
    var img = document.createElement('img')
    img.onload = function () {
        callback(img)
    }
    img.onerror = function () {
        fail()
    }
    img.src = src
}

var src= 'http://www.imooc.com/static/img/index/logo_new.png'
loadImg(src, function (img) {
    console.log(img.width)
}, function () {
    console.log('failed')
})
```



```js
function loadImg(src) {
    return new Promise(function (resolve, reject) {
        var img = document.createElement('img')
        img.onload = function () {
            resolve(img)
        }
        img.onerror = function () {
            reject()
        }
        img.src= src
    })
}
var src = 'http://www.imooc.com/static/img/index/logo_new.png'
var result = loadImg(src)
result.then(function (img) {
    console.log(img.width)
}, function () {
    console.log('failed')
})
result.then(function (img) {
    console.log(img.height)
})
```

### 其他常用功能

1. let/const

   ```js
   // js
   var i = 10
   i = 100
   
   // es6
   let i = 10
   i = 100
   const j = 20
   j = 200 // 报错
   ```

   

2. 多行字符串/模板变量

   ```js
   // js
   var name = 'an'
   var html + = '<div>'
   html += name
   html += '</div>'
   
   //es6
   var name = 'an'
   const html = `<div>${name}</div>`
   ```

   

3. 解构赋值

   ```js
   //js
   var obj = {a:1, b: 2}
   obj.a
   var arr = [1, 2, 3]
   arr[0]
   
   //es6
   const obj = {a:1, b: 2}
   const {a, b} = obj
   a
   const arr = [1, 2, 3]
   const [a, b, c] = arr
   x
   ```

   

4. 块级作用域

   ```js
   //js
   var obj = {a: 1, b: 2}
   for (var item in obj) {
       console.log(item)
   }
   console.log(item) // output: b
   
   // es6
   const obj = {a: 1, b: 2}
   for (let item in obj) {
       console.log(item)
   }
   console.log(item) // output: undefined
   ```

   

5. 函数默认参数

   ```js
   // js
   function(a, b) {
       if (b == null) b = 0
   }
   
   // es6
   function(a, b=0) {}
   ```

   

6. 箭头函数

   ```js
   //js
   var arr = [1, 2, 3]
   arr.map(function(item) {
       return item + 1
   })
   
   // es6
   const arr = [1, 2, 3]
   arr.map(item =>item + 1)
   arr.map((item, index) => {
       console.log(index)
       return item +1
   })
   ```

   ```js
   function fun() {
       console.log(this) // output: {a: 100}
       var arr = [1, 2, 3]
       // js
       arr.map(function(item) {
           console.log(this) // output: window
       	return item + 1
       })
       // 箭头函数:对普通JS的补充
       arr.map(item =>{
           console.log(this) // output: {a: 100}
           return item + 1
       })
   }
   fun.call({a: 100})
   ```

   

