### 引言

`JS `系列暂定 27 篇，从基础，到原型，到异步，到设计模式，到架构模式等，

本篇是  `JS `系列中最重要的一章，花费 3 分钟即可理解，如果你已了解，快速浏览即可。

本篇从对象、构造函数开始，讲到原型与原型链，从基础开始，加实例进一步说明，帮助理解原型与原型链，并深入探讨了 Symbol 、new 。



### 一、基础入门

#### 1. 对象

在JS中，万物皆对象，对象又分为普通对象和函数对象，其中 Object、Function 为 JS 自带的函数对象。

```js
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

**凡是通过 `new Function()` 创建的对象都是函数对象，其他的都是普通对象**。`fun1`、`fun2` 归根结底都是通过 `new Function()`的方式进行创建的。Function Object 也都是通过 `New Function()` 创建的。



#### 2. 构造函数

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



#### 3. 构造函数扩展

- `var a = {}` 其实是 `var a = new Object()` 的语法糖
- `var a = [] `其实是 `var a = new Array()` 的语法糖
- `function Foo(){ ... }` 其实是 `var Foo = new Function(...)`
- **可以使用 `instanceof` 判断一个函数是否为一个变量的构造函数**



#### 4. Symbol 是构造函数吗？

Symbol 是基本数据类型，它并不是构造函数，因为它不支持 `new Symbol()` 语法。我们直接使用`Symbol()` 即可。

```js
var an = Symbol("An");
var an1 = new Symbol("An"); // TypeError
```

但是，`Symbol()` 可以获取到它的 constructor 属性

```js
Symbol("An").constructor; 
// ƒ Symbol() { [native code] }
```

这个 `constructor` 实际上是 Symbol 原型上的，即

```js
Symbol.prototype.constructor; 
// ƒ Symbol() { [native code] }
```

对于 Symbol，你还需要了解以下知识点：



##### `Symbol()` 返回的 symbol **值是唯一**的

```js
Symbol("An") === Symbol("An"); 
// false
```


##### 可以通过 `Symbol.for(key)` 获取全局唯一的 symbol

```js
Symbol.for('An') === Symbol.for("An"); // true
```

它从运行时的 symbol 注册表中找到对应的 symbol，如果找到了，则返回它，否则，新建一个与该键关联的 symbol，并放入全局 symbol 注册表中。



##### Symbol.iterator ：返回一个对象的迭代器

```js
// 实现可迭代协议，使迭代器可迭代：Symbol.iterator
function createIterator(items) {
    var i = 0
    return {
        next: function () {
            var done = (i >= items.length)
            var value = !done ? items[i++] : undefined
            return {
                done: done,
                value: value
            }
        }
        [Symbol.iterator]: function () {
        	return this
    	}
    }
}
var iterator = createIterator([1, 2, 3])
...iterator		// 1, 2, 3
```


##### Symbol.toPrimitive：将对象转换成基本数据类型

```js
// Symbol.toPrimitive 来实现拆箱操作（ES6 之后）
var obj = {
    valueOf: () => {console.log("valueOf"); return {}},
    toString: () => {console.log("toString"); return {}}
}
obj[Symbol.toPrimitive] = () => {console.log("toPrimitive"); return "hello"}
console.log(obj) // toPrimitive  hello
```


##### Symbol.toStringTag：用于设置对象的默认描述字符串值

```js
// Symbol.toStringTag 代替 [[class]] 属性（ES5开始）
var o = { [Symbol.toStringTag]: "MyObject" }

console.log(o + ""); 
// [object MyObject]
```


#### 5. constructor 的值是只读的吗？

**对于引用类型来说 `constructor` 属性值是可以修改的，但是对于基本类型来说是只读的。**

##### 引用类型

```js
function An() {
    this.value = "An";
};
function Anran() {};

Anran.prototype.constructor = An; 
// 原型链继承中，对 constructor 重新赋值

var anran = new Anran(); 
// 创建 Anran 的一个新实例

