### 一、Function.prototype.call()

`call()` 方法调用一个函数, 其具有一个指定的 `this` 值和分别地提供的参数(**参数的列表**)。

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

需要注意的是，指定的 `this` 值并不一定是该函数执行时真正的 `this` 值，如果这个函数处于非严格模式下，则指定为 `null` 和 `undefined` 的 `this` 值会自动指向全局对象(浏览器中就是 window 对象)，同时值为原始值(数字，字符串，布尔值)的 `this` 会指向该原始值的自动包装对象。



#### 2. func.call 使用

例如，在下面的代码中，我们在对象的上下文中调用 `Bottle.call(bottle)` 运行 `Bottle` ，并 `bottle` 传递为 `Bottle` 的 `this`：

```js
function Bottle() {
  var hello = [this.name, 'say', this.word].join(' ');
  console.log(hello);
}

var bottle = {
  name: 'bottle', word: 'hello'
};

// 使用 call 将 bottle 传递为 Bottle 的 this
Bottle.call(bottle); 
// bottle say hello
```



#### 3. 使用 func.call 未指定 this

##### 非严格模式

```js
// 非严格模式下
var bottle = 'bottle'
function say(){
   // 注意：非严格模式下，this 为 Window
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



#### 4. call 在 JS 继承中的使用

```JS
function Bottle(name, time) {
  this.name = name
  if (time < 6) {
    throw RangeError(
      this.name + ' is sleep'
    );
  }
}

function SubBottle(name, time) {
  // 使用 call 调用父类的构造函数
  Bottle.call(this, name, time);
  // 定义子类属性
  this.category = 'subBottle';
}

//等同于
function LittleBottle(name, time) {
  
  // 这部分类是 Bottle.call(this, name, time); 的执行效果
  this.name = name;
  this.time = time;
  if (time < 6) {
    throw RangeError(
      this.name + ' is sleep'
    );
  }
    
  // 定义子类属性
  this.category = 'littleBottle';
}

// 实例对象
var subBottle = new SubBottle('subBottle', 5);
// Uncaught RangeError: subBottle is sleep

var littleBottle = new LittleBottle('littleBottle', 5);
// Uncaught RangeError: littleBottle is sleep
```



#### 5. 解决 var 作用域

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

```Js
func.apply(thisArg, [argsArray])
```

它运行 `func` 设置 `this=context` 并使用类似数组的对象 `args` 作为参数列表。

例如，这两个调用几乎相同：

```Js
func(1, 2, 3);
func.apply(context, [1, 2, 3])
```

两个都运行 `func` 给定的参数是 `1,2,3`。但是 `apply` 也设置了 `this = context`。

`call` 和 `apply` 之间唯一的语法区别是 `call` 接受一个参数列表，而 `apply` 则接受带有一个类似数组的对象。

需要注意：Chrome 14 以及 Internet Explorer 9 仍然不接受类数组对象。如果传入类数组对象，它们会抛出异常。



#### 1. call、apply 与 扩展运算符 

我们已经知道了[JS 基础之:  var、let、const、解构、展开、函数](https://github.com/sisterAn/blog/issues/48)  一章中的扩展运算符 `...`，它可以将数组（或任何可迭代的）作为参数列表传递。因此，如果我们将它与 `call` 一起使用，就可以实现与 `apply` 几乎相同的功能。

这两个调用结果几乎相同：

```js
let args = [1, 2, 3];

func.call(context, ...args); // 使用 spread 运算符将数组作为参数列表传递
func.apply(context, args);   // 与使用 apply 相同
```

如果我们仔细观察，那么 `call` 和 `apply` 的使用会有一些细微的差别。

- 扩展运算符 `...` 允许将 **可迭代的** `参数列表` 作为列表传递给 `call`。
- `apply` 只接受 **类似数组一样的** `参数列表`。 



#### 2. apply 函数转移

`apply` 最重要的用途之一是将调用传递给另一个函数，如下所示：

```js
let wrapper = function() {
  return anotherFunction.apply(this, arguments);
};
```

`wrapper` 通过 `anotherFunction.apply` 获得了：上下文 `this` 和 `anotherFunction` 的参数并返回其结果。

当外部代码调用这样的 `wrapper` 时，它与原始函数的调用无法区分。

#### 3. apply 连接数组

 `array.push.apply` 将数组添加到另一数组上：

```js
var array = ['a', 'b']
var elements = [0, 1, 2]
array.push.apply(array, elements)
console.info(array) // ["a", "b", 0, 1, 2]
```



#### 3. apply 来链接构造器

```js
Function.prototype.construct = function (aArgs) {
  var oNew = Object.create(this.prototype);
  this.apply(oNew, aArgs);
  return oNew;
};
```



#### 4. apply 和内置函数

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

`bind()` 方法会创建一个新绑定函数，当这个新绑定函数被调用时，`this` 键值为其提供的值，其参数列表前几项值为创建时指定的参数序列，绑定函数与被调函数具有相同的函数体（ES5中）。

```js
var module = {
    x: 42,
    getX: function() {
        return this.x
    }
}

