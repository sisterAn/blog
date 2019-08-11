接着上一部分继续了解[JS 基础之异步（四）：Generator（生成器、迭代器源码实现）](https://github.com/sisterAn/blog/issues/20)

#### Generator函数

`function *`会定义一个生成器函数，并返回一个Generator（生成器）对象，其内部可以通过 yield 暂停代码，通过调用 next 恢复执行。

**调用一个生成器对象并不会马上执行里面的代码语句，而是返回一个这个生成器的迭代器（iterator）对象。当这个迭代器对象的`next() `方法被调用时，其内部的语句就会被执行到`yield`的位置，`yield`后紧跟需要返回的值（可以是函数或表达式）。如果遇到的`yield`为`yield*`的话，则表示将执行权交给另一个生成器函数（当前生成器暂停执行）。**

**注意**：

- generator function内部才能使用`yield/yield*`命令，而generator function内部调用的或声明的其他普通函数是不能调用`yield/yield*`命令的。
- 箭头函数不能使用`yield`，即箭头函数不能用做generator function（但可以用做async function）

```js
// 声明式
function* generator() {}

// 表达式
let generator = function* (){}

// 作为对象属性
let obj = {
    generator: function* (){}
}

// 箭头函数不能用做generator function，报错
let obj = {
    generator: *() => {}
}

// 箭头函数可以用做 async 函数
let obj = {
    generator: async () => {}
}
```

**在生成器中return**

遍历返回对象的done值为true时迭代即结束，不对该value处理。

```js
function* createIterator() {
    yield 'a'
    return 1
    yield 'b'
}
let iterator = createIterator()
iterator.next()		// {value: "a", done: false}
iterator.next()		// {value: 1, done: true}
iterator.next()		// {value: undefined, done: true}
```

所以对这个迭代器遍历，不会对值`1`处理。当在生成器函数中显式 `return `时，会导致生成器立即变为完成状态，即调用 `next()` 方法返回的对象的 `done `为 `true`。如果 `return `后面跟了一个值，那么这个值会作为**当前**调用 `next()` 方法返回的 value 值，若  `return `后面没有跟任何值，则返回 `undefined`，相当于默认 `return undefined`。

**注意：生成器函数不能当构造函数使用**

```js
function* f() {}
var obj = new f; // throws "TypeError: f is not a constructor"
```



#### Generator对象

Generator对象（生成器对象）既是迭代器，又是可迭代对象。

```js
function* generator() {
    yield 'a'
    yield 'b'
    yield 'c'
}

var gen = generator()

// 满足迭代器协议，是迭代器
gen.next()	// {value: "a", done: false}
gen.next()	// {value: "b", done: false}
gen.next()	// {value: "c", done: false}
gen.next()	// {value: undefined, done: true}

// [Symbol.iterator]是一个无参函数，该函数执行后返回生成器对象本身（是迭代器），所以是可迭代对象
gen[Symbol.iterator]() === gen	// true

// 可以被迭代
var gen1 = generator()
[...gen1]	// ["a", "b", "c"]
```



#### yield与next

先看一个例子

```js
function* generator() {
    var val = yield 1
    return val + 1
}

var gen = generator()
var gen1 = gen.next
console.log('1: ', gen1.value)
gen1 = gen.next
console.log('2: ', gen1.value)
gen1 = gen.next
console.log('3: ', gen1.value)
// output: 1 NaN undefined
```

为什么会这样喃？这是因为调用 `next()`方法时，如果传入了参数，那么这个参数会作为**上一条执行的  yield 语句的返回值**。如果没有，则上一条执行的 yield 语句的返回值默认为 NaN，因为在处理的时候，我们没有给 next 函数传值，导致 yield 语句返回值为 NaN，则 val 为NaN，NaN + 1 当然也是 NaN了。

#### yield*

yield * 也称为 yield 委托。作用是将执行权交给另一个生成器或可迭代对象。（当前生成器暂停执行）。`yield*` 表达式迭代操作数，并产生它返回的每个值。

下面看一个例子：

```Js
function* generator(index) {
    yield index + 1
    yield index + 2
    yield index + 3
}

function* mainGenerator(index) {
    yield index
    yield* generator(index)
    yield index + 100
}

var gen = mainGenerator(1)

console.log(gen.next().value)	// 1
console.log(gen.next().value)	// 2
console.log(gen.next().value)	// 3
console.log(gen.next().value)	// 4
console.log(gen.next().value)	// 101
```

**注意：`yield*` 是一个表达式，不是语句，所以它会有自己的值，即为当前迭代器关闭时返回的值（即`done`为`true`时）。**

```js
function* gnerator1() {
  yield* [1, 2, 3];
  return "gnerator1";
}

var result;
function* gnerator2() {
  result = yield* gnerator1();
}

var iterator = gnerator2();
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: 3, done: false }
console.log(iterator.next()); // { value: undefined, done: true }, 
                              // 此时 g4() 返回了 { value: "foo", done: true }

console.log(result);          // "foo"
```

#### generator 执行异步函数

下面看一个例子：

```js
function asyncFun() {
    return new Promise((resolve, reject) => {
        setTimeout(()=>{
            resolve('promise')
        }, 3000)
    })
}
function *generator() {
    var result = yield asyncFun()
    console.log(result)
}
```

在这个例子中，yield 后跟着一个异步函数 asyncFun ，我们如何操作才能使整个流程顺序执行喃？

```js
var gen = generator()
var afun = gen.next()
afun.value.then( res => {
    console.log(res)
}).then( res => {
    gen.next('success')
})
// 3s后打印：promise success
```

上面代码中，首先执行 Generator 函数，获取遍历器对象 gen，然后使用`next`方法 ，执行异步任务的第一阶段。由于 asyncFun 函数返回的是一个 Promise 对象，因此要用`then`方法调用下一个`next`方法。

### Generator 实现及源码解读

#### Generatror 函数构建版本一

```js
// cb 也就是编译过的 test 函数
function generator(cb) {
    return (function() {
        var object = {
            next: 0,
            stop: function() {}
        }
        
        return {
            next: function() {
                var ret = cb(object)
                if (ret === undefined) return {value: undefined, done: true}
                return {
                    value: ret, 
                    done: false
                }
            }
        }
    })()
}

// 如果你使用babel编译后，可以发现 test 函数变成了这样
function test() {
    var a
    return generator(function(_context) {
        while (1) {
            switch ((_context.prev = _context.next)) {
            	// 可以发现通过 yield 将代码分割成几块
                // 每次执行 next 函数就执行一块代码
                // 并且表明下次需要执行那块代码
                case 0:
                    a = 1 + 2
                    _context.next = 4
                    return 2
                case 4:
                    _context.next = 6
                    return 3
                // 执行完毕
                case 6:
                case "end":
                    return _context.stop()
            }
        }
    })
}
```