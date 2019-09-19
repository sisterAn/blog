### 引言

`JS `系列暂定 27 篇，从基础，到原型，到异步，到设计模式，到架构模式等。

本篇是`JS`系列中第 5 篇，文章主讲 JS 中 `call` 、 `apply` 、 `bind` 、箭头函数以及柯里化，着重介绍它们之间的区别、对比使用，深入了解 `call` 、 `apply` 、 `bind` 。



### 一、Function.prototype.call()

`call()` 方法调用一个函数, 其具有一个指定的 `this` 值和多个参数(**参数的列表**)。

```js
func.call(thisArg, arg1, arg2, ...)
```

它运行 `func`，提供的第一个参数 `thisArg` 作为 `this`，后面的作为参数。



#### 1. func 与 func.call

先看一个例子：

```js
func(1, 2, 3);
func.call(obj, 1, 2, 3)
```

他们都调用的是 `func`，参数是 `1`，`2` 和 `3`。

唯一的区别是 `func.call` 也将 `this` 设置为 `obj`。

需要注意的是，设置的 thisArg 值并不一定是该函数执行时真正的 `this` 值，如果这个函数处于非严格模式下，则指定为 `null` 和 `undefined` 的 `this` 值会自动指向全局对象(浏览器中就是 window 对象)，同时值为原始值(数字，字符串，布尔值)的 `this` 会指向该原始值的自动包装对象。



#### 2. func.call 绑定上下文

例如，在下面的代码中，我们在对象的上下文中调用 `sayWord.call(bottle)` 运行 `sayWord` ，并 `bottle` 传递为 `sayWord` 的 `this`：

```js
function sayWord() {
  var talk = [this.name, 'say', this.word].join(' ');
  console.log(talk);
}

var bottle = {
  name: 'bottle', 
  word: 'hello'
};

// 使用 call 将 bottle 传递为 sayWord 的 this
sayWord.call(bottle); 
// bottle say hello
```



#### 3. 使用 func.call 时未指定 this

##### 非严格模式

```js
// 非严格模式下
var bottle = 'bottle'
function say(){
   // 注意：非严格模式下，this 为 window
   console.log('name is %s',this.bottle)
}

say.call()
// name is bottle
```

##### 严格模式

```js
// 严格模式下
'use strict'
var bottle = 'bottle'
function say(){
   // 注意：在严格模式下 this 为 undefined
   console.log('name is %s',this.bottle)
}

say.call()
// Uncaught TypeError: Cannot read property 'bottle' of undefined
```



#### 4. call 在 JS 继承中的使用: 构造继承

**基本思想**：在子类型的构造函数内部调用父类型构造函数。

**注意**：函数只不过是在特定环境中执行代码的对象，所以这里使用 apply/call 来实现。

**使用父类的构造函数来增强子类实例，等于是复制父类的实例属性给子类（没用到原型）**

```JS
// 父类
function SuperType (name) {
  this.name = name; // 父类属性
}
SuperType.prototype.sayName = function () { // 父类原型方法
  return this.name;
};

// 子类
function SubType () {
  // 调用 SuperType 构造函数
  // 在子类构造函数中，向父类构造函数传参
  SuperType.call(this, 'SuperType'); 
  // 为了保证子父类的构造函数不会重写子类的属性，需要在调用父类构造函数后，定义子类的属性
  this.subName = "SubType"; 
  // 子类属性
};

// 子类实例
let instance = new SubType(); 
// 运行子类构造函数，并在子类构造函数中运行父类构造函数，this绑定到子类
```



#### 5. 解决 var 作用域问题

```js
var bottle = [
  {name: 'an', age: '24'},
  {name: 'anGe', age: '12'}
];

for (var i = 0; i < bottle.length; i++) {
  // 匿名函数
  (function (i) { 
    setTimeout(() => {
      // this 指向了 bottle[i]
      console.log('#' + i  + ' ' + this.name + ': ' + this.age); 
    }, 1000)
  }).call(bottle[i], i);
  // 调用 call 方法，同时解决了 var 作用域问题
}
```