var unbindGetX = module.getX
console.log(unbindGetX())
// 在这种情况下，“this” 指向全局作用域
// output: undefined

var bindGetX = unbindGetX.bind(module)
// 创建一个新函数，将 this 绑定到 module 对象
console.log(bindGetX())
// output: 42
```

再看一个例子：

```js
this.value = 11
var module = {
    value: 42
}
function ubx() {
    console.log("ubv-")
    console.log(this.value)
    console.log("-ubv")
}

var bindv = ubx.bind(module) // this 指向 module
console.log(bindv()) 
// ubv-
// 42
// -ubv

console.log(new bindv())  // this 指向 ubx {}
// ubv-
// undefined
// -ubv
```

上面例子中，运行结果 `this.value` 输出为 `undefined` ，这不是全局 `value` ， 也不是 `ubx` 对象中的 `value` ，这说明 `bind` 的 `this` 对象失效了，`new` 的实现中生成一个新的对象，这个时候的 `this` 指向的是 `obj` 。

注意：绑定函数也可以使用 new 运算符构造：这样做就好像已经构造了目标函数一样。提供的 **this** 值将被忽略，而前置参数将提供给模拟函数。



#### 用法

##### 1. 创建绑定函数

`bind()` 最简单的用法是创建一个函数，使这个函数不论怎么调用都有同样的 **this** 值。JavaScript 新手经常犯的一个错误是将一个方法从对象中拿出来，然后再调用，希望方法中的 `this` 是原来的对象（比如在回调中传入这个方法）。如果不做特殊处理的话，一般会丢失原来的对象。从原来的函数和原来的对象创建一个绑定函数，则能很漂亮地解决这个问题：

```js
this.x = 9; 
var module = {
  x: 81,
  getX: function() { return this.x; }
};

module.getX(); // 返回 81

var retrieveX = module.getX;
retrieveX(); 
// 返回 9, 在这种情况下，this 指向全局作用域

// 创建一个新函数，将 this 绑定到 module 对象
var boundGetX = retrieveX.bind(module);
boundGetX(); // 返回 81
```


##### 2. 偏函数

```js
function list() {
  return Array.prototype.slice.call(arguments);
}

var list1 = list(1, 2, 3); // [1, 2, 3]

// Create a function with a preset leading argument
var leadingThirtysevenList = list.bind(undefined, 37);

var list2 = leadingThirtysevenList(); // [37]
var list3 = leadingThirtysevenList(1, 2, 3); // [37, 1, 2, 3]
```


##### 3. 配合 setTimeout

```js
function Bottle() {
  this.name = 'bottle';
}
Bottle.prototype.hello = function() {
  console.log('hello ' + this.name);
};
Bottle.prototype.say = function() {
  setTimeout(this.hello.bind(this), 1000);
};

var bottle = new Bottle();
bottle.say();  
// 一秒钟后, 调用 hello 方法，打印 hello bottle
```


##### 4. 作为构造函数使用的绑定函数

```js
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function() { 
  return this.x + ',' + this.y; 
};

var p = new Point(1, 2);
p.toString(); // '1,2'

var emptyObj = {};
var YAxisPoint = Point.bind(emptyObj, 0/*x*/);
// 以下这行代码在 polyfill 可能不支持,
// 在原生的 bind 方法运行没问题:
//(译注：polyfill 的 bind 方法如果加上把 bind 的第一个参数，
// 即新绑定的 this 执行 Object() 来包装为对象，Object(null) 则是 {} ，那么也可以支持)
var YAxisPoint = Point.bind(null, 0/*x*/);

var axisPoint = new YAxisPoint(5);
axisPoint.toString(); // '0,5'

