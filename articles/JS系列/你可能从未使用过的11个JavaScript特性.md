![](http://resource.muyiy.cn/image/20200105224640.png)

### 一、逗号运算符

`,` 是用于分隔表达式并返回链中最后一个表达式的运算符。

```js
let oo = (1, 2, 3)
console.log(oo) // 77
```

这里有三个主要表达式 `1` 、 `2` 和 `3`。所有这些表达式均被求值，最后一个赋给 oo。

我们在 for 循环中看到这个：

```js
for(let i = 0, ii = 1; i< 10; i++, ii--) { ... }
```

当我们要编写短的 lambda 函数时，这会派上用场：

```js
const lb = (a, b, arr) => (arr.push(a*b), a*b)
```

这里有两个语句，第一个将乘法结果推入数组arr，第二个将乘数a和b推入数组。 第二个结果就是返回给调用者的内容。

对于三元运算符，它也很有用，因为与短lambda语法相同，它仅接受表达式而不是语句。



### 二、in

in 是用于检查对象中属性是否存在的关键字。 我们在 `for..in` 循环中使用了它，但没有意识到，其实 `in` 也是一个关键字:)

如果对象上存在属性，则 `in` 将返回 `true` ，否则将返回 `false`。

```js
const o = {
    prop: 1
}
console.log("prop" in o) // true
```

看，`in` 可以独立使用，而不是在 `for..in` 中。

它将检查 `"prop"` 是否可作为 `o` 对象中的属性使用。 它返回 `true` ，因为我们在 `o` 中定义了 `"prop"` 属性。

如果我们检查未定义的属性：

```js
const o = {
    prop: 1
}
console.log("prop1" in o) // false
```

它返回 `false` ，因为 `"prop1"` 在 `o` 中未定义。



### 三、Array 构造函数

你知道我们可以不使用传统方法定义数组吗？

```js
const arr = [1, 2, 3]
```

怎么样？

我们也可以使用 `Array` ：

```js
const arr = new Array(1, 2, 3)
```

传递给 `Array` 构造函数的参数的排列将构成其索引的基础。

`1` 是第一个参数，其索引为 0； `2` 是第二个参数，其索引为 1； `3` 是第三个参数，其索引为 2。

```js
arr[0] // 90
arr[1] // 88
arr[2] // 77
```

所以，

```js
const arr = new Array(1, 2, 3)
```

和

```js
const arr = [1, 2, 3]
```

表达的是一个意思。

但使用 `new Array()` 有一个问题，例如：

```js
var a = new Array(10, 20);
a[0] // 返回 10
a.length // 返回 2
```

但：

```js
var a = new Array(10);
a[0] // 返回 undefined
a.length // 返回 10
```

当你仅给 `Array` 构造函数一个整数（大于等于 0 的整数，否则将会报错）时，才会发生这种情况。 这是为什么喃？ 

其实，新的 `Array` 构造函数正在从某些编程语言中提取思想，在这些语言中，你需要为数组指定内存，这样就不会出现 `ArrayIndexOutOfBounds` 异常。

```js
int *a = (int *) malloc( 10*sizeof(int) ); // ya ol' c
int *a = new int[10]; // c++
int[] a = new int[10]; // java
```

是的，它实际上是在创建一个长度为 `10` 的数组。我们在 `Javascript` 中没有 `sizeof` 函数，但是 `toString` 足以证明这一点。

```js
a.toString() // 返回 ",,,,,,,,," 它相当于 [,,,,,,,,,]
a // [empty × 10]
```

所以，当将一个参数传递给的 `new Array`，将导致 JS 引擎为传递的参数大小的数组分配空间。

 并且这也在 EcmaScript 规范中：

