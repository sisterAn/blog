### 一、instanceof

#### 1. 引入 instanceof 

在 JS 中，判断一个变量的类型，常常会用到 `typeof` 运算符，但当用 `typeof` 来判断引用类型变量时，无论是什么类型的变量，它都会返回 `Object` 。

```js
// 基本类型
console.log(typeof 100); // number
console.log(typeof 'bottle'); // string
console.log(typeof true); // boolean

// 引用类型
console.log(typeof {}); // object
console.log(typeof [1, 2, 3]); // object
```

为此，引入了`instanceof`。

`instanceof` 操作符用于检测对象是否属于某个 class，同时，检测过程中也会将继承关系考虑在内。

```js
// 类
class Bottle {}
// bottle 是 Bottle 类的实例对象
let bottle = new Bottle();
console.log(bottle instanceof Bottle); // true

// 也可以是构造函数，而非 class
function AnGe() {}
let an = new AnGe();
console.log(an instanceof AnGe); // true
```



#### 2. instanceof 与 typeof

`instanceof` 与 `typeof` 相比，`instanceof ` 方法要求开发者明确的确认对象为某特定类型。即 `instanceof` 用于判断引用类型属于哪个构造函数的方法。

```js
var arr = []
arr instanceof Array // true
typeof arr // "object"
// typeof 是无法判断类型是否为数组的
```



#### 3. instanceof 在继承关系中

另外，更重的一点是 `instanceof` 可以在继承关系中用来判断一个实例是否属于它的父类型。

```js
// 判断 f 是否是 Foo 类的实例 , 并且是否是其父类型的实例
function Aoo(){} 
function Foo(){} 
//JavaScript 原型继承
Foo.prototype = new Aoo();
 
var foo = new Foo(); 
console.log(foo instanceof Foo) // true 
console.log(foo instanceof Aoo) // true
```

`f instanceof Foo` 的判断逻辑是：

- f 的 `__proto__`一层一层往上，是否对应到 `Foo.prototype`
- 再往上，看是否对应着`Aoo.prototype`
- 再试着判断 `f instanceof Object`

即 `instanceof` 可以用于判断多层继承关系。

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

在这组数据中，Object、Function instanceof 自己均为 `true`， 其他的 instanceof 自己都为 `false`，这就要从 `instanceof` 的内部实现机制以及  JS  原型继承机制讲起。



### 二、instanceof 的内部实现机制

instanceof 的内部实现机制是：通过判断对象的原型链上是否能找到对象的 `prototype`，来确定 `instanceof` 返回值

#### 1. 内部实现

   ```js
// instanceof 的内部实现 
function instance_of(L, R) {//L 表左表达式，R 表示右表达式，即L为变量，R为类型
  // 取 R 的显示原型
  var prototype = R.prototype
  // 取 L 的隐式原型
  L = L.__proto__
  // 判断对象（L）的类型是否严格等于类型（R）的显式原型
  while (true) { 
    if (L === null) {
      return false
    }
      
    // 这里重点：当 prototype 严格等于 L 时，返回 true
    if (prototype === L) {
      return true
    } 
    
    L = L.__proto__
  } 
}
   ```

`instanceof ` 运算符用来检测 `constructor.prototype ` 是否存在于参数 `object` 的原型链上。

#### 2. 你真的了解 instanceof 了吗

看下面一个例子，`instanceof` 为什么会返回 `true`？很显然，`an` 并不是通过 `Bottle()` 创建的。

```js
function An() {}
function Bottle() {}
An.prototype = Bottle.prototype = {};

let an = new An();
console.log(an instanceof Bottle); // true
```

这是因为 `instanceof` 关心的并不是构造函数，而是原型链。

```js
an.__proto__ === An.prototype; // true
An.prototype === Bottle.prototype; // true
// 即
an.__proto__ === Bottle.prototype; // true
```

即有 `an.__proto__ === Bottle.prototype` 成立，所以 `an instanceof Bottle` 返回了 `true`。

所以，按照 `instanceof` 的逻辑，真正决定类型的是 `prototype`，而不是构造函数。



