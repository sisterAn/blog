超越 `console.log` ，学习从未用于调试的控制台功能！

![1*0d5jlHyPf6IIrSnodPb3mA](https://user-images.githubusercontent.com/19721451/65444339-6be76980-de62-11e9-8332-a1c025eeee5c.png)


### console API

每个 `JavaScript` 开发人员都使用过 `console.log()` 。 `console` 模块是 `JavaScript` 中最常见的模块之一，其 API 在 ` Node` 中表现为：

> 提供了一个简单的调试控制台，类似于 Web 浏览器提供的 JavaScript 控制台机制。

这是在 [Node.js文档](https://nodejs.org/dist/latest-v12.x/docs/api/console.html) 页面上对 Console 模块的定义。但是，初学者在学习新技术时，更倾向于直接在线搜索解决方案，而不是去阅读文档，这就使得初学者不能完全掌握该新技术的所有正确使用方案，容易在实际开发过程中踩坑。

在谈到 Console API，通常初学者只使用一些基本功能，如👌 `console.log()` 、⚠️ `console.warn()` ，或 ❌ `console.error()` 等。 但是还有许多其他方法可以满足我们的要求并提高调试效率。

以下所有方法在全局实例中都可用 `console` ，因此不需要 console 模块。



### 一、console.assert ✅

该 `console.assert` 函数用于测试传递的参数是真还是假。在传递的值为 `false` 的情况下，该函数记录在第一个参数之后传递的额外参数，否则，代码执行将继续进行，而不会记录任何日志。

```js
// true，没有打印输出
console.assert(1, 'Hello Bottle');
console.assert(true, 'Hello Bottle');
console.assert('hello an', 'Hello Bottle');

// false，有打印输出
console.assert(0, 'Hello Bottle');
// Assertion failed: Hello Bottle
console.assert(false, 'Hello Bottle');
// Assertion failed: Hello Bottle
console.assert('', 'Hello Bottle');
// Assertion failed: Hello Bottle
```

当我们想要检查值是否存在，并且希望保持控制台干净（避免记录较长的属性列表等），我们可以使用 `assert` 方法。



### 二、console.count 和 console.countReset 💯

这两种方法用于设置和清除计数器，以记录特定字符串在控制台中的登录次数：

```js
// 控制台打印 Hello Bottle 被打印的次数
console.count('Hello Bottle');
// Hello Bottle: 1
console.count('Hello Bottle');
// Hello Bottle: 2
console.count('Hello Bottle');
// Hello Bottle: 3

// 重置 Hello Bottle 打印次数为 0
console.countReset('Hello Bottle');

// 如果你重新重置 Hello Bottle 次数，那 Hello Bottle 将从 1 开始
console.countReset('Hello Bottle');
```

计算并重置 `Hello Bottle` 字符串的日志出现次数。



### 三、console.group 和console.groupEnd 🎳

`.group` 并 `.groupEnd` 在控制台中创建并结束一组日志。你可以传递标签作为 `.group()` 描述所关注内容的第一个参数：

```js
console.group('Grandparents group');
console.log('Grandpa');
console.log('Grandma');
console.groupEnd()

console.group('Parents group');
console.log('Father');
console.log('Mother');
console.groupEnd()

console.group('Children group');
console.log('Son');
console.log('Daughter');
console.groupEnd()
```

<img width="420" alt="console group" src="https://user-images.githubusercontent.com/19721451/65444658-1495c900-de63-11e9-88ef-df7ae304bf66.png">


### 四、console.table 📋

此特定方法对于描述人性化的表中的对象或数组内容非常有用：

```js
const users = [
  {name: 'Bottle', age: 12},
  {name: 'AnGe', age: 18}
];

console.table(users);
```

<img width="582" alt="console table" src="https://user-images.githubusercontent.com/19721451/65444669-1cee0400-de63-11e9-9125-1b6d88da4d67.png">

用户对象列表表。

`console.table` 使嵌套和复杂的数组/对象的检查和记录变得更加容易。



### 五、console.time 和 console.timeEnd ⏱

如果你想在执行时检查代码的性能，并解决该问题，你可以 `Date` 使用 API 创建一个开始时间戳，并使用它来计算代码执行后的差异？像这样：

```js
const start = Date.now();
// ...
const diff = Date.now() - start;
console.log('Time execution: ' + diff + ' ms');
// Time execution: <number> ms
```

好了，使用 `time` 和 `timeEnd` 函数，没有必要执行此技巧。你只需执行以下操作即可创建时序报告：

```js
const start = Date.now();
for(let i = 0; i < 10000000; i++) {
    1+1; // ...
}
const diff = Date.now() - start;
console.log('Time execution with Date.now: ' + diff + ' ms');
// Time execution with Date.now: 8 ms

console.time('Time execution with console.time');
for(let i = 0; i < 10000000; i++) {
    1+1; // ...
}
console.timeEnd('Time execution with console.time')
// Time execution with console.time: 7.21630859375ms
```

如你所见，console.time还返回了更准确的结果。

### 总结

现在只有3分钟的时间，你现在可以在 Console API 中使用更多出色的工具。将它们与你的调试习惯集成在一起，你的开发速度将成倍提高！