![](http://resource.muyiy.cn/image/20200105224702.png)

看，这不是矛盾的。 规格中都有所有描述。 在得出任何结论之前，我们应该始终先阅读任何语言的规范。

### 四、Function 构造函数

你是否知道我们可以使用 `Function` 构造函数定义 `Function` 。

你不明白吧？ 让我更清楚。 在 JavaScript 中，我们定义如下函数：

```js
const mul = (a, b) => a * b

// 或
function mul(a, b) {
    return a * b
}

// 或
const mul = function(a, b) {
    return a * b
}
```

我们也可以这样做，来实现相同的功能：

```js
const mul = new Function("a", "b", "return a * b")
```

传递给 `Function` 的参数形成函数的参数和主体。 变量 `mul` 成为函数名称。

并且，最后一个参数将是函数的主体，而最后一个参数之前的参数将成为函数的参数。

在在 `mul` 中。  `"a"` 和 `"b"` 是函数将接收的参数，`"return a * b"` 是函数的主体。 它实现将 `"a"` 和 `"b"` 相乘并返回结果。

我们使用 `mul（…）` 调用该函数，并传入参数：

```js
const mul = new Function("a", "b", "return a * b")

console.log(mul(7, 8)) // 56
```

根据 MDN：

> **Function 构造函数**创建一个新的 `Function` **对象**。直接调用此构造函数可用动态创建函数，但会遭遇来自 `eval` 的安全问题和相对较小的性能问题。然而，与 eval 不同的是，Function 构造函数只在全局作用域中运行。

### 五、数组解构

我们可以通过使用元素的索引号来分解数组中的元素。

```js
const arr = [1, 2, 3]
```

元素 `1` 、`2` 、`3` 的索引分别为 0、1、2，即：

```js
arr[0] // 1
```

在日常开发中，我们最常使用的是对象解构：

```js
let o = {
    prop: 1
}
o["prop"] // 1

// 解构
const {prop} = o
prop // 1
```

所以，我们将解构用于数组上：

```js
const arr = [1, 2, 3]
const { 0: firstA, 1: secA, 2: thirdA  } = arr

firstA // 1
secA // 2
thirdA // 3
```

所以我们可以使用索引号来提取元素。索引是定义数组中元素位置的属性。

```js
const arr = [1, 2, 3]
```

相当于：

```js
const arr = {
    0: 1,
    1: 2,
    2: 3,
    length: 3
}
```

数组也是对象，这就是为什么要对其进行对象分解的原因，但是还有一种特殊的数组分解语法：

```js
const [first, second, third] = arr

first // 1
second // 2
third // 3
```

**注意：应尽可能避免知道数组中的特定位置信息（开始、结束索引是什么）。**

### 六、使用 length 属性减少数组内容

数组中的 `length` 属性表示数组中元素的数目。

```js
const arr = [1, 2, 3]
arr.length // 3
```

减小 `length` 属性值，会使 `JS` 引擎将数组元素个数减少到与 `length` 属性的值相等。

```js
const arr = [1, 2, 3]
arr.length // 3
arr.length = 1
arr // [1]
```

`arr` 的 `length` 属性值更改为 `1`，因此 `arr` 减少了元素个数，使其等于 `length` 属性值。

如果增加 `length` 属性，则 `JS` 引擎将添加元素（未定义的元素）以使数组中的元素数量达到 `length` 属性的值。

```js
const arr = [1, 2, 3]
arr.length // 3
arr.length = 1
arr // [1]

arr.length = 5
arr // [1, empty × 4]
```

`arr` 中的元素只有一个，然后我们将长度增加到 5 ，因此又增加了 4 个元素长度，使元素数达到 5。

### 七、Arguments

我们可以使用 `arguments` 对象获取传递给函数的参数，而无需在函数中明确定义 `arguments` 变量：

```js
function myFunc() {
    console.log(arguments[0]) // 34
    console.log(arguments[1]) // 89
}

myFunc(34,89)
```

`arguments` 对象是数组索引的。 也就是说，属性是数字，因此可以通过键引用进行访问。

`arguments` 对象是从 `Arguments` 类实例化的，该类具有一些很酷的属性。

`arguments.callee.name` 指当前正在调用的函数的名称。

```js
function myFunc() {
    console.log(arguments.callee.name) // myFunc
}

myFunc(34, 89)
```

`arguments.callee.caller.name` 是指调用当前执行函数的函数的名称。

```js
function myFunc() {
    console.log(arguments.callee.name) // myFunc
    console.log(arguments.callee.caller.name) // myFuncCallee
}

(function myFuncCallee() {
    myFunc(34, 89)
})()
```

这在可变参数功能中特别有用。

### 八、跳过 ()

你是否知道实例化对象时可以跳过方括号 `()` ？

例如：

```js
class D {
    logger() {
        console.log("D")
    }
}

// 一般情况下，我们这么做：
(new D()).logger(); // D

// 其实，我们可以跳过 ():
(new D).logger(); // D
// 并且它可以正常运行
```

即使在内置类中，括号也是可选的：

```js
(new Date).getDay();
(new Date).getMonth();
(new Date).getYear();
```

### 九、void

`void` 是 JS 中的关键字，用于评估语句并返回未定义。

例如：

```js
class D {
   logger() {
        return 89
    }
}

const d = new D

console.log(void d.logger()) // undefined
```

`logger` 方法应该返回 `89` ，但是 `void` 关键字将使其无效并返回 `undefined` 。

我曾经读到过 `undefined` 之前可能会被赋予另一个值，而这会伪造其语义。 因此，使用 `void` 运算符可确保你得到一个真正的 `undefined` 。 也用于最小化目的。

### 十、通过 `__proto__` 继承

`_proto_` 是从 ` JavaScript` 中的对象继承属性的方法。 `__proto__` 是   `Object.prototype` 的访问器属性，它公开访问对象的 `[[Prototype]]` 。

此 `__proto__` 将其 `[[Prototype]]` 中设置的对象的所有属性设置为目标对象。

让我们看一个例子：

```js
const l = console.log
const obj = {
    method: function() {
        l("method in obj")
    }
}
const obj2 = {}
obj2.__proto__ = obj
obj2.method() // method in obj
```

我们有两个对象常量： `obj` 和 `obj2` 。 `obj` 具有 `method` 属性。 `obj2` 是一个空的对象常量，即它没有属性。

我们访问 `obj2` 的 `__proto__` 并将其设置为 `obj` 。 这会将通过 `Object.prototype` 可访问的 `obj` 的所有属性复制到 `obj2` 。 这就是为什么我们可以在 `obj2` 上调用方法而不会在没有定义的情况下得到错误的原因。

`obj2` 继承了 `obj` 的属性，因此 `method` 方法属性将在其属性中可用。

原型可用于对象，例如对象常量、对象、数组、函数、日期、RegEx、数字、布尔值、字符串。

### 十一、一元运算符 +

一元 `+` 运算符将其操作数转换为数字类型。

```js
+"23" // 23
+{} // NaN
+null // 0
+undefined // NaN
+{ valueOf: () => 67 } // 67
+"nnamdi45" // NaN
```

当我们希望将变量快速转换为 `Number` 时，这非常方便。

### 十二、一元运算符 -

一元运算符 `-` 将其操作数转换为 `Number` 类型，然后取反。

该运算符将一元 `+` 运算符的结果取反。 首先，它将操作数转换为其 `Number` 值，然后取反该值。

```js
-"23" // -23
```

此处发生的是，字符串 `"23"` 将转换为其数字类型，从而得到 `23` 。然后，此正数将转换为其负数形式 `-23` 。

```js
-{} // NaN
-null // -0
-undefined // NaN
-{ valueOf: () => 67 } // -67
-"nnamdi45" // NaN
```

如果转换为数值的结果为 `NaN` ，则不会应用取反。

取负 `+0` 产生 `-0` ，取负 `-0` 产生 `+0` 。

```js
- +0 // -0
- -0 // 0
```

### 十三、指数运算符 **

该运算符用于指定数字的指数。

在数学中， 2^3^ 意味着将 2 乘以三次：

```js
2 * 2 * 2
```

我们可以使用 `**` 运算符在 JS 中进行相同的操作：

```js
2 ** 3 // 8
9 ** 3 // 729
```





