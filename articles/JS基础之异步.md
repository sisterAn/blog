# JS 基础之异步

已知，JS是单线程的，天生异步，适合IO密集型，不适合CPU密集型，但是，为什么是异步的喃，异步由何而来的喃，我们将在这里逐渐讨论实现

## 单线程的JavaScript

所谓单线程，是指在JS引擎中负责解释和执行JS代码的线程唯一，同一时间上只能执行一件任务。

首先为什么要引入单线程喃？

原因：避免DOM渲染的冲突

- 浏览器需要渲染DOM
- JS可以修改DOM结构
- JS执行时，浏览器DOM渲染停止
- 两段JS不能同时执行（都修改DOM就冲突了）
- [web worker](http://www.ruanyifeng.com/blog/2018/07/web-worker.html)支持多线程，但是不能访问DOM

引入单线程就意味着，所有任务需要排队，前一个任务结束，才会执行后一个任务。这同时又导致了一个问题：如果前一个任务耗时很长，后一个任务就不得不一直等着。

```js
// 实例1
var i, sum = 0
for(i = 0; i < 1000000000; i ++) {
    sum += i
}
console.log(sum)

// 实例2
console.log(1)
alert('hello')
console.log(2)
```

在实例1中，`sum`并不能立刻打印出来，必须在for循环执行完成之后才能执行`console.log(sum)`。在实例2中，浏览器先打印`1`，然后弹出弹框，点击确定后才执行`console.log(2)`。

总结：

- 优点：实现比较简单，执行环境相对单纯
- 缺点：只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段Javascript代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。

为了解决这个问题，JS语言将任务的执行模式分为两种：同步和异步

## 同步与异步

1. 同步

   ```js
   A(args...)
   ```

   如果在函数A返回的时候，调用者就能够得到预期结果(即拿到了预期的返回值或者看到了预期的效果)，那么这个函数就是同步的。

   ```js
   let a = 1
   Math.floor(a)
   console.log(a)
   ```

2. 异步

   如果在函数A返回的时候，调用者还不能够得到预期结果，而是需要在将来通过一定的手段得到，那么这个函数就是异步的。

   ```js
   fs.readFile('foo.txt', 'utf8', function(err, data) {
       console.log(data);
   });
   ```

   JavaScript采用异步编程原因有两点，一是JavaScript是单线程，二是为了提高CPU的利用率。

## 异步过程

```js
fs.readFile('data.json', 'utf8', function(err, data) {
    console.log(data)
})
```

在执行这段代码时，`fs.readFile`函数返回时，并不会立刻打印`data`，只有`data.json`读取完成时才打印。也就是异步函数`fs.readFile`执行很快，但后面还有工作线程执行异步任务、通知主线程、主线程回调等操作，这个过程就叫做异步过程。

> 主线程发起一个异步操作，相应的工作线程接受请求并告知主线程已收到（异步函数返回）；只线程继续执行后面的任务，同时工作线程执行异步任务；工作线程完成任务后，通知主线程；主线程收到通知后，执行一定的动作（调用回调函数）。

工作线程在异步操作完成后通知主线程，那么这个通知机制又是如何显现喃？答案就是就是消息队列与事件循环。

## 消息队列与事件循环

> 工作线程将消息放在消息队列，主线程通过事件循环过程去取消息。

- 消息队列：消息队列是一个先进先出的队列，它里面存放着各种消息。
- 事件循环：事件循环是指主线程重复从消息队列中取消息、执行的过程。

1. 事件循环（eventloop）

主线成不断的从消息队列中取消息，执行消息，这个过程称为事件循环，这种机制叫事件循环机制，取一次消息并执行的过程叫一次循环。

大致实现过程如下：

```js
while(true) {
    var message = queue.get()
    execute(message)
}
```

举个例子：

```js
$.ajax({
    url: 'xxxx',
    success: function(result) {
        console.log(1)
    }
})
setTimeout(function() {
    console.log(2)
}, 100)
setTimeout(function() {
    console.log(3)
})
console.log(4)

// 主线程
console.log(4)

// 异步队列
function () {
    console.log(3)
}
function () { // 100ms后
    console.log(2)
}
function() { // ajax加载完成之后
    console.log(1)
}

// output：4321 或 4312
```

**事件循环是JS实现异步的具体解决方案，其中同步代码，直接执行；异步函数先放在异步队列中，待同步函数执行完毕后，轮询执行 异步队列 的回调函数。**

2. 消息队列

其中，消息就是注册异步任务时添加的回调函数。

```js
$.ajax('XXX', function(res) {
    console.log(res)
})
...
```

主线程在发起AJAX请求后，会继续执行其他代码，AJAX线程负责请求XXX，拿到请求后，会封装成JS对象，然后构造一条消息：

```js
// 消息队列里的消息
var message = function () {
    callback(response)
}
```

其中`callback`是AJAX网络请求成功响应时的回调函数。

主线程在执行完当前循环中的所有代码后，就会到消息队列取出这条消息(也就是`message`函数)，并执行它。到此为止，就完成了工作线程对主线程的`通知`，回调函数也就得到了执行。如果一开始主线程就没有提供回调函数，AJAX线程在收到HTTP响应后，也就没必要通知主线程，从而也没必要往消息队列放消息。

![异步过程](/Users/a123/Desktop/Study/JS/img/异步过程.png)

异步过程中的回调函数，一定不在当前这一轮事件循环中执行。

## 异步与事件

消息队列中的每条消息实际上都对应着一个事件。

其中一个重要的一步过程过程就是：**DOM事件**

```js
var button = document.getElementById('button1')
button.addEventLister('click', function(e) {
    console.log('事件')
})
```

从异步的角度看，`addEventLister`函数就是异步过程的发起函数，事件监听器函数就是异步过程的回调函数。事件触发时，表示异步任务完成，会将事件监听器函数封装成一条消息放在消息队列中，等待主线程执行。

事件的概念实际上并不是必须的，事件机制实际上就是异步过程的通知机制。我觉得它的存在就是为了编程接口对开发者更友好。

另一方面，所有的异步过程也都可以用事件来描述。例如：

```js
setTimeout(fn, 1000)
// 可以看成：
timer.addEventLister('timeout', 1000, fn)
```

其中关于事件的详细描述，[事件绑定、事件监听、事件委托](http://blog.xieliqun.com/2016/08/12/event-delegate/)

##  生产者与消费者

从生产者与消费者的角度看，异步过程是这样的：

> 工作线程是生产者，主线程是消费者(只有一个消费者)。工作线程执行异步任务，执行完成后把对应的回调函数封装成一条消息放到消息队列中；主线程不断地从消息队列中取消息并执行，当消息队列空时主线程阻塞，直到消息队列再次非空。

   那么异步的实现方式有哪些喃？

   - ES6之前：callback、eventloop、Promise
   - ES6：Generator
   - ES7:Async/Await

## step1：callback

```js
asyncFunction(function(value) {
    // todo
})
```

这种回调函数，大家是最熟悉的。一般是需要在某个耗时操作之后执行某个回调函数。

例如：

```js
setTimeout(function() {
    console.log('Time out')
}, 1000)
```

其中，我们称`setTimeout`为发起函数，`fn`为回调函数。都是在主线程上调用的，其中发起函数用来发动异步过程，回调函数用来处理结果。在执行`setTimeout`1s后，执行function函数。

下面，我们再看一种情况。

```js
$.ajax({
    url:'XXX1',
    success: function(res) {
        $.ajax({
            url:'XXX2',
            success: function(res) {
                $.ajax({
                    url: 'XXX3',
                    success: function(res) {
                        // todo
                    },
                    fail: function(err) {
                        console.log(err)
                    }
                })
            },
            fail: function(err) {
                console.log(err)
            }
        }) 
    },
    fail: function(err) {
    	console.log(err)
	}
})
```

在上例中，我们看到这段回调函数，不断的在回调，这只是三层回调，在实际应用中，我们遇到的需求会更复杂，回调也许更多，调试起来也就更麻烦，代码也更不美观，这就是我们要引入的第一个问题：回调地狱。

**问题1: 回调地狱**

回调地狱是JS里一个约定俗成的名称，一般情况下，一个业务依赖于上层业务，上层业务又依赖于更上一层的业务，以此类推，如果我们使用回调函数来处理异步的话，就会出现回调地狱。

主要是因为：大脑对业务的逻辑处理是线性的、阻塞的、单线程的，但是回调表达异步的方式是非线形的、非顺序的，这使得正确推导这类代码的难度很大，很容易出bug。

再例如：

```js
// A
$.ajax({
    ...
    success: function (...) {
        // C
    }
});
// B
```

A和B发生于现在，在JavaScript主程序的直接控制之下，而C会延迟到将来发生，并且是在第三方的控制下，在本例中就是函数$.ajax(...)。从根本上来说，这种控制的转移通常不会给程序带来很多问题。

但是，请不要被这个小概率迷惑而认为这种控制切换不是什么大问题。实际上，这是回调驱动设计最严重（也是最微妙）的问题。它以这样一个思路为中心：有时候ajax(...)，也就是你交付回调函数的第三方不是你编写的代码，也不在你的直接控制之下，它是某个第三方提供的工具。

这种情况称为**控制反转**，也就是把自己程序一部分的执行控制交给某个第三方，在你的代码和第三方工具直接有一份并没有明确表达的契约。

既然是无法控制的第三方在执行你的回调函数，那么就有可能存在以下问题，当然通常情况下是不会发生的：

1. 调用回调过早
2. 调用回调过晚
3. 调用回调次数太多或者太少
4. 未能把所需的参数成功传给你的回调函数
5. 吞掉可能出现的错误或异常
6. ......

这种控制反转会导致信任链的完全断裂，如果你没有采取行动来解决这些控制反转导致的信任问题，那么你的代码已经有了隐藏的Bug，尽管我们大多数人都没有这样做。

这里，我们引出了回调函数处理异步的第二个问题：控制反转。

**问题2：控制反转**

综上，回调函数处理异步流程存在2个问题：

**1. 缺乏顺序性： 回调地狱导致的调试困难，和大脑的思维方式不符**

**2. 缺乏可信任性： 控制反转导致的一系列信任问题**

那么如何来解决这两个问题，先驱者们开始了探索之路......

## Step2：Promise

**Promise就是为了解决callback的问题而产生的。**

Promise 本质上就是一个绑定了回调的对象，而不是将回调传回函数内部。

**开门见山，Promise解决的是回调函数处理异步的第2个问题：控制反转**。

我们把上面那个多层回调嵌套的例子用Promise的方式重构：

```js
let getPromise1 = function () {
    return new Promsie(function (resolve, reject) {
        $.ajax({
            url: 'XXX1',
            success: function (data) {
               let key = data;
               resolve(key);         
            },
            error: function (err) {
                reject(err);
            }
        });
    });
};

let getPromise2 = function (key) {
    return new Promsie(function (resolve, reject) {
        $.ajax({
            url: 'XXX2',
            data: {
                key: key
            },
            success: function (data) {
                resolve(data);         
            },
            error: function (err) {
                reject(err);
            }
        });
    });
};

let getPromise3 = function () {
    
    return new Promsie(function (resolve, reject) {
        $.ajax({
            url: 'XXX3',
            success: function (data) {
                resolve(data);         
            },
            error: function (err) {
                reject(err);
            }
        });
    });
};

getPromise1()
    .then(function (key) {
        return getPromise2(key);
    })
    .then(function (data) {
        return getPromise3(data);
    })
    .then(function (data) {
    	// todo
        console.log('业务数据：', data);
    })
    .catch(function (err) {
        console.log(err);
    }); 
```

Promise 在一定程度上其实改善了回调函数的书写方式；另外逻辑性更明显了，将异步业务提取成单个函数，整个流程可以看到是一步步向下执行的，依赖层级也很清晰，最后需要的数据是在整个代码的最后一步获得。

所以，Promise在一定程度上解决了回调函数的书写结构问题，但回调函数依然在主流程上存在，只不过都放到了then(...)里面，和我们大脑顺序线性的思维逻辑还是有出入的。

### Promise 是什么

Promise是什么，无论是ES6的Promise也好，jQuery的Promise也好，不同的库有不同的实现，但是大家遵循的都是同一套规范，所以，Promise并不指特定的某个实现，**它是一种规范，是一套处理JavaScript异步的机制**。

Promise的规范会多，如Promise/A、Promise/B、Promise/D以及Promise/A的升级版Promise/A+，其中ES6遵循Promise/A+规范，有关Promise/A+，你可以参考一下：

- 英文版：[Promise/A+](https://promisesaplus.com)
- 翻译版：[【翻译】Promises/A+规范](http://www.ituring.com.cn/article/66566)

这里只简要介绍下几点与接下来内容相关的规范：

- Promise 本质是一个状态机，每个 Promise 有三种状态：pending、fulfilled以及rejected。状态转变只能是pending —> fulfilled 或者 pending —> rejected。状态转变不可逆。
- then 方法可以被同一个 promise 调用多次。
- then 方法必须返回一个 promise。规范2.2.7中规定， then 必须返回一个新的 Promise
- 值穿透

### Promise 实现及源码解读

首先，我们看一下Promise的简单使用：

```js
var p = new Promise(function(resolve, reject) {
    // Do an async task async task and then...
    if(/* good condition */) {
        resolve('Success!');
    }
    else {
        reject('Failure!');
    }
});
p.then(function() { 
    /* do something with the result */
}).catch(function() {
    /* error :( */
})
```

我们通过这种使用构建Promise实现的第一个版本

#### Promise构建版本一

```js
function MyPromise(callback) {
    var _this = this
    _this.value = void 0 // Promise的值
    var onResolvedCallback  // Promise resolve回调函数
    var onRejectedCallback  // Promise reject回调函数
    // resolve 处理函数
    _this.resolve = function (value) {
        onResolvedCallback()
    } 
    // reject 处理函数
    _this.reject = function (error) {
        onRejectedCallback()
    } 
    callback(_this.resolve, _this.reject) // 执行executor并传入相应的参数
}
// 添加 then 方法
MyPromise.prototype.then = function(resolve, reject) {}
```

大致框架已经出来了，但我们看到Promise状态、reslove函数、reject函数以及then等都没有处理。

#### Promise构建之二：链式存储

首先，举个例子：

```js
new Promise(function (resolve, reject) {
    setTimeout(function () {
        var a=1;
        resolve(a);
    }, 1000);
}).then(function (res) {
    console.log(res);
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            var b=2;
            resolve(b);
        }, 1000);
    })
}).then(function (res) {
    console.log(res);
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            var c=3
            resolve(c);
        }, 1000);
    })
}).then(function (res) {
    console.log(res);
})
```

上例结果是每间隔1s打印一个数字，顺序为1、2、3。

这里保证了： 

- 让a,b,c的值能在then里面的回调接收到
- 在连续调用异步，如何确保异步函数的执行顺序

Promise一个常见的需求就是连续执行两个或者多个异步操作，这种情况下，每一个后来的操作都在前面的操作执行成功之后，带着上一步操作所返回的结果开始执行。这里用`setTimeout`来处理

```js
function MyPromise(callback) {
    var _this = this
    _this.value = void 0 // Promise的值
    // 用于保存 then 的回调， 只有当 promise
    // 状态为 pending 时才会缓存，并且每个实例至多缓存一个
    _this.onResolvedCallbacks = [] // Promise resolve时的回调函数集
    _this.onRejectedCallbacks = [] // Promise reject时的回调函数集
    _this.resolve = function (value) {
        setTimeout(() => { // 异步执行
            _this.onResolvedCallbacks.forEach(cb => cb())
        })
    } // resolve 处理函数
    _this.reject = function (error) {
        setTimeout(() => { // 异步执行
            _this.onRejectedCallbacks.forEach(cb => cb())
        })
    } // reject 处理函数
    callback(_this.resolve, _this.reject) // 执行executor并传入相应的参数
}
// 添加 then 方法
MyPromise.prototype.then = function() {}
```

#### Promise构建之三：状态机制、顺序执行

为了保证Promise的异步操作时的顺序执行，这里给Promise加上状态机制

```js
// 三种状态
const PENDING = "pending"
const FULFILLED = "fulfilled"
const REJECTED = "rejected"
function MyPromise(callback) {
    var _this = this
    _this.currentState = PENDING // Promise当前的状态
    _this.value = void 0 // Promise的值
    // 用于保存 then 的回调， 只有当 promise
    // 状态为 pending 时才会缓存，并且每个实例至多缓存一个
    _this.onResolvedCallbacks = [] // Promise resolve时的回调函数集
    _this.onRejectedCallbacks = [] // Promise reject时的回调函数集
    _this.resolve = function (value) {
        setTimeout(() => { // 异步执行，保证顺序执行
            if (_this.currentState === PENDING) {
                _this.currentState = FULFILLED // 状态管理
                _this.value = value
                _this.onResolvedCallbacks.forEach(cb => cb())
            }
        })
    } // resolve 处理函数
    _this.reject = function (error) {
        setTimeout(() => { // 异步执行，保证顺序执行
            if (_this.currentState === PENDING) {
            	_this.currentState = REJECTED // 状态管理
            	_this.value = value
            	_this.onRejectedCallbacks.forEach(cb => cb())
        	}
        })
    } // reject 处理函数
    callback(_this.resolve, _this.reject) // 执行executor并传入相应的参数
}
// 添加 then 方法
MyPromise.prototype.then = function() {}
```

#### Promise构建之四：递归执行

每个Promise后面链一个对象该对象包含onfulfiled,onrejected,子promise三个属性.

当父Promise 状态改变完毕,执行完相应的onfulfiled/onrejected的时候，拿到子promise,在等待这个子promise状态改变，在执行相应的onfulfiled/onrejected。依次循环直到当前promise没有子promise。

```js
// 三种状态
const PENDING = "pending"
const FULFILLED = "fulfilled"
const REJECTED = "rejected"
function MyPromise(callback) {
    var _this = this
    _this.currentState = PENDING // Promise当前的状态
    _this.value = void 0 // Promise的值
    // 用于保存 then 的回调， 只有当 promise
    // 状态为 pending 时才会缓存，并且每个实例至多缓存一个
    _this.onResolvedCallbacks = [] // Promise resolve时的回调函数集
    _this.onRejectedCallbacks = [] // Promise reject时的回调函数集
    _this.resolve = function (value) {
        if (value instanceof MyPromise) {
            // 如果 value 是个 Promise， 递归执行
            return value.then(_this.resolve, _this.reject)
        }
        setTimeout(() => { // 异步执行，保证顺序执行
            if (_this.currentState === PENDING) {
                _this.currentState = FULFILLED // 状态管理
                _this.value = value
                _this.onResolvedCallbacks.forEach(cb => cb())
            }
        })
    } // resolve 处理函数
    _this.reject = function (error) {
        setTimeout(() => { // 异步执行，保证顺序执行
            if (_this.currentState === PENDING) {
            	_this.currentState = REJECTED // 状态管理
            	_this.value = value
            	_this.onRejectedCallbacks.forEach(cb => cb())
        	}
        })
    } // reject 处理函数
    callback(_this.resolve, _this.reject) // 执行executor并传入相应的参数
}
// 添加 then 方法
MyPromise.prototype.then = function() {}
```

#### Promise构建之五：异常处理

```js
// 三种状态
const PENDING = "pending"
const FULFILLED = "fulfilled"
const REJECTED = "rejected"
function MyPromise(callback) {
    var self = this
    self.currentState = PENDING // Promise当前的状态
    self.value = void 0 // Promise的值
    // 用于保存 then 的回调， 只有当 promise
    // 状态为 pending 时才会缓存，并且每个实例至多缓存一个
    self.onResolvedCallbacks = [] // Promise resolve时的回调函数集
    self.onRejectedCallbacks = [] // Promise reject时的回调函数集
    self.resolve = function (value) {
        if (value instanceof MyPromise) {
            // 如果 value 是个 Promise， 递归执行
            return value.then(self.resolve, self.reject)
        }
        setTimeout(() => { // 异步执行，保证顺序执行
            if (self.currentState === PENDING) {
                self.currentState = FULFILLED // 状态管理
                self.value = value
                self.onResolvedCallbacks.forEach(cb => cb())
            }
        })
    } // resolve 处理函数
    self.reject = function (error) {
        setTimeout(() => { // 异步执行，保证顺序执行
            if (self.currentState === PENDING) {
            	self.currentState = REJECTED // 状态管理
            	self.value = value
            	self.onRejectedCallbacks.forEach(cb => cb())
        	}
        })
    } // reject 处理函数
    
    // 异常处理
    // new Promise(() => throw Error('error'))
    try {
        callback(self.resolve, self.reject) // 执行executor并传入相应的参数
    } catch(e) {
        self.reject(e)
    }
}
// 添加 then 方法
MyPromise.prototype.then = function() {}
```

#### Promise构建之六：then的实现

then 方法是 Promise 的核心，这里做一下详细介绍。

```js
promise.then(onFulfilled, onRejected)
```

一个 Promise 的then接受两个参数： onFulfilled和onRejected（都是可选参数，并且为函数，若不是函数将被忽略）

- onFulfilled 特性：

  - 当 Promise 执行结束后其必须被调用，其第一个参数为 promise 的终值，也就是 resolve 传过来的值
  - 在 Promise 执行结束前不可被调用
  - 其调用次数不可超过一次

- onRejected 特性

  - 当 Promise 被拒绝执行后其必须被调用，第一个参数为 Promise 的拒绝原因，也就是reject传过来的值
  - 在 Promise 执行结束前不可被调用
  - 其调用次数不可超过一次

- 调用时机

  `onFulfilled` 和 `onRejected` 只有在[执行环境](http://es5.github.io/#x10.3)堆栈仅包含**平台代码**时才可被调用（平台代码指引擎、环境以及 promise 的实施代码）

- 调用要求

  `onFulfilled` 和 `onRejected` 必须被作为函数调用（即没有 `this` 值，在 **严格模式（strict）** 中，函数 `this` 的值为 `undefined` ；在非严格模式中其为全局对象。）

- 多次调用

  `then` 方法可以被同一个 `promise` 调用多次

  - 当 `promise` 成功执行时，所有 `onFulfilled` 需按照其注册顺序依次回调
  - 当 `promise` 被拒绝执行时，所有的 `onRejected` 需按照其注册顺序依次回调

- 返回

  `then`方法会返回一个`Promise`，关于这一点，Promise/A+标准并没有要求返回的这个Promise是一个新的对象，但在Promise/A标准中，明确规定了then要返回一个新的对象，目前的Promise实现中then几乎都是返回一个新的Promise([详情](https://promisesaplus.com/differences-from-promises-a#point-5))对象，所以在我们的实现中，也让then返回一个新的Promise对象。 

  ```
  promise2 = promise1.then(onFulfilled, onRejected);
  ```

  - 如果 `onFulfilled` 或者 `onRejected` 返回一个值 `x` ，则运行下面的 **Promise 解决过程**：`[[Resolve]](promise2, x)`
  - 如果 `onFulfilled` 或者 `onRejected` 抛出一个异常 `e` ，则 `promise2` 必须拒绝执行，并返回拒因 `e`
  - 如果 `onFulfilled` 不是函数且 `promise1` 成功执行， `promise2` 必须成功执行并返回相同的值
  - 如果 `onRejected` 不是函数且 `promise1` 拒绝执行， `promise2` 必须拒绝执行并返回相同的拒因

  **不论 promise1 被 reject 还是被 resolve ， promise2 都会被 resolve，只有出现异常时才会被 rejected**。

     每个Promise对象都可以在其上多次调用then方法，而每次调用then返回的Promise的状态取决于那一次调用then时传入参数的返回值，所以then不能返回this，因为then每次返回的Promise的结果都有可能不同。

下面代码实现：

```js
// then 方法接受两个参数，onFulfilled，onRejected，分别为Promise成功或失败的回调
MyPromise.prototype.then = function(onFulfilled, onRejected) {
    var self = this
    // 规范 2.2.7，then 必须返回一个新的 promise
    var promise2
    // 根据规范 2.2.1 ，onFulfilled、onRejected 都是可选参数
    // onFulfilled、onRejected不是函数需要忽略，同时也实现了值穿透
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    onRejected = typeof onRejected === 'function' ? onRejected : error => {throw error}
    
    if (self.currentState === FULFILLED) {
        return promise2 = new MyPromise(function(resolve, reject) {
            
        })
    }
    if (self.currentState === REJECTED) {
        return promise2 = new MyPromise(function(resolve, reject) {
            
        })
    }
    if (self.currentState === PENDING) {
        return promise2 = new MyPromise(function(resolve, reject) {
            
        })
    }
}
```

**附：值穿透解读**

```js
MyPromise.prototype.then = function (onFulfilled, onRejected) {
    ...
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    onRejected = typeof onRejected === 'function' ? onRejected : error => {throw error}
    ...
}
```

上面提到值穿透，值穿透即：

```js
var promise = new MyPromise((resolve, reject) => {
    setTimeout(() => {
        resolve('1')
    }, 1000)
})
promise.then('2').then(console.log)
```

最终打结果是`1`而不是`2`

再例如：

```js
new MyPromise(resolve => resolve('1'))
    .then()
    .then()
    .then(function foo(value) {
        alert(value)
    })
// output: alert 出 1
```

通过 `return this` 只实现了值穿透的一种情况，其实值穿透有两种情况：

1. promise 已经是 FULFILLED/REJECTED 时，通过 return this 实现的值穿透：

   ```js
   var promise = new Promise(function (resolve) {
       setTimeout(() => {
           resolve('1')
       }, 1000)
   })
   promise.then(() => {
       promise.then().then((res) => { // 状况A
           console.log(res) // output: 1
       })
       promise.catch().then((res) => { // 状况B
           console.log(res) // output: 1
       })
       console.log(promise.then() === promise.catch()) // output: true
       console.log(promise.then(1) === promise.catch({name: 'anran'})) // output: true
   })
   ```

   状况A与B处 promise 已经是 FULFILLED 了符合条件，所以执行了 `return this`。

   注意：原生的Promise实现里并不是这样实现的，会打印出两个false

2. promise 是 PENDING时，通过生成新的 promise 加入到父 promise 的 queue，父 promise 有值时调用 callFulfilled->doResolve 或 callRejected->doReject（因为 then/catch 传入的参数不是函数）设置子 promise 的状态和值为父 promise 的状态与值。如：

   ```js
   var promise = new Promise((resolve) => {
       setTimeout(() => {
           resolve('1')
       }, 1000)
   })
   var a = promise.then()
   a.then((res) => {
       console.log(res) // output: 1
   })
   var b = promise.catch()
   b.then((res) => {
       console.log(res) // output: 1
   })
   console.log(a === b) // output: false
   ```

   

Promise 有三种状态，我们分3个if块来处理，每块都返回一个new Promise。

根据标准，我们知道，对于一下代码，promise2的值取决于then里面的返回值：

```js
promise2 = promise1.then(function(value) {
    return 1
}, function(err) {
    throw new Error('error')
})
```

如果promise1被resolve了，promise2的被`1`resolve，如果promise1 被reject了，promise2将被`new Error('error')`reject。

所以，我们需要在then里面执行onFulfilled或者onRejected，并根据返回着（标记中记为`x`）来确定promise2的结果，并且，如果onFulfilled/onRejected返回的是一个Promise，promise将直接取这个Promise的结果。

```js
// then 方法接受两个参数，onFulfilled，onRejected，分别为Promise成功或失败的回调
MyPromise.prototype.then = function(onFulfilled, onRejected) {
    var self = this
    // 规范 2.2.7，then 必须返回一个新的 promise
    var promise2
    // 根据规范 2.2.1 ，onFulfilled、onRejected 都是可选参数
    // onFulfilled、onRejected不是函数需要忽略，同时也实现了值穿透
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    onRejected = typeof onRejected === 'function' ? onRejected : error => {throw error}
    
    if (self.currentState === FULFILLED) {
        // 如果promise1（此处为self/this）的状态已经确定并且为fulfilled，我们调用onFulfilled
        // 如果考虑到有可能throw，所以我们将其包在try/catch块中
        return promise2 = new MyPromise(function(resolve, reject) {
            // 规范 2.2.4，保证 onFulfilled，onRjected 异步执行
      		// 所以用了 setTimeout 包裹下
            setTimeout(function() {
                try {
                	var x = onFulfilled(self.value)
                	// 如果 onFulfilled 的返回值是一个 Promise 对象，直接取它的结果作为 promise2 的结果
                	if (x instanceof MyPromise) {
                    	x.then(resolve, reject)
                	}
                	resolve(x) // 否则，以它的返回值为 promise2 的结果
            	} catch (err) {
                	reject(err) // 如果出错，以捕获到的错误作为promise2的结果
            	}
            })
        })
    }
    // 此处实现与FULFILLED相似，区别在使用的是onRejected而不是onFulfilled
    if (self.currentState === REJECTED) {
        return promise2 = new MyPromise(function(resolve, reject) {
            setTimeout(function() {
                try {
                	var x = onRejected(self.value)
                	if (x instanceof Promise){
                    	x.then(resolve, reject)
                	}
            	} catch(err) {
                	reject(err)
            	}
            })
        })
    }
    if (self.currentState === PENDING) {
        // 如果当前的Promise还处于PENDING状态，我们并不能确定调用onFulfilled还是onRejected
        // 只有等待Promise的状态确定后，再做处理
        // 所以我们需要把我们的两种情况的处理逻辑做成callback放入promise1（此处即self/this）的回调数组内
        // 处理逻辑和以上相似
        return promise2 = new MyPromise(function(resolve, reject) {
            self.onResolvedCallbacks.push(function() {
                try {
                    var x = onFulfilled(self.value)
                    if (x instanceof MyPromise) {
                        x.then(resolve, reject)
                    }
                    resolve(x)
                } catch(err) {
                    reject(err)
                }
            })
            self.onRejectedCallbacks.push(function() {
                try {
                    var x = onRejected(self.value)
                    if (x instanceof MyPromise) {
                        x.then(resolve, reject)
                    }
                } catch (err) {
                    reject(err)
                }
            })
        })
    }
}
```

#### Promise构建之七：catch的实现

```js
// catch 的实现
MyPromise.prototype.catch = function (onRejected) {
    return this.then(null, onRejected)
}
```

至此，我们大致实现了Promise标准中所涉及到的内容。

#### Promise构建之八：问题补充：无缝调用

不同的Promise实现之间需要无缝的可交互，如ES6的Promise，和我们自己实现的Promise之间以及其他的Promise实现，必须是无缝调用的。

```js
new MyPromise(function(resolve, reject) {
    setTimeout(function() {
        resolve('1')
    }, 1000)
}).then(function() {
    return new Promise.reject('2') // ES6 的 Promise
}).then(function() {
    return Q.all([ // Q 的 Promise
        new MyPromise(resolve => resolve('3')) // 我们实现的Promise
        new Promise.eresolve('4') // ES6 的 Promise
        Q.resolve('5') // Q 的 Promise
    ])
})
```

我之前实现的代码只是判断OnFullfilled/onRejected的返回值是否为我们自己实现的实例，并没有对其他类型Promise的判断，所以，上面的代码无法正常运行。

接下来，我们解决这个问题

关于不同Promise之间的交互，其实[Promise/A+标准](https://promisesaplus.com/#point-46)中有介绍，其中详细的指定了如何通过then的实参返回的值来决定promise2的状态，我们只需要按照标准把标准的内容转成代码即可。

即我们要**把onFulfilled/onRejected的返回值x。当成是一个可能是Promise的对象**，也即标准中的thenable，并以最保险的姿势调用x上的then方法，如果大家都按照标准来实现，那么不同的Promise之间就可以交互了。

而标准为了保险起见，即使x返回了一个带有then属性但不遵循Promise标准的对象（不如说这个x把它then里的两个参数都调用了，同步或者异步调用（PS，原则上then的两个参数需要异步调用，下文会讲到），或者是出错后又调用了它们，或者then根本不是一个函数），也能尽可能正确处理。

关于为何需要不同的Promise实现能够相互交互，我想原因应该是显然的，Promise并不是JS一早就有的标准，不同第三方的实现之间是并不相互知晓的，如果你使用的某一个库中封装了一个Promise实现，想象一下如果它不能跟你自己使用的Promise实现交互的场景。。。

代码实现：

```js
// 规范 2.3
/*
resolutionProcedure函数即为根据x的值来决定promise2的状态的函数
也即标准中的[Promise Resolution Procedure](https://promisesaplus.com/#point-47)
x 为 promise2 = promise1.then(onFulfilled, onRejected)里onFulfilled/onRejected的返回值
resolve 和 reject 实际上是 promise2 的executor的两个实参，因为很难挂在其他地方，所以一并传过来。
相信各位一定可以对照标准转换成代码，这里就只标出代码在标准中对应的位置，只在必要的地方做一些解释。
*/
function resolutionProcedure(promise2, x, resolve, reject) {
    // 规范 2.3.1，x 不能和 promise2 相同，避免循环引用
    if (promise2 === x) {
        return reject(new TypeError("Chaining cycle detected for promise!"))
    }
    // 规范 2.3.2
    // 如果 x 为 Promise，状态为 pending 需要继续等待否则执行
    if (x instanceof MyPromise) {
        // 2.3.2.1 如果x为pending状态，promise必须保持pending状态，直到x为fulfilled/rejected
        if (x.currentState === PENDING) {
            x.then(function(value) {
                // 再次调用该函数是为了确认 x resolve 的
                // 参数是什么类型，如果是基本类型就再次 resolve
                // 把值传给下个 then
                resolutionProcedure(promise2, value, resolve, reject)
            }, reject)
        } else { // 但如果这个promise的状态已经确定了，那么它肯定有一个正常的值，而不是一个thenable，所以这里可以取它的状态
            x.then(resolve, reject)
        }
        return
    }
    
    let called = false
    // 规范 2.3.3，判断 x 是否为对象或函数
    if (x !== null && (typeof x === "object" || typeof x === "function")) {
        // 规范 2.3.3.2，如果不能取出 then，就 reject
        try {
            // 规范2.3.3.1 因为x.then可能是一个getter，这种情况下多次读取就有可能产生副作用
            // 既要判断它的类型，又要调用它，这就是两次读取
            let then = x.then
            // 规范2.3.3.3，如果 then 是函数，调用 x.then
            if (typeof then === "function") {
                // 规范 2.3.3.3
    			// reject 或 reject 其中一个执行过的话，忽略其他的
                then.call(
                    x,
                    y => { // 规范 2.3.3.3.1
                        if (called) return // 规范 2.3.3.3.3，即这三处谁先执行就以谁的结果为准
                        called = true
                        // 规范 2.3.3.3.1
                        return resolutionProcedure(promise2, y, resolve, reject)
                    },
                    r => {
                        if (called) return // 规范 2.3.3.3.3，即这三处谁先执行就以谁的结果为准
                        called = true
                         return reject(r)
                    }
                )
            } else {
                // 规范 2.3.3.4
                resolve(x)
            }
        } catch (e) { // 规范 2.3.3.2
            if (called) return // 规范 2.3.3.3.3，即这三处谁先执行就以谁的结果为准
            called = true
            return reject(e)
        }
    } else {
        // 规范 2.3.4，x 为基本类型
        resolve(x)
    }
}
```

然后，我们使用`resolutionProcedure`函数替换`MyPromise.prototype.then`里面几处判断x是否为MyPromise对象的位置即可。即：

```js
if (x instanceof MyPromise) {
    x.then(resolve, reject)
}
```

替换为：

```js
resolutionProcedure(promise2, x, resolve, reject)
```

总共四处，不要遗漏了

#### Promise构建九：完整代码实现

```js
// 三种状态
const PENDING = "pending"
const FULFILLED = "fulfilled"
const REJECTED = "rejected"
function MyPromise(callback) {
    var self = this
    self.currentState = PENDING // Promise当前的状态
    self.value = void 0 // Promise的值
    // 用于保存 then 的回调， 只有当 promise
    // 状态为 pending 时才会缓存，并且每个实例至多缓存一个
    self.onResolvedCallbacks = [] // Promise resolve时的回调函数集
    self.onRejectedCallbacks = [] // Promise reject时的回调函数集
    self.resolve = function (value) {
        if (value instanceof MyPromise) {
            // 如果 value 是个 Promise， 递归执行
            return value.then(self.resolve, self.reject)
        }
        setTimeout(() => { // 异步执行，保证顺序执行
            if (self.currentState === PENDING) {
                self.currentState = FULFILLED // 状态管理
                self.value = value
                self.onResolvedCallbacks.forEach(cb => cb())
            }
        })
    } // resolve 处理函数
    self.reject = function (error) {
        setTimeout(() => { // 异步执行，保证顺序执行
            if (self.currentState === PENDING) {
                self.currentState = REJECTED // 状态管理
                self.value = value
                self.onRejectedCallbacks.forEach(cb => cb())
            }
        })
    } // reject 处理函数

    // 异常处理
    // new Promise(() => throw Error('error'))
    try {
        callback(self.resolve, self.reject) // 执行executor并传入相应的参数
    } catch(e) {
        self.reject(e)
    }
}
// then 方法接受两个参数，onFulfilled，onRejected，分别为Promise成功或失败的回调
MyPromise.prototype.then = function(onFulfilled, onRejected) {
    var self = this
    // 规范 2.2.7，then 必须返回一个新的 promise
    var promise2
    // 根据规范 2.2.1 ，onFulfilled、onRejected 都是可选参数
    // onFulfilled、onRejected不是函数需要忽略，同时也实现了值穿透
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    onRejected = typeof onRejected === 'function' ? onRejected : error => {throw error}

    if (self.currentState === FULFILLED) {
        // 如果promise1（此处为self/this）的状态已经确定并且为fulfilled，我们调用onFulfilled
        // 如果考虑到有可能throw，所以我们将其包在try/catch块中
        return promise2 = new MyPromise(function(resolve, reject) {
            try {
                var x = onFulfilled(self.value)
                // 如果 onFulfilled 的返回值是一个 Promise 对象，直接取它的结果作为 promise2 的结果
                resolutionProcedure(promise2, x, resolve, reject)
                resolve(x) // 否则，以它的返回值为 promise2 的结果
            } catch (err) {
                reject(err) // 如果出错，以捕获到的错误作为promise2的结果
            }
        })
    }
    // 此处实现与FULFILLED相似，区别在使用的是onRejected而不是onFulfilled
    if (self.currentState === REJECTED) {
        return promise2 = new MyPromise(function(resolve, reject) {
            try {
                var x = onRejected(self.value)
                resolutionProcedure(promise2, x, resolve, reject)
            } catch(err) {
                reject(err)
            }
        })
    }
    if (self.currentState === PENDING) {
        // 如果当前的Promise还处于PENDING状态，我们并不能确定调用onFulfilled还是onRejected
        // 只有等待Promise的状态确定后，再做处理
        // 所以我们需要把我们的两种情况的处理逻辑做成callback放入promise1（此处即self/this）的回调数组内
        // 处理逻辑和以上相似
        return promise2 = new MyPromise(function(resolve, reject) {
            self.onResolvedCallbacks.push(function() {
                try {
                    var x = onFulfilled(self.value)
                    resolutionProcedure(promise2, x, resolve, reject)
                    resolve(x)
                } catch(err) {
                    reject(err)
                }
            })
            self.onRejectedCallbacks.push(function() {
                try {
                    var x = onRejected(self.value)
                    resolutionProcedure(promise2, x, resolve, reject)
                } catch (err) {
                    reject(err)
                }
            })
        })
    }

    // 规范 2.3
    /*
    resolutionProcedure函数即为根据x的值来决定promise2的状态的函数
    也即标准中的[Promise Resolution Procedure](https://promisesaplus.com/#point-47)
    x 为 promise2 = promise1.then(onFulfilled, onRejected)里onFulfilled/onRejected的返回值
    resolve 和 reject 实际上是 promise2 的executor的两个实参，因为很难挂在其他地方，所以一并传过来。
    相信各位一定可以对照标准转换成代码，这里就只标出代码在标准中对应的位置，只在必要的地方做一些解释。
    */
    function resolutionProcedure(promise2, x, resolve, reject) {
        // 规范 2.3.1，x 不能和 promise2 相同，避免循环引用
        if (promise2 === x) {
            return reject(new TypeError("Chaining cycle detected for promise!"))
        }
        // 规范 2.3.2
        // 如果 x 为 Promise，状态为 pending 需要继续等待否则执行
        if (x instanceof MyPromise) {
            // 2.3.2.1 如果x为pending状态，promise必须保持pending状态，直到x为fulfilled/rejected
            if (x.currentState === PENDING) {
                x.then(function(value) {
                    // 再次调用该函数是为了确认 x resolve 的
                    // 参数是什么类型，如果是基本类型就再次 resolve
                    // 把值传给下个 then
                    resolutionProcedure(promise2, value, resolve, reject)
                }, reject)
            } else { // 但如果这个promise的状态已经确定了，那么它肯定有一个正常的值，而不是一个thenable，所以这里可以取它的状态
                x.then(resolve, reject)
            }
            return
        }

        let called = false
        // 规范 2.3.3，判断 x 是否为对象或函数
        if (x !== null && (typeof x === "object" || typeof x === "function")) {
            // 规范 2.3.3.2，如果不能取出 then，就 reject
            try {
                // 规范2.3.3.1 因为x.then可能是一个getter，这种情况下多次读取就有可能产生副作用
                // 既要判断它的类型，又要调用它，这就是两次读取
                let then = x.then
                // 规范2.3.3.3，如果 then 是函数，调用 x.then
                if (typeof then === "function") {
                    // 规范 2.3.3.3
                    // reject 或 reject 其中一个执行过的话，忽略其他的
                    then.call(
                        x,
                        y => { // 规范 2.3.3.3.1
                            if (called) return // 规范 2.3.3.3.3，即这三处谁先执行就以谁的结果为准
                            called = true
                            // 规范 2.3.3.3.1
                            return resolutionProcedure(promise2, y, resolve, reject)
                        },
                        r => {
                            if (called) return // 规范 2.3.3.3.3，即这三处谁先执行就以谁的结果为准
                            called = true
                            return reject(r)
                        }
                    )
                } else {
                    // 规范 2.3.3.4
                    resolve(x)
                }
            } catch (e) { // 规范 2.3.3.2
                if (called) return // 规范 2.3.3.3.3，即这三处谁先执行就以谁的结果为准
                called = true
                return reject(e)
            }
        } else {
            // 规范 2.3.4，x 为基本类型
            resolve(x)
        }
    }
}
// catch 的实现
MyPromise.prototype.catch = function (onRejected) {
    return this.then(null, onRejected)
}
```

终于实现搞定了，继续加油

 

## Step3：Generator

### Generator 是什么

**生成器（Generator）**对象是ES6中新增的语法，和Promise一样，都可以用来异步编程。但与Promise不同的是，它不是使用JS现有能力按照一定标准制定出来的，而是一种新型底层操作，async/await就是在它的基础上实现的。

generator对象是由generator function返回的，符合**可迭代协议**和**迭代器协议**。

generator function 可以在JS单线程的背景下，使JS的执行权与数据自由的游走在多个执行栈之间，实现协同开发编程，当项目调用generator function时，会在内部开辟一个单独的执行栈，在执行一个generator function 中，可以暂停执行，或去执行另一个generator function，而当前generator function并不会销毁，而是处于一种被暂停的状态，当执行权回来的时候，再继续执行。

![迭代器](/Users/a123/Desktop/Study/JS/img/迭代器.png)



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

所以对这个迭代器遍历，不会对值`1`处理。当在生成器函数中显式 `return `时，会导致生成器立即变为完成状态，即调用 `next()` 方法返回的对象的 `done `为 `true`。如果 `return `后面跟了一个值，那么这个值会作为**当前**调用 `next()` 方法返回的 value 值，若  `return `后面没有跟任何值，则返回 `undefined`，相当于默认 `return undefined`。

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

为什么会这样喃？这是因为调用 `next()`方法时，如果传入了参数，那么这个参数会作为**上一条执行的  yield 语句的返回值**。如果没有，则上一条执行的 yield 语句的返回值默认为 NaN，因为在处理的时候，我们没有给 next 函数传值，导致 yield 语句返回值为 NaN，则 val 为NaN，NaN + 1 当然也是 NaN了。

#### yield*

yield * 也称为 yield 委托。作用是将执行权交给另一个生成器或可迭代对象。（当前生成器暂停执行）。`yield*` 表达式迭代操作数，并产生它返回的每个值。

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

**注意：`yield*` 是一个表达式，不是语句，所以它会有自己的值，即为当前迭代器关闭时返回的值（即`done`为`true`时）。**

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



## Step4：async/await

### async Function

MDN 上介绍说，`async function` 声明用于定义一个返回 `AsyncFunction` 对象的异步函数。

其中： 异步函数是指一个使用隐式的`Promise`返回其结果，异步函数旨在让它的语法和结构更像是标准的同步函数。

举个例子：

```js
async function asyncFun() {
    return 'async function'
}
```

当这个异步函数调用时，将返回一个 `Promise`，我们可以获取到它的 `Promise` 值，对它使用 `then` 函数。

```js
const p = asyncFun() // 相当于 Promise
p.then(console.log) // 打印：async function
```

只有 p 在下一次运行 `microtask` 时才能得此 `Promise` 值，所以，上面的程序等同于对值调用 `Promise.resolve`。

即，等同于：

```js
async function asyncFun() {
    return Promise.resolve('async function')
}
```

若 `Promise` 处理异常(`rejected`)，`await` 表达式会把 `Promise` 的异常原因抛出。

### await

上面说到，`async Function` 相当于给将 `return` 结果放在 `Promise.resolve` 中返回。那 `await` 喃？

`await` 会暂停函数的执行直到 `Promise` 状态由 `pending -> resolved`，并将 `resolved` 的结果返回。

```js
async function fetchAsync() {
    const result = await asyncFun()
    console.log(result) // async function
}
```

 `await` 暂停了 `fetchAsync` 的执行，稍后在 `fetch` 返回的 `Promise` 状态变为 `resolved` 时恢复了执行，这或多或少等同于把处理过程写在 `asyncFun` 返回的 `Promise` 的 `then` 链上了。

相当于：

```js
function fetchAsync() {
    asyncFun().then(res => {
        console.log(res) // async function
    })
}
```

值得注意的是：`await` 可以使用任何 `thenable`，即任何带有 `then` 方法的对象，即使它不是真正的 `Promise`。

```js
class Sleep {
    constructor(timeout) {
        this.timeout = timeout
    }
    then(resolve, reject) {
        const startTime = Date.now()
        setTimeout(() => resolve(Date.now() - startTime), this.timeout)
    }
}
(async () => {
    const actualTime = await new Sleep(1000)
    console.log(actualTime)
})()
```



### 引擎底部的 await

