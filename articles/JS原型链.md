## JS原型与原型链

### 对象

在JS中，万物皆对象，对象又分为普通对象和函数对象，其中Object、Function为JS自带的函数对象。

```Js
var obj1 = {}; 
var obj2 = new Object();
var obj3 = new fun1()

function fun1(){}; 
var fun2 = function(){};
var fun3 = new Function('some','console.log(some)');

// JS自带的函数对象
console.log(typeof Object); //function 
console.log(typeof Function); //function  

// 普通对象
console.log(typeof obj1); //object 
console.log(typeof obj2); //object 
console.log(typeof obj3); //object

// 函数对象
console.log(typeof f1); //function 
console.log(typeof f2); //function 
console.log(typeof f3); //function   
```

凡是通过 `new Function()` 创建的对象都是函数对象，其他的都是普通对象。`fun1`、`fun2`归根结底都是通过 `new Function()`的方式进行创建的。Function Object 也都是通过 `New Function()` 创建的。

### 构造函数

```js
function Foo(name, age) {
    // this 默认为空
    this.name = name
    this.age = age
    this.class = 'class'
    // return this // 默认有这一行
}

// Foo 的实例
var f = new Foo('aa', 20)
```

每个实例都有一个`constructor`（构造函数）属性，该属性指向对象本身。

```js
f.constructor === Foo // true
```

JS本身不提供一个`class`实现。（在 ES2015/ES6 中引入了`class`关键字，但只是语法糖，JavaScript 仍然是基于原型的）。

### 构造函数扩展

- `var a = {}` 其实是 `var a = new Object()` 的语法糖
- `var a = [] `其实是 `var a = new Array()` 的语法糖
- `function Foo(){ ... }` 其实是 `var Foo = new Function(...)`
- 可以使用 `instanceof` 判断一个函数是否为一个变量的构造函数

### 原型规则与示例

首先，贴上

![figure1](/Users/a123/Desktop/Study/JS/img/figure1.jpg)

图片来自于http://www.mollypages.org/tutorials/js.mp，请根据下文仔细理解这张图

在JS中，每个对象都有自己的原型。当我们访问对象的属性和方法时，JS会先访问对象本身的方法和属性。如果对象本身不包含这些属性和方法，则访问对象对应的原型。

```js
// 构造函数
function Foo(name) {
    this.name = name
}
Foo.prototype.alertName = function() {
    alert(this.name)
}
// 创建实例
var f = new Foo('some')
f.printName = function () {
    console.log(this.name)
}
// 测试
f.printName()// 对象的方法
f.alertName()// 原型的方法
```

1. prototype

   所有函数都有一个 `prototype` （显式原型）属性，属性值也是一个普通的对象。但有一个例外： `Function.prototype.bind()`

   ```js
   let fun = Function.prototype.bind()// fun没有prototype属性
   ```

   当我们创建一个函数时，例如

   ```js
   function Foo () {}
   ```

   ![prototype](/Users/a123/Desktop/Study/JS/img/prototype.png)

   `prototype`属性就被自动创建了

   我们访问`fn.prototype`，返回了一个对象，该对象里只有一个属性`constructor`

