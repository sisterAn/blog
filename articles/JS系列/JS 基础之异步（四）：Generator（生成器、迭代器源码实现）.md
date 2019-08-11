## Step3：Generator

### Generator 是什么

**生成器（Generator）**对象是ES6中新增的语法，和Promise一样，都可以用来异步编程。但与Promise不同的是，它不是使用JS现有能力按照一定标准制定出来的，而是一种新型底层操作，async/await就是在它的基础上实现的。

generator对象是由generator function返回的，符合**可迭代协议**和**迭代器协议**。

generator function 可以在JS单线程的背景下，使JS的执行权与数据自由的游走在多个执行栈之间，实现协同开发编程，当项目调用generator function时，会在内部开辟一个单独的执行栈，在执行一个generator function 中，可以暂停执行，或去执行另一个generator function，而当前generator function并不会销毁，而是处于一种被暂停的状态，当执行权回来的时候，再继续执行。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190124171322303.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1bmFoYWlqaWFv,size_16,color_FFFFFF,t_70)

**可迭代协议**和**迭代器协议**都是ES6的补充

#### 迭代器（Iterator）

顾名思义，所谓迭代器对象就是满足迭代器协议的对象。

> 迭代器协议

迭代器协议定义了一种标准的方式来产生一个有限或无限序列的值。使的迭代器对象拥有一个`next()`对象，并有以下含义：

- next： 返回一个对象的无参函数，返回对象有两个属性：
  - done（boolean)
    - 如果迭代器已经经过了被迭代序列时为 true。这时value可能描述了该迭代器的返回值。
    - 如果迭代器可以产生序列中的下一个值，则为false。这等于说done属性不指定。
  - value
    - 迭代器返回的任何 JavaScript 值。done为true时可省略。

为了加深一下理解，下面贴出

**迭代器构建版本一：Iterator的源码实现**

```js
// 源码实现
function createIterator(items) {
    var i = 0
    return {
        next: function() {
            var done = (i >= items.length)
            var value = !done ? items[i++] : undefined
            
            return {
                done: done,
                value: value
            }
        }
    }
}

// 应用
var iterator = createIterator([1, 2, 3])
console.log(iterator.next())	// {value: 1, done: false}
console.log(iterator.next())	// {value: 2, done: false}
console.log(iterator.next())	// {value: 3, done: false}
console.log(iterator.next())	// {value: undefined, done: true}
```



#### 可迭代（Iterable）

满足可迭代协议的对象就是可迭代对象。

**可迭代协议**：允许JS对象去定义或定制它们的迭代行为。

**可迭代对象**：该对象必须实现@@iterator方法，即这个对象或它原型链（prototype chain）上的某个对象必须有一个名字是Symbol.iterator的属性。

> Symbol.iterator：返回一个对象的无参函数，被返回对象符合迭代器协议

在ES6中，所有的集合对象（Array、Set与Map）以及String、TypedArray、arguments都是可迭代对象，它们都有默认的迭代器。

**当一个对象被迭代的时候，它的@@iterator方法被调用并且无参数，并返回一个值迭代器**

- 扩展运算符

  ```js
  [...'abc']	// ["a", "b", "c"]
  ...['a', 'b', 'c']	// ["a", "b", "c"]
  ```

  

- yield*

  ```js
  function* generator() {
      yield* ['a', 'b', 'c']
  }
  generator().next()	// { value: "a", done: false }
  ```

  

- 解构赋值

  ```js
  let [a, b, c] = new Set(['a', 'b', 'c'])
  a	// 'a'
  ```

#### 可迭代对象

这里以`for ...of`为例子，加深对可迭代对象的理解

`for...of`接受一个可迭代对象（Iterable），或者能强制转换/包装成一个可迭代对象的值（如'abc'）。遍历时，`for...of`会获取可迭代对象的`[Symbol.iterator]()`，对该迭代器逐次调用next()，直到迭代器返回对象的done属性为true时，遍历结束，不对该value处理。

`for...of`循环实例：

```js
var a = ['a', 'b', 'c', 'd', 'e']

for (var val of a) {
    console.log(val)
}
// 'a' 'b' 'c' 'd' 'e'
```

转换成普通的for循环实例，等价于上面`for...of`循环

```js
var a = ["a", "b", "c", "d", "e"]
for (var val, ret, it = a[Symbol.iterator]();
    (ret = it.next()) && !ret.done;
    ) {
    val = ret.value
    console.log(val)
}
// "a" "b" "c" "d" "e"
```



#### 使迭代器可迭代

在**迭代器**部分我们定义了一个简单的迭代器函数`createIterator`，但是该函数生成的迭代器部分并没有实现可迭代协议，所以不能在`for...of`等语法中使用。需要为该对象实现可迭代协议，

在`[Symbol.iterator]`函数中返回该迭代器自身。

```js
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
**添加[Symbol.iterator]使Object可迭代**

根据可迭代协议，给Object的原型添加[Symbol.iterator]，值为返回一个对象的无参函数，被返回对象符合迭代器协议。

```js
Object.prototype[Symbol.iterator] = function () {
    var i = 0
    var items = Object.entries(this)
    return {
        next: function () {
            var done = (i >= items.length)
            var value = !done ? items[i++] : undefined
            
            return {
                done: done,
                value: value
            }
        }
    }
}
```

使用生成器简化代码

```js
Object.prototype[Symbol.iterator] = function* () {
    for (const key in this) {
        if (this.hasOwnPrototype(key)) {
            yield [key, this[key]]
        }
    }
}
```
文章参考自[理解ES6的 Iterator 、Iterable 、 Generator](https://github.com/yueshuiniao/blog/issues/2)