console.log(anran);
```
<img width="502" alt="constructor" src="https://user-images.githubusercontent.com/19721451/56793881-93002780-683f-11e9-8bd8-0a6e166e5813.png">

这说明，依赖一个 引用对象的 constructor 属性，并不是安全的。

##### 基本类型

```js
function An() {};
var an = 1;
an.constructor = An;
console.log(an.constructor); 
// ƒ Number() { [native code] }
```

这是因为：**原生构造函数（`native constructors`）是只读的**。

注意：**`null` 和 `undefined` 是没有 `constructor` 属性的**。



### 二、原型

首先，贴上

![原型](https://img-blog.csdnimg.cn/20181212155756420.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1bmFoYWlqaWFv,size_16,color_FFFFFF,t_70)

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



#### 1. prototype

所有函数都有一个 `prototype` （显式原型）属性，属性值也是一个普通的对象。对象以其原型为模板，从原型继承方法和属性，这些属性和方法定义在对象的构造器函数的 `prototype` 属性上，而非对象实例本身。

但有一个例外： `Function.prototype.bind()`，它并没有 prototype 属性

```js
let fun = Function.prototype.bind(); 
// ƒ () { [native code] }
```

当我们创建一个函数时，例如

```js
function Foo () {}
```
<img width="436" alt="FOO" src="https://user-images.githubusercontent.com/19721451/56793854-7fed5780-683f-11e9-88d8-c1af1414d57a.png">

`prototype`属性就被自动创建了

从上面这张图可以发现，`Foo` 对象有一个原型对象 `Foo.prototype`，其上有两个属性，分别是 `constructor` 和 `__proto__`，其中 `__proto__` 已被弃用。

构造函数 `Foo` 有一个指向原型的指针，原型 `Foo.prototype` 有一个指向构造函数的指针 `Foo.prototype.constructor`，这就是一个**循环引用**，即：

```js
Foo.prototype.constructor === Foo; // true
```



#### 2. `__proto__`

每个实例对象（object ）都有一个隐式原型属性（称之为`__proto__`）指向了创建该对象的构造函数的原型。也就时指向了函数的 `prototype`属性。

```js
function Foo () {}
let foo = new Foo()
```
<img width="393" alt="Foo1" src="https://user-images.githubusercontent.com/19721451/56793821-6c41f100-683f-11e9-941a-2734defd42f9.png">

当`new Foo()`时，`__proto__`自动创建了。并且

```js
foo.__proto__ === Foo.prototype; // true
```

#### 注意

`__proto__` 属性在 `ES6` 时才被标准化，以确保 Web 浏览器的兼容性，但是不推荐使用，除了标准化的原因之外还有性能问题。为了更好的支持，推荐使用 `Object.getPrototypeOf()`。

>  通过现代浏览器的操作属性的便利性，可以改变一个对象的 `[[Prototype]]` 属性, 这种行为在每一个JavaScript引擎和浏览器中都是一个非常慢且影响性能的操作，使用这种方式来改变和继承属性是对性能影响非常严重的，并且性能消耗的时间也不是简单的花费在 `obj.__proto__ = ...` 语句上, 它还会影响到所有继承来自该 `[[Prototype]]` 的对象，如果你关心性能，你就不应该在一个对象中修改它的 [[Prototype]]。相反, 创建一个新的且可以继承 `[[Prototype]]` 的对象，推荐使用 [`Object.create()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)。

如果要读取或修改对象的 `[[Prototype]]` 属性，建议使用如下方案，但是此时设置对象的 `[[Prototype]]` 依旧是一个缓慢的操作，如果性能是一个问题，就要避免这种操作。

```Js
// 获取（两者一致）
Object.getPrototypeOf()
Reflect.getPrototypeOf()

// 修改（两者一致）
Object.setPrototypeOf()
Reflect.setPrototypeOf()
```

如果要创建一个新对象，同时继承另一个对象的  `[[Prototype]]` ，推荐使用 `Object.create()`。

```Js
function An() {};
var an = new An();
var anran = Object.create(an);
```

这里 `anran` 是一个新的空对象，有一个指向对象 `an` 的指针 `__proto__`。



#### 3. new 的实现过程

- 新生成了一个对象

- 链接到原型

- 绑定 this

- 返回新对象

```js
function new_object() {
  // 创建一个空的对象
  let obj = new Object()
  // 获得构造函数
  let Con = [].shift.call(arguments)
  // 链接到原型 （不推荐使用）
  obj.__proto__ = Con.prototype
  // 绑定 this，执行构造函数
  let result = Con.apply(obj, arguments)
  // 确保 new 出来的是个对象
  return typeof result === 'object' ? result : obj
}
```



##### 优化 new 实现

```js
// 优化后 new 实现
function create() {
  // 1、获得构造函数，同时删除 arguments 中第一个参数
  Con = [].shift.call(arguments);
  // 2、创建一个空的对象并链接到原型，obj 可以访问构造函数原型中的属性
  var obj = Object.create(Con.prototype);
  // 3、绑定 this 实现继承，obj 可以访问到构造函数中的属性
  var ret = Con.apply(obj, arguments);
  // 4、优先返回构造函数返回的对象
  return ret instanceof Object ? ret : obj;
};
```



#### 4. 总结

- 所有的引用类型（数组、对象、函数）都有对象特性，即可自由扩展属性（null除外）。
- 所有的引用类型，都有一个 `__proto__` （隐式原型）属性，属性值是一个普通的对象，该原型对象也有一个自己的原型对象(`__proto__`) ，层层向上直到一个对象的原型对象为 `null`。根据定义，`null` 没有原型，并作为这个**原型链**中的最后一个环节。
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



### 三、原型链

每个实例对象（ object ）都有一个私有属性（称之为 __proto__ ）指向它的原型对象（ **prototype** ）。该原型对象也有一个自己的原型对象( __proto__ ) ，层层向上直到一个对象的原型对象为 `null`，这就是**原形链**。根据定义，`null` 没有原型，并作为这个**原型链**中的最后一个环节。

原型链的基本思想是利用原型，让一个引用类型继承另一个引用类型的属性及方法。

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

也就是

```js
Function.__proto__.__proto__ === Object.prototype
```

![原型链](https://img-blog.csdnimg.cn/20181212155939943.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1bmFoYWlqaWFv,size_16,color_FFFFFF,t_70)

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
暂时就这些，后续我将持续更新