2. `__proto__`

   每个实例对象（object ）都有一个隐式原型属性（称之为`__proto__`）指向了创建该对象的构造函数的原型。也就时指向了函数的 `prototype`属性。

   ```js
   var obj = {}; obj.a = 100;
   var arr = []; arr.a = 100;
   function fn () {}
   fn.a = 100;
   
   console.log(obj.__proto__)
   console.log(arr.__proto__)
   console.log(fn.__proto__)
   
   console.log(fn.prototype) 
   
   console.log(obj.__proto__ === Object.prototype) // true
   ```
   ![proto](/Users/a123/Desktop/Study/JS/img/proto.png)

   当`new Foo()`时，`__proto__`自动创建了。

   new的实现过程

   - 新生成了一个对象

   - 链接到原型

   - 绑定 this

   - 返回新对象

     ```
     function new_object() {
         // 创建一个空的对象
         let obj = new Object()
         // 获得构造函数
         let Con = [].shift.call(arguments)
         // 链接到原型
         obj.__proto__ = Con.prototype
         // 绑定 this，执行构造函数
         let result = Con.apply(obj, arguments)
         // 确保 new 出来的是个对象
         return typeof result === 'object' ? result : obj
     }
     ```

   3. 总结

   - 所有的引用类型（数组、对象、函数）都有对象特性，即可自由扩展属性（null除外）。
   - 所有的引用类型，都有一个 `__proto__` （隐式原型）属性，属性值是一个普通的对象，该原型对象也有一个自己的原型对象(`__proto__`) ，层层向上直到一个对象的原型对象为 `null`。根据定义，`null` 没有原型，并作为这个**原型链**中的最后一个环节。
   - 当试图得到一个对象的某个属性时，如果这个对象本身没有这个属性，那么会去它的`__proto__`(即它的构造函数`prototype`)中寻找。

   ```js
   // 构造函数
   function Foo(name) {
       this.name = name
   }
   Foo.prototype.alertName = function() {
       alert(this.name)
   }
   // 创建实例
   var f = new Foo('some')
   f.printName = function () {
       console.log(this.name)
   }
   // 测试
   f.printName()
   f.alertName()
   // 循环对象自身的属性
   var item
   for (item in f) {
       // 高级浏览器已经在 for in 中屏蔽了来自原型的属性
       // 但是这里还是建议大家还是加上这个判断，保证程序的健壮性
       if (f.hasOwnPrototype(item)) {
           console.log(item)
       }
   }
   ```

   

###原型链

```Js
// 构造函数
function Foo(name) {
    this.name = name
}
// 创建实例
var f = new Foo('some')
// 测试
f.toString() // f.__proto__.__proto__中寻找
```

`f.__proto__=== Foo.prototype`，`Foo.prototype` 也是一个对象，也有自己的`__proto__` 指向 `Object.prototype`， 找到`toString()`方法。

![image-20181212102359024](/Users/a123/Desktop/Study/JS/img/toString()原型链.png)

下面是原型链继承的例子

```js
function Elem(id) {
    this.elem = document.getElementById(id)
}

Elem.prototype.html = function(val) {
    var elem = this.elem
    if (val) {
        elem.innerHtml = val
        return this // 链式操作
    } else {
        return elem.innerHtml
    }
}

Elem.prototype.on = function( type, fn) {
    var elem = this.elem
    elem.addEventListener(type, fn)
}

var div1 = new Elem('div1')
// console.log(div1.html())
div1.html('<p>hello</p>').on('click', function() {
    alert('clicked')
})// 链式操作
```



## instanceof

在JS中，判断一个变量的类型，常常会用到`typeof`运算符，但当用来判断引用类型变量时，无论是什么类型的变量，它都会返回`Object`。为此，引入了`instanceof`。

`instanceof`相比与`typeof`来说，`instanceof`方法要求开发者明确的确认对象为某特定类型。即`instanceof`用于判断引用类型属于哪个构造函数的方法。

```js
var arr = []
arr instanceof Array // true
typeof arr // object, typeof 是无法判断是否为数组的
```

另外，更重的一点是 instanceof 可以在继承关系中用来判断一个实例是否属于它的父类型。

```js
// 判断 f 是否是 Foo 类的实例 , 并且是否是其父类型的实例
function Aoo(){} 
function Foo(){} 
Foo.prototype = new Aoo();//JavaScript 原型继承
 
var f = new Foo(); 
console.log(foo instanceof Foo)//true 
console.log(foo instanceof Aoo)//true
```

`f instanceof Foo` 的判断逻辑是：

- f 的 `__proto__`一层一层往上，是否对应到 `Foo.prototype`
- 再往上，看是否对应着`Aoo.prototype`
- 再试着判断 `f instanceof Object`

即`instanceof`可以用于判断多层继承关系。

