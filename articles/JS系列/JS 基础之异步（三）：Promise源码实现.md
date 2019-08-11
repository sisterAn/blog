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

- Promise 本质是一个状态机，每个 Promise 有三种状态：pending、resolved以及rejected。状态转变只能是pending —> resolved 或者 pending —> rejected。状态转变不可逆。
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
    callback(_this.resolve, _this.reject) // 执行callback并传入相应的参数
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
    callback(_this.resolve, _this.reject) // 执行callback并传入相应的参数
}
// 添加 then 方法
MyPromise.prototype.then = function() {}
```

#### Promise构建之三：状态机制、顺序执行

为了保证Promise的异步操作时的顺序执行，这里给Promise加上状态机制

```js
// 三种状态
const PENDING = "pending"
const RESOLVED = "resolved"
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
                _this.currentState = RESOLVED // 状态管理
                _this.value = value
                _this.onResolvedCallbacks.forEach(cb => cb())
            }
        })
    } // resolve 处理函数
    _this.reject = function (value) {
        setTimeout(() => { // 异步执行，保证顺序执行
            if (_this.currentState === PENDING) {
            	_this.currentState = REJECTED // 状态管理
            	_this.value = value
            	_this.onRejectedCallbacks.forEach(cb => cb())
        	}
        })
    } // reject 处理函数
    callback(_this.resolve, _this.reject) // 执行callback并传入相应的参数
}
// 添加 then 方法
MyPromise.prototype.then = function() {}
```

#### Promise构建之四：递归执行

每个Promise后面链接一个对象，该对象包含onresolved,onrejected,子promise三个属性.

当父Promise 状态改变完毕,执行完相应的onresolved/onrejected的时候，拿到子promise,在等待这个子promise状态改变，在执行相应的onresolved/onrejected。依次循环直到当前promise没有子promise。

```js
// 三种状态
const PENDING = "pending"
const RESOLVED = "resolved"
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
            // 如果 value 是个 MyPromise， 递归执行
            return value.then(_this.resolve, _this.reject)
        }
        setTimeout(() => { // 异步执行，保证顺序执行
            if (_this.currentState === PENDING) {
                _this.currentState = RESOLVED // 状态管理
                _this.value = value
                _this.onResolvedCallbacks.forEach(cb => cb())
            }
        })
    } // resolve 处理函数
    _this.reject = function (value) {
        setTimeout(() => { // 异步执行，保证顺序执行
            if (_this.currentState === PENDING) {
            	_this.currentState = REJECTED // 状态管理
            	_this.value = value
            	_this.onRejectedCallbacks.forEach(cb => cb())
        	}
        })
    } // reject 处理函数
    callback(_this.resolve, _this.reject) // 执行callback并传入相应的参数
}
// 添加 then 方法
MyPromise.prototype.then = function() {}
```

#### Promise构建之五：异常处理

```js
// 三种状态
const PENDING = "pending"
const RESOLVED = "resolved"
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
            // 如果 value 是个 MyPromise， 递归执行
            return value.then(_this.resolve, _this.reject)
        }
        setTimeout(() => { // 异步执行，保证顺序执行
            if (_this.currentState === PENDING) {
                _this.currentState = RESOLVED // 状态管理
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
    
    // 异常处理
    // new Promise(() => throw Error('error'))
    try {
        callback(_this.resolve, _this.reject) // 执行callback并传入相应的参数
    } catch(e) {
        _this.reject(e)
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

  `onFulfilled` 和 `onRejected` 只有在[执行环境](http://es5.github.io/#x10.3)堆栈仅包含**平台代码**时才可被调用（平台代码指引擎、环境以及 promise 的实施代码）

- 调用要求

  `onFulfilled` 和 `onRejected` 必须被作为函数调用（即没有 `this` 值，在 **严格模式（strict）** 中，函数 `this` 的值为 `undefined` ；在非严格模式中其为全局对象。）

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
    var _this = this
    // 规范 2.2.7，then 必须返回一个新的 promise
    var promise2
    // 根据规范 2.2.1 ，onFulfilled、onRejected 都是可选参数
    // onFulfilled、onRejected不是函数需要忽略，同时也实现了值穿透
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    onRejected = typeof onRejected === 'function' ? onRejected : error => {throw error}
    
    if (_this.currentState === RESOLVED) {
        return promise2 = new MyPromise(function(resolve, reject) {
            
        })
    }
    if (_this.currentState === REJECTED) {
        return promise2 = new MyPromise(function(resolve, reject) {
            
        })
    }
    if (_this.currentState === PENDING) {
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

1. promise 已经是 RESOLVED/REJECTED 时，通过 return this 实现的值穿透：

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

   状况A与B处 promise 已经是 RESOLVED 了符合条件，所以执行了 `return this`。

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
    var _this = this
    // 规范 2.2.7，then 必须返回一个新的 promise
    var promise2
    // 根据规范 2.2.1 ，onFulfilled、onRejected 都是可选参数
    // onFulfilled、onRejected不是函数需要忽略，同时也实现了值穿透
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    onRejected = typeof onRejected === 'function' ? onRejected : error => {throw error}
    
    if (_this.currentState === RESOLVED) {
        // 如果promise1（此处为self/this）的状态已经确定并且为resolved，我们调用onFulfilled
        // 如果考虑到有可能throw，所以我们将其包在try/catch块中
        return promise2 = new MyPromise(function(resolve, reject) {
            // 规范 2.2.4，保证 onFulfilled，onRjected 异步执行
      		// 所以用了 setTimeout 包裹下
            setTimeout(function() {
                try {
                	var x = onFulfilled(_this.value)
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
    if (_this.currentState === REJECTED) {
        return promise2 = new MyPromise(function(resolve, reject) {
            setTimeout(function() {
                try {
                	var x = onRejected(_this.value)
                	if (x instanceof Promise){
                    	x.then(resolve, reject)
                	}
            	} catch(err) {
                	reject(err)
            	}
            })
        })
    }
    if (_this.currentState === PENDING) {
        // 如果当前的Promise还处于PENDING状态，我们并不能确定调用onFulfilled还是onRejected
        // 只有等待Promise的状态确定后，再做处理
        // 所以我们需要把我们的两种情况的处理逻辑做成callback放入promise1（此处即self/this）的回调数组内
        // 处理逻辑和以上相似
        return promise2 = new MyPromise(function(resolve, reject) {
            _this.onResolvedCallbacks.push(function() {
                try {
                    var x = onFulfilled(_this.value)
                    if (x instanceof MyPromise) {
                        x.then(resolve, reject)
                    }
                    resolve(x)
                } catch(err) {
                    reject(err)
                }
            })
            _this.onRejectedCallbacks.push(function() {
                try {
                    var x = onRejected(_this.value)
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
// resolve(x) // 否则，以它的返回值为 promise2 的结果
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
const RESOLVED = "resolved"
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
                _this.currentState = RESOLVED // 状态管理
                _this.value = value
                _this.onResolvedCallbacks.forEach(cb => cb())
            }
        })
    } // resolve 处理函数
    _this.reject = function (value) {
        setTimeout(() => { // 异步执行，保证顺序执行
            if (_this.currentState === PENDING) {
                _this.currentState = REJECTED // 状态管理
                _this.value = value
                _this.onRejectedCallbacks.forEach(cb => cb())
            }
        })
    } // reject 处理函数

    // 异常处理
    // new Promise(() => throw Error('error'))
    try {
        callback(_this.resolve, _this.reject) // 执行callback并传入相应的参数
    } catch(e) {
        _this.reject(e)
    }
}
// then 方法接受两个参数，onFulfilled，onRejected，分别为Promise成功或失败的回调
MyPromise.prototype.then = function(onFulfilled, onRejected) {
    var _this = this
    // 规范 2.2.7，then 必须返回一个新的 promise
    var promise2
    // 根据规范 2.2.1 ，onFulfilled、onRejected 都是可选参数
    // onFulfilled、onRejected不是函数需要忽略，同时也实现了值穿透
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    onRejected = typeof onRejected === 'function' ? onRejected : error => {throw error}

    if (_this.currentState === RESOLVED) {
        // 如果promise1（此处为self/this）的状态已经确定并且为resolved，我们调用onFulfilled
        // 如果考虑到有可能throw，所以我们将其包在try/catch块中
        return promise2 = new MyPromise(function(resolve, reject) {
            try {
                var x = onFulfilled(_this.value)
                // 如果 onFulfilled 的返回值是一个 Promise 对象，直接取它的结果作为 promise2 的结果
                resolutionProcedure(promise2, x, resolve, reject)
            } catch (err) {
                reject(err) // 如果出错，以捕获到的错误作为promise2的结果
            }
        })
    }
    // 此处实现与FULFILLED相似，区别在使用的是onRejected而不是onFulfilled
    if (_this.currentState === REJECTED) {
        return promise2 = new MyPromise(function(resolve, reject) {
            try {
                var x = onRejected(_this.value)
                resolutionProcedure(promise2, x, resolve, reject)
            } catch(err) {
                reject(err)
            }
        })
    }
    if (_this.currentState === PENDING) {
        // 如果当前的Promise还处于PENDING状态，我们并不能确定调用onFulfilled还是onRejected
        // 只有等待Promise的状态确定后，再做处理
        // 所以我们需要把我们的两种情况的处理逻辑做成callback放入promise1（此处即_this/this）的回调数组内
        // 处理逻辑和以上相似
        return promise2 = new MyPromise(function(resolve, reject) {
            _this.onResolvedCallbacks.push(function() {
                try {
                    var x = onFulfilled(_this.value)
                    resolutionProcedure(promise2, x, resolve, reject)
                } catch(err) {
                    reject(err)
                }
            })
            _this.onRejectedCallbacks.push(function() {
                try {
                    var x = onRejected(_this.value)
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
// finally 的实现
MyPromise.prototype.finally = function (callback) {
  return this.then(function (value) {
    return MyPromise.resolve(callback()).then(function () {
      return value
    })
  }, function (err) {
    return MyPromise.resolve(callback()).then(function () {
      throw err
    })
  })
}
```
额外，附加 `Promise.race` 与 `Promise.all` 的实现，有兴趣的可以了解一下
```js
// race
MyPromise.race = function(values) {
    return new MyPromise(function(resolve, reject) {
        values.forEach(function(value) {
            MyPromise.resolve(value).then(resolve, reject)
        })
    })
}
// all
MyPromise.all = function(arr) {
    var args = Array.prototype.slice.call(arr)
    return new MyPromise(function (resolve, reject) {
        if (args.length === 0) return resolve([])
        var remaining = args.length
        for (var i = 0; i < args.length; i++) {
            res(i, args[i])
        }
        function res(i, val) {
            if (val && (typeof val === 'object' || typeof val === 'function')) {
                if (val instanceof MyPromise && val.then === MyPromise.prototype.then) {
                    if (val.currentState === RESOLVED) return res(i, val.value)
                    if (val.currentState === REJECTED) reject(val.value)
                    val.then(function (val) {
                        res(i, val)
                    }, reject)
                    return
                } else {
                    var then = val.then
                    if (typeof then === 'function') {
                        var p = new MyPromise(then.bind(val))
                        p.then(function(val) {
                            res(i, val)
                        }, reject)
                        return
                    }
                }
            }
            args[i] = val
            if (--remaining === 0) {
                resolve(args)
            }
        }
    })
}
```
终于实现搞定了，继续加油