打印结果：

```js
#0 an: 24
#1 anGe: 12
```

在上面例中的 `for` 循环体内，我们创建了一个匿名函数，然后通过调用该函数的 `call` 方法，将每个数组元素作为指定的 `this` 值立即执行了那个匿名函数。这个立即执行的匿名函数的作用是打印出 `bottle[i]` 对象在数组中的正确索引号。



### 二、Function.prototype.apply()

`apply()` 方法调用一个具有给定 `this` 值的函数，以及作为一个数组（或[类似数组对象）提供的参数。

```js
func.apply(thisArg, [argsArray])
```

它运行 `func` 设置 `this = context` 并使用类数组对象 `args` 作为参数列表。

例如，这两个调用几乎相同：

```js
func(1, 2, 3);
func.apply(context, [1, 2, 3])
```

两个都运行 `func` 给定的参数是 `1,2,3`。但是 `apply` 也设置了 `this = context`。

`call` 和 `apply` 之间唯一的语法区别是 `call` 接受一个参数列表，而 `apply` 则接受带有一个类数组对象。

需要注意：Chrome 14 以及 Internet Explorer 9 仍然不接受类数组对象。如果传入类数组对象，它们会抛出异常。



#### 1. call、apply 与 扩展运算符 

我们已经知道了[JS 基础之:  var、let、const、解构、展开、函数](https://github.com/sisterAn/blog/issues/48)  一章中的扩展运算符 `...`，它可以将数组（或任何可迭代的）作为参数列表传递。因此，如果我们将它与 `call` 一起使用，就可以实现与 `apply` 几乎相同的功能。

这两个调用结果几乎相同：

```js
let args = [1, 2, 3];

func.call(context, ...args); // 使用 spread 运算符将数组作为参数列表传递
func.apply(context, args);   // 与使用 call 相同
```

如果我们仔细观察，那么 `call` 和 `apply` 的使用会有一些细微的差别。

- 扩展运算符 `...` 允许将 **可迭代的** `参数列表` 作为列表传递给 `call`。
- `apply` 只接受 **类数组一样的** `参数列表`。 



#### 2. apply 函数转移

`apply` 最重要的用途之一是将调用传递给另一个函数，如下所示：

```js
let wrapper = function() {
  return anotherFunction.apply(this, arguments);
};
```

`wrapper` 通过 `anotherFunction.apply` 获得了上下文 `this` 和 `anotherFunction` 的参数并返回其结果。

当外部代码调用这样的 `wrapper` 时，它与原始函数的调用无法区分。



#### 3. apply 连接数组

 `array.push.apply` 将数组添加到另一数组上：

```js
var array = ['a', 'b']
var elements = [0, 1, 2]
array.push.apply(array, elements)
console.info(array) // ["a", "b", 0, 1, 2]
```



#### 4. apply 来链接构造器

```js
Function.prototype.constructor = function (aArgs) {
  var oNew = Object.create(this.prototype);
  this.apply(oNew, aArgs);
  return oNew;
};
```



#### 5. apply 和内置函数

```js
/* 找出数组中最大/小的数字 */
let numbers = [5, 6, 2, 3, 7]
/* 应用(apply) Math.min/Math.max 内置函数完成 */

let max = Math.max.apply(null, numbers) 
/* 基本等同于 Math.max(numbers[0], ...) 或 Math.max(5, 6, ..) */

let min = Math.min.apply(null, numbers)

console.log('max: ', max)
// max: 7
console.log('min: ', min)
// min: 2
```

它相当于：

```js
/* 代码对比： 用简单循环完成 */
let numbers = [5, 6, 2, 3, 7]
let max = -Infinity, min = +Infinity
for (var i = 0; i < numbers.length; i++) {
  if (numbers[i] > max)
    max = numbers[i]
  if (numbers[i] < min) 
    min = numbers[i]
}

console.log('max: ', max)
// max: 7
console.log('min: ', min)
// min: 2
```

但是：如果用上面的方式调用 `apply`，会有超出 JavaScript 引擎的参数长度限制的风险。更糟糕的是其他引擎会直接限制传入到方法的参数个数，导致参数丢失。

所以，当数据量较大时

```js
function minOfArray(arr) {
  var min = Infinity
  var QUANTUM = 32768 // JavaScript 核心中已经做了硬编码  参数个数限制在65536

  for (var i = 0, len = arr.length; i < len; i += QUANTUM) {
    var submin = Math.min.apply(null, arr.slice(i, Math.min(i + QUANTUM, len)))
    min = Math.min(submin, min)
  }
  return min
}
var min = minOfArray([5, 6, 2, 3, 7])
// max 同样也是如此
```



### 三、Function.prototype.bind()

JavaScript 新手经常犯的一个错误是将一个方法从对象中拿出来，然后再调用，希望方法中的 `this` 是原来的对象（比如在回调中传入这个方法）。如果不做特殊处理的话，一般 `this` 就丢失了。

 例如：

```js
let bottle = {
  nickname: "bottle",
  sayHello() {
    console.log(`Hello, ${this.nickname}!`)
  },
  sayHi(){
    setTimeout(function(){
      console.log('Hello, ', this.nickname)
    }, 1000)
  }
};

// 问题一
bottle.sayHi();
// Hello, undefined!

// 问题二
setTimeout(bottle.sayHello, 1000); 
// Hello, undefined!
```

问题一的 this.nickname 是 undefined ，原因是 this 指向是在运行函数时确定的，而不是定义函数时候确定的，再因为 sayHi  中 setTimeout 在全局环境下执行，所以 this 指向 setTimeout 的上下文：window。

问题二的 this.nickname 是 undefined ，是因为 setTimeout 仅仅只是获取函数 bottle.sayHello 作为 setTimeout 回调函数，this 和 bottle 对象分离了。

问题二可以写为：

```js
// 在这种情况下，this 指向全局作用域
let func = bottle.sayHello;
setTimeout(func, 1000); 
// 用户上下文丢失
// 浏览器上，访问的实际上是 Window 上下文
```

那么怎么解决这两个问题喃？

**解决方案一： 缓存 this 与包装**

首先通过缓存 this 解决问题一 `bottle.sayHi();` ：

```js
let bottle = {
  nickname: "bottle",
  sayHello() {
    console.log(`Hello, ${this.nickname}!`)
  },
  sayHi(){
    var _this = this // 缓存this
    setTimeout(function(){
      console.log('Hello, ', _this.nickname)
    }, 1000)
  }
};

bottle.sayHi();
// Hello,  bottle
```

那问题二 `setTimeout(bottle.sayHello, 1000); ` 喃？

```js
let bottle = {
  nickname: "bottle",
  sayHello() {
    console.log(`Hello, ${this.nickname}!`);
  }
};

// 加一个包装层
setTimeout(() => {
  bottle.sayHello()
}, 1000); 
// Hello, bottle!
```

这样看似解决了问题二，但如果我们在 `setTimeout` 异步触发之前更新 `bottle` 值又会怎么样呢？

```js
var bottle = {
  nickname: "bottle",
  sayHello() {
    console.log(`Hello, ${this.nickname}!`);
  }
};

setTimeout(() => {
  bottle.sayHello()
}, 1000); 

// 更新 bottle
bottle = {
  nickname: "haha",
  sayHello() {
    console.log(`Hi, ${this.nickname}!`)
  }
};
// Hi, haha!
```

`bottle.sayHello()` 最终打印为 `Hi, haha!` ，那么怎么解决这种事情发生喃？

**解决方案二： bind**

`bind()` 最简单的用法是创建一个新绑定函数，当这个新绑定函数被调用时，`this` 键值为其提供的值，其参数列表前几项值为创建时指定的参数序列，绑定函数与被调函数具有相同的函数体（ES5中）。

```js
let bottle = {
  nickname: "bottle",
  sayHello() {
    console.log(`Hello, ${this.nickname}!`);
  }
};

// 未绑定，“this” 指向全局作用域
let sayHello = bottle.sayHello
console.log(sayHello())
// Hello, undefined!

// 绑定
let bindSayHello = sayHello.bind(bottle)
// 创建一个新函数，将 this 绑定到 bottle 对象
console.log(bindSayHello())
// Hello, bottle!
```

所以，从原来的函数和原来的对象创建一个绑定函数，则能很漂亮地解决上面两个问题：

```js
let bottle = {
  nickname: "bottle",
  sayHello() {
    console.log(`Hello, ${this.nickname}!`);
  },
  sayHi(){
    // 使用 bind
    setTimeout(function(){
      console.log('Hello, ', this.nickname)
    }.bind(this), 1000)
    
    // 或箭头函数
    setTimeout(() => {
      console.log('Hello, ', this.nickname)
    }, 1000)
  }
};

// 问题一：完美解决
bottle.sayHi()
// Hello,  bottle
// Hello,  bottle

let sayHello = bottle.sayHello.bind(bottle); // (*)

sayHello(); 
// Hello, bottle!

// 问题二：完美解决
setTimeout(sayHello, 1000); 
// Hello, bottle!

// 更新 bottle
bottle = {
  nickname: "haha",
  sayHello() {
    console.log(`Hi, ${this.nickname}!`)
  }
};
```

问题一，可以通过 `bind` 或箭头函数完美解决。

最终更新 `bottle` 后， `setTimeout(sayHello, 1000);` 打印依然是 `Hello, bottle!`， 问题二完美解决！



#### 1. bind 与 new

再看一个例子：

```js
this.nickname = 'window'
let bottle = {
  nickname: 'bottle'
}
function sayHello() {
  console.log('Hello, ', this.nickname)
}

let bindBottle = sayHello.bind(bottle) // this 指向 bottle
console.log(bindBottle()) 
// Hello,  bottle

console.log(new bindBottle())  // this 指向 sayHello {}
// Hello,  undefined
```

上面例子中，运行结果 `this.nickname` 输出为 `undefined` ，这不是全局 `nickname` ， 也不是 `bottle` 对象中的 `nickname` ，这说明 `bind` 的 `this` 对象失效了，`new` 的实现中生成一个新的对象，这个时候的 `this` 指向的是 `sayHello` 。

**注意** ：绑定函数也可以使用 `new` 运算符构造：这样做就好像已经构造了目标函数一样。提供的 **this** 值将被忽略，而前置参数将提供给模拟函数。



#### 2. 二次 bind

```js
function sayHello() {
  console.log('Hello, ', this.nickname)
}

sayHello = sayHello.bind( {nickname: "Bottle"} ).bind( {nickname: "AnGe" } );
sayHello();
// Hello,  Bottle
```

输出依然是 `Hello,  Bottle` ，这是因为 `func.bind(...)` 返回的外来的绑定函数对象仅在创建的时候记忆上下文（如果提供了参数）。

一个函数不能作为重复绑定。



#### 2. 偏函数

当我们确定一个函数的一些参数时，返回的函数（更加特定）被称为**偏函数**。我们可以使用 `bind` 来获取偏函数：

```js
function list() {
  return Array.prototype.slice.call(arguments);
}

var list1 = list(1, 2, 3); // [1, 2, 3]

var leadingThirtysevenList = list.bind(undefined, 37);
var list2 = leadingThirtysevenList(); // [37]
var list3 = leadingThirtysevenList(1, 2, 3); // [37, 1, 2, 3]
```

当我们不想一遍又一遍重复相同的参数时，偏函数很方便。



#### 3. 作为构造函数使用的绑定函数

```js
function Bottle(nickname) {
  this.nickname = nickname;
}
Bottle.prototype.sayHello = function() { 
  console.log('Hello, ', this.nickname)
};

let bottle = new Bottle('bottle');
let BindBottle = Bottle.bind(null, 'bindBottle');

let b1 = new BindBottle('b1');
b1 instanceof Bottle; // true
b1 instanceof BindBottle; // true
new Bottle('bottle1') instanceof BindBottle; // true

b1.sayHello()
// Hello,  bindBottle
```



### 四、柯里化

在计算机科学中，柯里化（Currying）是把接受多个参数的函数变换成接受一个单一参数(最初函数的第一个参数)的函数，并且返回接受余下的参数且返回结果的新函数的技术。这个技术由 Christopher Strachey 以逻辑学家 Haskell Curry 命名的，尽管它是 Moses Schnfinkel 和 Gottlob Frege 发明的。

```js
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var increment = add(1);
var addTen = add(10);

increment(2);
// 3

addTen(2);
// 12

add(1)(2);
// 3
```

这里定义了一个 `add` 函数，它接受一个参数并返回一个新的函数。调用 `add` 之后，返回的函数就通过闭包的方式记住了 `add` 的第一个参数。所以说 `bind` 本身也是闭包的一种使用场景。

**柯里化**是将 `f(a,b,c)` 可以被以 `f(a)(b)(c)` 的形式被调用的转化。JavaScript 实现版本通常保留函数被正常调用和在参数数量不够的情况下返回偏函数这两个特性。



### 五、扩展：箭头函数



#### 1. 没有 this

```js
let bottle = {
  nickname: "bottle",
  sayHi(){
    setTimeout(function(){
      console.log('Hello, ', this.nickname)
    }, 1000)
    
    // 或箭头函数
    setTimeout(() => {
      console.log('Hi, ', this.nickname)
    }, 1000)
  }
};

bottle.sayHi()
// Hello,  undefined
// Hi,  bottle
```

报错是因为 `Hello,  undefined` 是因为运行时 `this=Window` ， `Window.nickname`  为 `undefined`。

但箭头函数就没事，因为箭头函数没有 `this`。在外部上下文中，`this` 的查找与普通变量搜索完全相同。`this` 指向定义时的环境。



#### 2. 不可 new 实例化

不具有 `this` 自然意味着另一个限制：箭头函数不能用作构造函数。他们不能用 `new` 调用。



#### 3. 箭头函数 vs bind

箭头函数 `=>` 和正常函数通过 `.bind(this)` 调用有一个微妙的区别：

- `.bind(this)` 创建该函数的 “绑定版本”。
- 箭头函数 `=>` 不会创建任何绑定。该函数根本没有 `this`。在外部上下文中，`this` 的查找与普通变量搜索完全相同。



#### 4. 没有 arguments 对象

箭头函数也没有 `arguments` 变量。

因为我们需要用当前的 `this` 和 `arguments` 转发一个调用，所有这对于装饰者来说非常好。

例如，`defer(f, ms)` 得到一个函数，并返回一个包装函数，以 `毫秒` 为单位延迟调用：

```js
function defer(f, ms) {
  return function() {
    setTimeout(() => f.apply(this, arguments), ms)
  };
}

function sayHi(who) {
  alert('Hello, ' + who);
}

let sayHiDeferred = defer(sayHi, 2000);
sayHiDeferred("John"); // 2 秒后打印 Hello, John
```

没有箭头功能的情况如下所示：

```js
function defer(f, ms) {
  return function(...args) {
    let ctx = this;
    setTimeout(function() {
      return f.apply(ctx, args);
    }, ms);
  };
}
```

在这里，我们必须创建额外的变量 `args` 和 `ctx`，以便 `setTimeout` 内部的函数可以接收它们。



#### 5. 总结

- this 指向定义时的环境
- **不可 new 实例化**
- this 不可变
- **没有 arguments 对象**