下面看一组复杂例子

```js
console.log(Object instanceof Object) //true 
console.log(Function instanceof Function) //true 
console.log(Number instanceof Number) //false 
console.log(String instanceof String) //false 
console.log(Array instanceof Array) // false
 
console.log(Function instanceof Object) //true 
 
console.log(Foo instanceof Function) //true 
console.log(Foo instanceof Foo) //false
```

在这组数据中，Object、Function instanceof 自己为true， 其他的instanceof自己都为false，这就要从instanceof的内部实现机制以及JS原型继承机制讲起。

1. instanceof的内部实现机制

   instanceof的内部实现机制是通过判断对象的原型链上是否能找到对象的 `prototype`

   ```js
   // instanceof 的内部实现 
   function instance_of(L, R) {//L 表示左表达式，R 表示右表达式，即L为变量，R为类型
    // 取 R 的显示原型
    var prototype = R.prototype
    // 取 L 的隐式原型
    L = L.__proto__
    // 判断对象（L）的类型是否严格等于类型（R）的显式原型
    while (true) { 
      if (L === null) 
        return false
      if (prototype === L)// 这里重点：当 prototype 严格等于 L 时，返回 true 
        return true
      L = L.__proto__
    } 
   }
   ```

2. JS原型链继承关系

   ![figure1](/Users/a123/Desktop/Study/JS/img/figure1.jpg)

   图片来自于http://www.mollypages.org/tutorials/js.mp

   由其本文涉及显示原型和隐式原型，所以下面对这两个概念作一下简单说明。在 JavaScript 原型继承结构里面，规范中用 [[Prototype]] 表示对象隐式的原型，在 JavaScript 中用 `__proto__` 表示，并且在 Firefox 和 Chrome 浏览器中是可以访问得到这个属性的，但是 IE 下不行。所有 JavaScript 对象都有 __proto__ 属性，但只有 `Object.prototype.__proto__` 为 null，前提是没有在 Firefox 或者 Chrome 下修改过这个属性。这个属性指向它的原型对象。 至于显示的原型，在 JavaScript 里用 prototype 属性表示，这个是 JavaScript 原型继承的基础知识，在这里就不在叙述了。

   下面介绍几个例子，加深你的理解：

   - Object instanceof Object

     ```js
     // 为了方便表述，首先区分左侧表达式和右侧表达式
     ObjectL = Object, ObjectR = Object; 
     // 下面根据规范逐步推演
     O = ObjectR.prototype = Object.prototype 
     L = ObjectL.__proto__ = Function.prototype 
     // 第一次判断
     O != L 
     // 循环查找 L 是否还有 __proto__ 
     L = Function.prototype.__proto__ = Object.prototype 
     // 第二次判断
     O === L 
     // 返回 true
     ```

   - Function instanceof Function

     ```js
     // 为了方便表述，首先区分左侧表达式和右侧表达式
     FunctionL = Function, FunctionR = Function; 
     // 下面根据规范逐步推演
     O = FunctionR.prototype = Function.prototype 
     L = FunctionL.__proto__ = Function.prototype 
     // 第一次判断
     O === L 
     // 返回 true
     ```

   - Foo instanceof Foo

     ```js
     // 为了方便表述，首先区分左侧表达式和右侧表达式
     FooL = Foo, FooR = Foo; 
     // 下面根据规范逐步推演
     O = FooR.prototype = Foo.prototype 
     L = FooL.__proto__ = Function.prototype 
     // 第一次判断
     O != L 
     // 循环再次查找 L 是否还有 __proto__ 
     L = Function.prototype.__proto__ = Object.prototype 
     // 第二次判断
     O != L 
     // 再次循环查找 L 是否还有 __proto__ 
     L = Object.prototype.__proto__ = null 
     // 第三次判断
     L == null 
     // 返回 false
     ```

     

   本文参考https://www.ibm.com/developerworks/cn/web/1306_jiangjj_jsinstanceof/



