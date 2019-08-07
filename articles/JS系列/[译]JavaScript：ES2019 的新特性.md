作为最流行的编程语言和最重要的 Web 开发语言之一，JavaScript 不断演变，每次迭代都会得到一些新的内部更新。让我们来看看 ES2019 有哪些新的特性，并加入到我们日常开发中:

### Array.prototype.flat()

`Array.prototype.flat()` 递归地将嵌套数组拼合到指定深度。默认值为 1，如果要全深度则使用 **Infinity** 。此方法不会修改原始数组，但会创建一个新数组:

```js
const arr1 = [1, 2, [3, 4]];
arr1.flat(); 
// [1, 2, 3, 4]

const arr2 = [1, 2, [3, 4, [5, 6]]];
arr2.flat(2); 
// [1, 2, 3, 4, 5, 6]

const arr3 = [1, 2, [3, 4, [5, 6, [7, 8]]]];
arr3.flat(Infinity); 
// [1, 2, 3, 4, 5, 6, 7, 8]
```

`flat()` 方法会移除数组中的空项:

```js
const arr4 = [1, 2, , 4, 5];
arr4.flat(); // [1, 2, 4, 5]
```



### Array.prototype.flatMap()

`flatMap()` 方法首先使用映射函数映射每个元素，然后将结果压缩成一个新数组。它与 `Array.prototype.map` 和 深度值为 1的 `Array.prototype.flat` 几乎相同，但 `flatMap` 通常在合并成一种方法的效率稍微高一些。

```js
const arr1 = [1, 2, 3];

arr1.map(x => [x * 4]); 
// [[4], [8], [12]]

arr1.flatMap(x => [x * 4]); 
// [4, 8, 12]
```

更好的示例：

```js
const sentence = ["This is a", "regular", "sentence"];

sentence.map(x => x.split(" ")); 
// [["This","is","a"],["regular"],["sentence"]]

sentence.flatMap(x => x.split(" ")); 
// ["This","is","a","regular", "sentence"]

// 可以使用 归纳（reduce） 与 合并（concat）实现相同的功能
sentence.reduce((acc, x) => acc.concat(x.split(" ")), []);
```



### String.prototype.trimStart() 和 String.prototype.trimEnd()

除了能从字符串两端删除空白字符的 `String.prototype.trim()` 之外，现在还有单独的方法，只能从每一端删除空格:

```js
const test = " hello ";

test.trim(); // "hello";
test.trimStart(); // "hello ";
test.trimEnd(); // " hello";
```

- `trimStart()` ：别名 `trimLeft()`，移除原字符串左端的连续空白符并返回，并不会直接修改原字符串本身。
- `trimEnd()` ：别名 `trimRight()`，移除原字符串右端的连续空白符并返回，并不会直接修改原字符串本身。

### Object.fromEntries

将键值对列表转换为 Object 的新方法。

它与已有 **Object.entries()** 正好相反，`Object.entries()`方法在将对象转换为数组时使用，它返回一个给定对象自身可枚举属性的键值对数组。

但现在您可以通过 `Object.fromEntries` 将操作的数组返回到对象中。

下面是一个示例（将所有对象属性的值平方）:

```js
const obj = { prop1: 2, prop2: 10, prop3: 15 };

// 转化为键值对数组：
let array = Object.entries(obj); 
// [["prop1", 2], ["prop2", 10], ["prop3", 15]]
```

将所有对象属性的值平方:

```js
array = array.map(([key, value]) => [key, Math.pow(value, 2)]); 
// [["prop1", 4], ["prop2", 100], ["prop3", 225]]
```

我们将转换后的数组 `array` 作为参数传入 **Object.fromEntries** ，将数组转换成了一个对象:

```js
const newObj = Object.fromEntries(array); 
// {prop1: 4, prop2: 100, prop3: 225}
```



### 可选的 Catch 参数

新提案允许您完全省略 `catch()` 参数，因为在许多情况下，您并不想使用它:

```js
try {
  //...
} catch (er) {
  //handle error with parameter er
}

try {
  //...
} catch {
  //handle error without parameter
}
```



### Symbol.description

`description` 是一个只读属性，它会返回 `Symbol` 对象的可选描述的字符串，用来代替 `toString()` 方法。

```js
const testSymbol = Symbol("Desc");

testSymbol.description; // "Desc"

testSymbol.toString(); // "Symbol(Desc)"
```



### Function.toString()

现在,在函数上调用 `toString()` 会返回函数，与它的定义完全一样，包括空格和注释。

之前：

```js
function /* foo comment */ foo() {}

foo.toString(); // "function foo() {}"
```

现在：

```js
foo.toString(); // "function /* foo comment */ foo() {}"
```



### JSON.parse() 改进

行分隔符 **(\u2028)** 和段落分隔符 **(\u2029)**，现在被正确解析，而不是报一个语法错误。

```js
var str = '{"name":"Bottle\u2028AnGe"}'
JSON.parse(str)
// {name: "Bottle AnGe"}
```




原文链接：[JavaScript: What’s new in ES2019](https://blog.tildeloop.com/posts/javascript-what’s-new-in-es2019)