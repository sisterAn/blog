已知，JavaScript 是单线程的，天生异步，适合 IO 密集型，不适合 CPU 密集型，但是，为什么是异步的喃，异步由何而来的喃，我们将在这里逐渐讨论实现。

### 一、科普：进程与线程

#### 1. 浏览器是多进程的

它主要包括以下进程：

- Browser 进程：浏览器的主进程，唯一，负责创建和销毁其它进程、网络资源的下载与管理、浏览器界面的展示、前进后退等。
- GPU 进程：用于 3D 绘制等，最多一个。
- 第三方插件进程：每种类型的插件对应一个进程，仅当使用该插件时才创建。
- 浏览器渲染进程（浏览器内核）：内部是多线程的，每打开一个新网页就会创建一个进程，主要用于页面渲染，脚本执行，事件处理等。

#### 2. 渲染进程（浏览器内核）

浏览器的渲染进程是多线程的，页面的渲染，JavaScript 的执行，事件的循环，都在这个进程内进行：

- GUI 渲染线程：负责渲染浏览器界面，当界面需要重绘（Repaint）或由于某种操作引发回流(reflow)时，该线程就会执行。
- JavaScript 引擎线程：也称为 JavaScript 内核，负责处理 Javascript 脚本程序、解析 Javascript 脚本、运行代码等。（例如 V8 引擎）
- 事件触发线程：用来控制浏览器事件循环，注意这不归 JavaScript 引擎线程管，当事件被触发时，该线程会把事件添加到待处理队列的队尾，等待 JavaScript 引擎的处理。
- 定时触发器线程：传说中的 `setInterval` 与 `setTimeout` 所在线程，注意，W3C 在 HTML 标准中规定，规定要求 `setTimeout` 中低于 4ms 的时间间隔算为 4ms 。
- 异步 http 请求线程：在 `XMLHttpRequest` 在连接后是通过浏览器新开一个线程请求，将检测到状态变更时，如果设置有回调函数，异步线程就**产生状态变更事件**，将这个回调再放入事件队列中。再由 JavaScript 引擎执行。

注意，**GUI 渲染线程与 JavaScript 引擎线程是互斥的**，当 JavaScript 引擎执行时 GUI 线程会被挂起（相当于被冻结了），GUI 更新会被保存在一个队列中**等到 JavaScript 引擎空闲时**立即被执行。所以如果 JavaScript 执行的时间过长，这样就会造成页面的渲染不连贯，导致页面渲染加载阻塞。

### 二、单线程的 JavaScript

所谓单线程，是指在 JavaScript 引擎中负责解释和执行 JavaScript 代码的线程唯一，同一时间上只能执行一件任务。

**问题：首先为什么要引入单线程喃？**

我们知道：

- 浏览器需要渲染 DOM
- JavaScript 可以修改 DOM 结构
- JavaScript 执行时，浏览器 DOM 渲染停止

如果 JavaScript 引擎线程不是单线程的，那么可以同时执行多段 JavaScript，如果这多段 JavaScript 都修改 DOM，那么就会出现 DOM 冲突。