axisPoint instanceof Point; // true
axisPoint instanceof YAxisPoint; // true
new Point(17, 42) instanceof YAxisPoint; // true
```


#### 然而实际使用时会碰到这样的问题：

```js
function Bottle(name) {
    this.name = name
    this.hello = function(){
        setTimeout(function(){
            console.log('Hello, ', this.name)
        }, 1000)
    }
}

var bottle = new Bottle('bottle')
bottle.hello() // 1s 后打印： Hello，
```

这个时候输出的 `this.name` 是 `null` ，原因是 `this` 指向是在运行函数时确定的，而不是定义函数时候确定的，再因为 `setTimeout` 在全局环境下执行，所以 `this` 指向 `setTimeout` 的上下文：`window`。



##### 解决方法一： 缓存 this

```js
function Bottle(name) {
    this.name = name
    this.hello = function(){
        var _this = this // 缓存this
        setTimeout(function(){
            console.log('Hello, ', _this.name)
        }, 1000)
    }
}

var bottle = new Bottle('bottle')
bottle.hello()// 1s 后打印：Hello,  bottle
```


##### 解决方法二： bind

```js
function Bottle(name) {
    this.name = name
    this.hello = function(){
        setTimeout(function(){
            console.log('Hello, ', this.name)
        }.bind(this), 1000)
    }
}

var bottle = new Bottle('bottle')
bottle.hello()// 1s 后打印： Hello，bottle
```



### 四、call、apply、bind 区别与实现

`call` 、`apply` 都是为了解决 `this` 的指向。作用是相同的，只是传参的方式不同。

除了第一个参数外，`call` 可以接收一个参数列表，`apply` 只能接收一个参数数组。

```js
let a = {
    value: 1
}
function getValue(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value)
}
getValue.call(a, 'yck', '24')
getValue.apply(a, ['yck', '24'])
```

模拟实现 `call` 、`apply`

可以从一下几点考虑实现

- 不传入第一个参数，那么默认为 `window`
- 改变了 `this` 指向，让新的对象可以执行该函数，那么思路是否可以变成新的对象添加一个函数，然后再执行完成后删除



#### 模拟实现 call

```js
Function.prototype.myCall = function(context) {
    var context = context || windows
    // 给 context 添加一个属性
    // getValue.call(a, 'yck', '24') => a.fn = gatValue
    context.fn = this
    // 将 context 后面的参数取出来
    var args = [...arguments].slice(1)
    // getValue.call(a, 'yck', '24') => a.fn('yck', '24')
    var result = context.fn(...args)
    // 删除 fn
    delete context.fn
    return result
}
```

以上就是 `call` 的思路， `apply` 的实现也类似



#### 模拟实现 apply

```js
Function.prototype.myApply = function(context) {
    var context = context || window
    context.fn = this
    
    var result
    // 需要判断是否存储第二个参数
    // 如果存在，就将第二个参数展开
    if (arguments[1]) {
        result = context.fn(...arguments[1])
    } else {
        result = context.fn()
    }
    
    delete context.fn
    return result
}
```



#### 模拟实现 bind

`bind` 和其他两个方法作用是一致的，只是该方法会返回一个函数，并且我们可以通过 `bind` 来实现柯里化。

调用绑定函数通常会导致执行包装函数，绑定函数有以下内部属性：

- `[[BoundTargetFunction]]`：包装的函数（ `function` ）
- `[[BoundThis]]`：调用包装函数的 this 值
- `[[BoundArguments]]`：值列表，其元素用于对包装函数调用的第一个参数
- `[[Call]]`：执行与此对象关联的代码。通过函数调用表达式调用，内部方法的参数是 `this` 值和参数列表

当调用绑定函数时，它调用 `[[BoundTargetFunction]]` 上的内部方法 `[[Call]]` ，后跟参数 **Call(boundThis, args)** 。其中，`boundThis` 是 `[[BoundThis]]` ，`args` 是 `[[BoundArguments]]`，后跟函数调用传递的参数。

```js
Function.prototype.myBind = function(context) {
    if (typeof this !== 'function') {
        throw new TypeError('error')
    }
    var _this = this
    var args = [...arguments].slice(1)
    // 返回一个函数
    return function Fun() {
        // 因为返回一个函数， 我们可以 new Fun(), 所以需要判断
        if (this instanceof Fun) {
            return new _this(...args, ...arguments)
        }
        return _this.call(context, ...args, ...arguments)
    }
}
```



### 五、柯里化

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
