一道面试题引发的血案，下面进入主题：

```js
// 今日头条面试题
async function async1() {
    console.log('async1 start')
    await async2()
    console.log('async1 end')
}
async function async2() {
    console.log('async2')
}
console.log('script start')
setTimeout(function () {
    console.log('settimeout')
})
async1()
new Promise(function (resolve) {
    console.log('promise1')
    resolve()
}).then(function () {
    console.log('promise2')
})
console.log('script end')
```

题目的本质，就是考察`setTimeout`、`promise`、`async await`的实现及执行顺序，以及JS的事件循环的相关问题。

答案：

```Js
script start
async1 start
async2
promise1
script end
async1 end
promise2
settimeout
```

这里涉及到`Microtasks`、`Macrotasks`、event loop 以及JS的异步运行机制。

### 一、event loop

JS主线程不断的循环往复的从任务队列中读取任务，执行任务，这中运行机制称为事件循环（event loop）。

### 二、Microtasks、Macrotasks

Microtasks和Macrotasks是异步任务的一种类型，`Microtasks`的优先级要高于`Macrotasks`，下面是它们所包含的api：

- microtasks
  - process.nextTick
  - promise
  - Object.observe (废弃)
  - MutationObserver
- macrotasks
  - setTimeout
  - setImmerdiate
  - setInterval
  - I/O
  - UI 渲染

**注意：**

1. 每一个 event loop 都有一个 microtask queue
2. 每个 event loop 会有一个或多个macrotaks queue ( 也可以称为task queue )
3. 一个任务 task 可以放入 macrotask queue 也可以放入 microtask queue中
4. 每一次event loop，会首先执行 microtask queue， 执行完成后，会提取 macrotask queue 的一个任务加入 microtask queue， 接着继续执行microtask queue，依次执行下去直至所有任务执行结束。

### 三、异步运行机制

我们已知， JS 是单线程的，至于为什么，详见https://blog.csdn.net/lunahaijiao/article/details/85329510。

下面看一个例子：

```js
// 1. 开始执行
console.log(1)	// 	2. 打印 1
setTimeout(function () {	// 6. 浏览器在 0ms 后，将该函数推入任务队列
    console.log(2)	// 7. 打印 2
    Promise.resolve(1).then(function () {	// 8. 将 resolve(1) 推入任务队列  9. 将 function函数推入任务队列
        cosole.log('ok')	// 10. 打印 ok
    })
})	// 3.调用 setTimeout 函数，并定义其完成后执行的回调函数
setTimeout(function (){		// 11. 浏览器 0ms 后，将该函数推入任务队列
    console.log(3)	// 12. 打印 3
})  // 4. 调用 setTimeout 函数，并定义其完成后执行的回调函数
// 5. 主线程执行栈清空，开始读取 任务队列 中的任务
// output： 1  2 ok 3
```

JS 主线程拥有一个 **执行栈（同步任务）** 和 一个 **任务队列（microtasks queue）**，主线程会依次执行代码，

- 当遇到函数（同步）时，会先将函数入栈，函数运行结束后再将该函数出栈；
- 当遇到task任务（异步）时，这些 task 会返回一个值，让主线程不在此阻塞，使主线程继续执行下去，而真正的task任务将交给 **浏览器内核** 执行，浏览器内核执行结束后，会将该任务事先定义好的**回调函数**加入相应的**任务队列（microtasks queue/ macrotasks queue）**中。
- 当JS主线程**清空执行栈**之后，会按先入先出的顺序读取microtasks queue中的**回调函数**，并将该函数入栈，继续运行执行栈，直到清空执行栈，再去读取**任务队列**。
- 当microtasks queue中的任务执行完成后，会提取 macrotask queue 的一个任务加入 microtask queue， 接着继续执行microtask queue，依次执行下去直至所有任务执行结束。

这就是 **JS的异步执行机制**

### 四、async await、Promise、setTimeout

1. setTimeout

   ```js
   console.log('script start')	//1. 打印 script start
   setTimeout(function(){
       console.log('settimeout')	// 4. 打印 settimeout
   })	// 2. 调用 setTimeout 函数，并定义其完成后执行的回调函数
   console.log('script end')	//3. 打印 script start
   // 输出顺序：script start->script end->settimeout
   ```

   

2. Promise

   Promise本身是**同步的立即执行函数**， 当在executor中执行resolve或者reject的时候, 此时是异步操作， 会先执行then/catch等，当主栈完成后，才会去调用resolve/reject中存放的方法执行，打印p的时候，是打印的返回结果，一个Promise实例。

   ```js
   console.log('script start')
   let promise1 = new Promise(function (resolve) {
       console.log('promise1')
       resolve()
       console.log('promise1 end')
   }).then(function () {
       console.log('promise2')
   })
   setTimeout(function(){
       console.log('settimeout')
   })
   console.log('script end')
   // 输出顺序: script start->promise1->promise1 end->script end->promise2->settimeout
   ```

   当JS主线程执行到Promise对象时，

   - promise1.then() 的回调就是一个 task

   - promise1 是 resolved或rejected: 那这个 task 就会放入当前事件循环回合的 microtask queue
   - promise1 是 pending: 这个 task 就会放入 事件循环的未来的某个(可能下一个)回合的 microtask queue 中
   - setTimeout 的回调也是个 task ，它会被放入 macrotask queue 即使是 0ms 的情况

3. async await

   ```js
   async function async1(){
      console.log('async1 start');
       await async2();
       console.log('async1 end')
   }
   async function async2(){
       console.log('async2')
   }
   
   console.log('script start');	// 1.打印 script start
   async1(); 	// 2. 
   console.log('script end')
   
   // 输出顺序：script start->async1 start->async2->script end->async1 end
   ```

   async 函数返回一个 Promise 对象，当函数执行的时候，一旦遇到 await 就会先返回，等到触发的异步操作完成，再执行函数体内后面的语句。可以理解为，是让出了线程，跳出了 async 函数体。

   举个例子：

   ```js
   async function func1() {
       return 1
   }
   
   console.log(func1())
   ```

   ![屏幕快照 2019-01-31 下午5.21.11](/Users/a123/Desktop/Study/JS/img/async.png)

   很显然，func1的运行结果其实就是一个Promise对象。因此我们也可以使用then来处理后续逻辑。

   ```
   func1().then(res => {
       console.log(res);  // 30
   })
   ```

    await的含义为等待，也就是 async 函数需要等待await后的函数执行完成并且有了返回结果（Promise对象）之后，才能继续执行下面的代码。await通过返回一个Promise对象来实现同步的效果。

   

参考： 一道题引发的EventLoop思考https://juejin.im/post/5affd4786fb9a07aa0484602

理解 JavaScript 中的 macrotask 和 microtaskhttps://juejin.im/entry/58d4df3b5c497d0057eb99ff