### 三、 JS 原型链继承关系

   ![原型链](http://www.mollypages.org/tutorials/jsobj_full.jpg)

图片来自于 [JS原型链](http://www.mollypages.org/tutorials/js.mp)

由其本文涉及显示原型 `prototype` 和隐式原型 `__proto__` ，所以下面对这两个概念作一下简单说明。

在 JavaScript 原型继承结构里面，规范中用 `[Prototype]]` 表示对象隐式的原型，在 JavaScript 中用 `__proto__` 表示，并且在 Firefox 和 Chrome 浏览器中是可以访问得到这个属性的，但是 IE 下不行。所有 JavaScript 对象都有 ` __proto__`  属性，但只有 `Object.prototype.__proto__` 为 null，前提是没有在 Firefox 或者 Chrome 下修改过这个属性。这个属性指向它的原型对象。 至于显示的原型，在 JavaScript 里用 `prototype` 属性表示，这个是 JavaScript 原型继承的基础知识，如果想进一步了解，请参考 [JS 基础之: 深入 constructor、prototype、`__proto__`、[[Prototype]] 及 原型链](https://github.com/sisterAn/blog/issues/5)

下面介绍几个例子（及其推演过程），加深你的理解：

#### 1. Object instanceof Object

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


#### 2. Function instanceof Function

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


#### 3. Foo instanceof Foo

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



### 四、 instanceof 与 Symbol.hasInstance

**Symbol.hasInstance** 用于判断某对象是否为某构造器的实例。因此你可以用它自定义 [`instanceof`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof) 操作符在某个类上的行为。

你可实现一个自定义的`instanceof` 行为，例如：

```js
class MyArray {  
  static [Symbol.hasInstance](instance) {
    return Array.isArray(instance);
  }
}
console.log([] instanceof MyArray); // true
```



### 五、 instanceof 与 isPrototypeOf

`isPrototypeOf` 也是用来判断一个对象是否存在与另一个对象的原型链上。

```js
// 判断 f 是否是 Foo 类的实例 , 
// 并且是否是其父类型的实例
function Aoo(){} 
function Foo(){} 
// JavaScript 原型继承
Foo.prototype = new Aoo();
 
var foo = new Foo(); 
console.log(Foo.prototype.isPrototypeOf(foo)) //true 
console.log(Aoo.prototype.isPrototypeOf(foo)) //true
```

需要注意的是：

- `instanceof `：`foo` 的原型链是针对 `Foo.prototype` 进行检查的
- `isPrototypeOf`：`foo` 的原型链是针对 `Foo` 本身



### 六、 instanceof 和多全局对象(多个 frame 或多个 window 之间的交互)
instanceof 在多个全局作用域下，判断会有问题，例如：
```js
// parent.html
<iframe src="child.html" onload="test()">
</iframe>
<script>
  function test(){
    var value = window.frames[0].v;
    console.log(value instanceof Array); // false
  }
</script>
```
```js
// child.html
<script>
  window.name = 'child';
  var v = [];
</script>
```
严格上来说 value 就是数组，但 parent 页面中打印输出： false ；

这是因为 Array.prototype !== window.frames[0].Array.prototype ，并且数组从前者继承。

出现问题主要是在浏览器中，当我们的脚本开始开始处理多个 frame 或 windows 或在多个窗口之间进行交互。多个窗口意味着多个全局环境，不同的全局环境拥有不同的全局对象，从而拥有不同的内置类型构造函数。

#### 解决方法

可以通过使用

-  `Array.isArray(myObj)` 或者 

- `Object.prototype.toString.call(myObj) === "[object Array]"` 

来安全的检测传过来的对象是否是一个数组



### 七、扩展：Object.prototype.toString 方法

默认情况下(不覆盖 `toString` 方法前提下)，任何一个对象调用 `Object` 原生的 `toString` 方法都会返回 `"[object type]"`，其中 `type` 是对象的类型；

```js
let obj = {};

console.log(obj); // {}
console.log(obj.toString()); // "[object Object]"
```



#### 1. 使用 `toString()` 检测对象类型

可以通过 `toString()` 来获取每个对象的类型。这是因为  `toString` 方法可以从对象中提取出来，以其他值作为上下文（context）对象进行调用，调用结果取决于传入的上下文对象。

```js
function isFunction(value) {
  return Object.prototype.toString.call(value) === "[object Function]"
}
function isDate(value) {
  return Object.prototype.toString.call(value) === "[object Date]"
}
function isRegExp(value) {
  return Object.prototype.toString.call(value) === "[object RegExp]"
}

isDate(new Date()); // true
isRegExp(/\w/); // true
isFunction(function(){}); //true
```
或者可写为：

```js
function generator(type){
  return function(value){
    return Object.prototype.toString.call(value) === "[object "+ type +"]"
  }
}

let isFunction = generator('Function')
let isArray = generator('Array');
let isDate = generator('Date');
let isRegExp = generator('RegExp');

isArray([]));    // true
isDate(new Date()); // true
isRegExp(/\w/); // true
isFunction(function(){}); //true
```

每个类的内部都有一个 `[[Class]]` 属性，这个属性中就指定了上述字符串中的 `type` (构造函数名) ；



#### 2. Symbol.toStringTag

`Object.prototype.toString` 方法可以使用 `Symbol.toStringTag` 这个特殊的对象属性进行自定义输出。

举例说明：

```js
let bottle = {
  [Symbol.toStringTag]: "Bottle"
};

console.log(Object.prototype.toString.call(bottle)); // [object Bottle]
```

大部分和环境相关的对象也有这个属性。以下输出可能因浏览器不同而异：

```js
// 环境相关对象和类的 toStringTag：
console.log(window[Symbol.toStringTag]); // Window
console.log(XMLHttpRequest.prototype[Symbol.toStringTag]); // XMLHttpRequest

console.log(Object.prototype.toString.call(window)); // [object Window]
console.log(Object.prototype.toString.call(new XMLHttpRequest())); // [object XMLHttpRequest]
```

输出结果和 `Symbol.toStringTag`（前提是这个属性存在）一样，只不过被包裹进了 `[object ...]` 里。

所以，如果希望以字符串的形式获取内置对象类型信息，而不仅仅只是检测类型的话，可以用这个方法来替代 `instanceof`。

### 八、总结

|                             | 适用于                                                       | 返回       |
| --------------------------- | ------------------------------------------------------------ | ---------- |
| typeof                      | 基本数据类型                                                 | string     |
| instanceof                  | 任意对象                                                     | true/false |
| `Object.prototype.toString` | 基本数据类型、内置对象以及包含 `Symbol.toStringTag` 属性的对象 | string     |

`Object.prototype.toString` 基本上就是一增强版 `typeof`。

`instanceof` 在涉及多层类结构的场合中比较实用，这种情况下需要将类的继承关系考虑在内。



### 九、参考

 [JavaScript instanceof 运算符深入剖析](https://www.ibm.com/developerworks/cn/web/1306_jiangjj_jsinstanceof/)

[类型检测："instanceof"](https://zh.javascript.info/instanceof)