你可能会说，[web worker](http://www.ruanyifeng.com/blog/2018/07/web-worker.html) 就支持多线程，但是 web worker 不能访问 DOM。

**原因：避免 DOM 渲染的冲突**

当然，我们可以为浏览器引入**锁** 的机制来解决这些冲突，但其大大提高了复杂性，所以 JavaScript从诞生开始就选择了单线程执行。

引入单线程就意味着，所有任务需要排队，前一个任务结束，才会执行后一个任务。这同时又导致了一个问题：如果前一个任务耗时很长，后一个任务就不得不一直等着。

```js
// 实例1
let i, sum = 0
for(i = 0; i < 1000000000; i ++) {
    sum += i
}
console.log(sum)
```

在实例1中，`sum` 并不能立刻打印出来，必须在 for 循环执行完成之后才能执行 `console.log(sum)` 。

```js
// 实例2
console.log(1)
alert('hello')
console.log(2)
```

在实例2中，浏览器先打印 `1` ，然后弹出弹框，点击确定后才执行 `console.log(2)` 。

**总结：**

- 优点：实现比较简单，执行环境相对单纯
- 缺点：只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段 Javascript 代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。

为了解决这个问题，JavaScript 语言将任务的执行模式分为两种：同步和异步

### 三、同步与异步

#### 1. 同步

```js
func(args...)
```

如果在函数 `func` 返回的时候，调用者就能够得到预期结果(即拿到了预期的返回值或者看到了预期的效果)，那么这个函数就是同步的。

```js
let a = 1
Math.floor(a)
console.log(a) // 1
```


#### 2. 异步

如果在函数 `func` 返回的时候，调用者还不能够得到预期结果，而是需要在将来通过一定的手段得到，那么这个函数就是异步的。

```js
fs.readFile('foo.txt', 'utf8', function(err, data) {
    console.log(data);
});
```

**总结：** 

JavaScript 采用异步编程原因有两点，

- 一是 JavaScript 是单线程；
- 二是为了提高 CPU 的利用率。

### 四、异步过程

```js
fs.readFile('data.json', 'utf8', function(err, data) {
    console.log(data)
})
```

在执行这段代码时，`fs.readFile` 函数返回时，并不会立刻打印 `data` ，只有 `data.json` 读取完成时才打印。也就是异步函数 `fs.readFile` 执行很快，但后面还有工作线程执行异步任务、通知主线程、主线程回调等操作，这个过程就叫做异步过程。

> 主线程发起一个异步操作，相应的工作线程接受请求并告知主线程已收到（异步函数返回）；主线程继续执行后面的任务，同时工作线程执行异步任务；工作线程完成任务后，通知主线程；主线程收到通知后，执行一定的动作（调用回调函数）。

工作线程在异步操作完成后通知主线程，那么这个通知机制又是如何显现喃？答案就是就是消息队列与事件循环。

### 五、消息队列与事件循环

> 工作线程将消息放在消息队列，主线程通过事件循环过程去取消息。

- 消息队列：消息队列是一个先进先出的队列，它里面存放着各种消息。
- 事件循环：事件循环是指主线程重复从消息队列中取消息、执行的过程。

#### 事件循环（eventloop）

主线程不断的从消息队列中取消息，执行消息，这个过程称为事件循环，这种机制叫事件循环机制，取一次消息并执行的过程叫一次循环。

大致实现过程如下：

```js
while(true) {
    var message = queue.get()
    execute(message)
}
```

例如：

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
// output：4321 或 4312
```

其中，主线程：

```js
// 主线程
console.log(4)
```

异步队列：

```js
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
```

**事件循环是JavaScript实现异步的具体解决方案，其中同步代码，直接执行；异步函数先放在异步队列中，待同步函数执行完毕后，轮询执行 异步队列 的回调函数。**

#### 消息队列

其中，消息就是注册异步任务时添加的回调函数。

```js
$.ajax('XXX', function(res) {
    console.log(res)
})
...
```

主线程在发起 AJAX 请求后，会继续执行其他代码，AJAX 线程负责请求 `XXX`，拿到请求后，会封装成 JavaScript 对象，然后构造一条消息：

```js
// 消息队列里的消息
var message = function () {
    callback(response)
}
```

其中 `callback` 是 AJAX 网络请求成功响应时的回调函数。

主线程在执行完当前循环中的所有代码后，就会到消息队列取出这条消息(也就是 `message` 函数)，并执行它。到此为止，就完成了工作线程对主线程的 `通知` ，回调函数也就得到了执行。如果一开始主线程就没有提供回调函数，AJAX 线程在收到 HTTP 响应后，也就没必要通知主线程，从而也没必要往消息队列放消息。

![异步过程](https://img-blog.csdnimg.cn/2018122817595652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1bmFoYWlqaWFv,size_16,color_FFFFFF,t_70)

异步过程中的回调函数，一定不在当前这一轮事件循环中执行。

### 六、异步与事件

消息队列中的每条消息实际上都对应着一个事件。

其中一个重要的异步过程就是： **DOM事件** 

```js
var button = document.getElementById('button')
button.addEventLister('click', function(e) {
    console.log('事件')
})
```

从异步的角度看，`addEventLister` 函数就是异步过程的发起函数，事件监听器函数就是异步过程的回调函数。事件触发时，表示异步任务完成，会将事件监听器函数封装成一条消息放在消息队列中，等待主线程执行。

事件的概念实际上并不是必须的，事件机制实际上就是异步过程的通知机制。

另外，所有的异步过程也都可以用事件来描述。例如：

```js
setTimeout(func, 1000)
// 可以看成：
timer.addEventLister('timeout', 1000, func)
```

其中关于事件的详细描述，可以看这篇文章： [事件绑定、事件监听、事件委托](http://blog.xieliqun.com/2016/08/12/event-delegate/)，这里不再深入介绍。

### 七、生产者与消费者

生产者和消费者问题是线程模型中的经典问题：生产者和消费者在同一时间段内共用同一个存储空间，生产者往存储空间中添加数据，消费者从存储空间中取走数据，当存储空间为空时，消费者阻塞，当存储空间满时，生产者阻塞。

![](http://resource.muyiy.cn/image/20200225192721.gif)

从生产者与消费者的角度看，异步过程是这样的：

> 工作线程是生产者，主线程是消费者(只有一个消费者)。工作线程执行异步任务，执行完成后把对应的回调函数封装成一条消息放到消息队列中；主线程不断地从消息队列中取消息并执行，当消息队列空时主线程阻塞，直到消息队列再次非空。

   那么异步的实现方式有哪些喃？

   - ES6之前：callback、eventloop、Promise
   - ES6：Generator
   - ES7:Async/Await
