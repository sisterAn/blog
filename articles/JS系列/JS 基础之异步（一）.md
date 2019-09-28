   - 已知，`JavaScript` 是单线程的，天生异步，适合 IO 密集型，不适合 CPU 密集型，但是，为什么是异步的喃，异步由何而来的喃，我们将在这里逐渐讨论实现。

     

     ### 单线程的 JavaScript

     所谓单线程，是指在 JS 引擎中负责解释和执行 JS 代码的线程唯一，同一时间上只能执行一件任务。

     首先为什么要引入单线程喃？

     **原因：避免 DOM 渲染的冲突**

     - 浏览器需要渲染 DOM 
     - JS 可以修改 DOM 结构
     - JS 执行时，浏览器 DOM 渲染停止
     - 两段 JS 不能同时执行（都修改 DOM 就冲突了）
     - [web worker](http://www.ruanyifeng.com/blog/2018/07/web-worker.html)支持多线程，但是不能访问 DOM

     当然，我们可以为浏览器引入“锁”的机制来解决这些冲突，但大大提高复杂性，所以 JavaScript从诞生开始就选择了单线程执行。

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

     在实例1中，`sum` 并不能立刻打印出来，必须在for循环执行完成之后才能执行 `console.log(sum)` 。在实例2中，浏览器先打印 `1` ，然后弹出弹框，点击确定后才执行 `console.log(2)` 。

     总结：

     - 优点：实现比较简单，执行环境相对单纯
     - 缺点：只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。常见的浏览器无响应（假死），往往就是因为某一段 Javascript 代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。

     为了解决这个问题，JS 语言将任务的执行模式分为两种：同步和异步

     

     ### 同步与异步

     #### 同步

     ```js
     A(args...)
     ```

     如果在函数A返回的时候，调用者就能够得到预期结果(即拿到了预期的返回值或者看到了预期的效果)，那么这个函数就是同步的。

     ```js
     let a = 1
     Math.floor(a)
     console.log(a)
     ```
     #### 异步

     如果在函数A返回的时候，调用者还不能够得到预期结果，而是需要在将来通过一定的手段得到，那么这个函数就是异步的。

     ```js
     fs.readFile('foo.txt', 'utf8', function(err, data) {
         console.log(data);
     });
     ```

     JavaScript 采用异步编程原因有两点，一是JavaScript是单线程，二是为了提高CPU的利用率。

     

     ### 异步过程

     ```js
     fs.readFile('data.json', 'utf8', function(err, data) {
         console.log(data)
     })
     ```

     在执行这段代码时，`fs.readFile`函数返回时，并不会立刻打印`data`，只有`data.json`读取完成时才打印。也就是异步函数`fs.readFile`执行很快，但后面还有工作线程执行异步任务、通知主线程、主线程回调等操作，这个过程就叫做异步过程。

     > 主线程发起一个异步操作，相应的工作线程接受请求并告知主线程已收到（异步函数返回）；只线程继续执行后面的任务，同时工作线程执行异步任务；工作线程完成任务后，通知主线程；主线程收到通知后，执行一定的动作（调用回调函数）。

     工作线程在异步操作完成后通知主线程，那么这个通知机制又是如何显现喃？答案就是就是消息队列与事件循环。

     

     ### 消息队列与事件循环

     > 工作线程将消息放在消息队列，主线程通过事件循环过程去取消息。

     - 消息队列：消息队列是一个先进先出的队列，它里面存放着各种消息。
     - 事件循环：事件循环是指主线程重复从消息队列中取消息、执行的过程。

     #### 事件循环（eventloop）

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

     #### 消息队列

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

     ![异步过程](https://img-blog.csdnimg.cn/2018122817595652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1bmFoYWlqaWFv,size_16,color_FFFFFF,t_70)

     异步过程中的回调函数，一定不在当前这一轮事件循环中执行。

     

     ### 异步与事件

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

     

     ### 生产者与消费者

     从生产者与消费者的角度看，异步过程是这样的：

     > 工作线程是生产者，主线程是消费者(只有一个消费者)。工作线程执行异步任务，执行完成后把对应的回调函数封装成一条消息放到消息队列中；主线程不断地从消息队列中取消息并执行，当消息队列空时主线程阻塞，直到消息队列再次非空。

        那么异步的实现方式有哪些喃？

        - ES6之前：callback、eventloop、Promise
        - ES6：Generator
        - ES7:Async/Await