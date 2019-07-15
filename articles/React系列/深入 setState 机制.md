React 是通过管理状态来实现对组件的管理，即使用 this.state 获取 state，通过 this.setState() 来更新 state，当使用 this.setState() 时，React 会调用 render 方法来重新渲染 UI。

首先看一个例子：

```js
class Example extends React.Component {
  constructor() {
    super();
    this.state = {
      val: 0
    };
  }
  
  componentDidMount() {
    this.setState({val: this.state.val + 1});
    console.log(this.state.val);    // 第 1 次 log

    this.setState({val: this.state.val + 1});
    console.log(this.state.val);    // 第 2 次 log

    setTimeout(() => {
      this.setState({val: this.state.val + 1});
      console.log(this.state.val);  // 第 3 次 log

      this.setState({val: this.state.val + 1});
      console.log(this.state.val);  // 第 4 次 log
    }, 0);
  }

  render() {
    return null;
  }
};
```

答案是： 0	0	2	3，你做对了吗？

### 一、setState 异步更新

setState 通过一个**队列机制**来实现 state 更新，当执行 setState() 时，会将需要更新的 state **浅合并**后放入 状态队列，而不会立即更新 state，队列机制可以高效的**批量更新** state。而如果不通过setState，直接修改this.state 的值，则不会放入状态队列，当下一次调用 setState 对状态队列进行合并时，之前对 this.state 的修改将会被忽略，造成无法预知的错误。

React通过状态队列机制实现了 setState 的异步更新，避免重复的更新 state。

```js
setState(nextState, callback)
```

在 setState 官方文档中介绍：**将 nextState 浅合并到当前 state。这是在事件处理函数和服务器请求回调函数中触发 UI 更新的主要方法。不保证 `setState` 调用会同步执行，考虑到性能问题，可能会对多次调用作批处理。**

举个例子： 

```js
// 假设 state.count === 0
this.setState({count: state.count + 1});
this.setState({count: state.count + 1});
this.setState({count: state.count + 1});
// state.count === 1, 而不是 3
```

本质上等同于：

```js
// 假设 state.count === 0
Object.assign(state,
              {count: state.count + 1},
              {count: state.count + 1},
              {count: state.count + 1}
             )
// {count: 1}
```

但是如何解决这个问题喃，在文档中有提到：

**也可以传递一个签名为 `function(state, props) => newState` 的函数作为参数。这会将一个原子性的更新操作加入更新队列，在设置任何值之前，此操作会查询前一刻的 state 和 props。`...setState()` 并不会立即改变  `this.state` ，而是会创建一个待执行的变动。调用此方法后访问 `this.state` 有可能会得到当前已存在的 state（译注：指 state 尚未来得及改变）。**

即使用 setState() 的第二种形式：以一个函数而不是对象作为参数，此函数的第一个参数是前一刻的state，第二个参数是 state 更新执行瞬间的 props。

```js
// 正确用法
this.setState((prevState, props) => ({
    count: prevState.count + props.increment
}))
```

这种函数式 setState() 工作机制类似：

```js
[
    {increment: 1},
    {increment: 1},
    {increment: 1}
].reduce((prevState, props) => ({
    count: prevState.count + props.increment
}), {count: 0})
// {count: 3}
```

关键点在于**更新函数（updater function）**：

```
(prevState, props) => ({
  count: prevState.count + props.increment
})
```

这基本上就是个 reducer，其中 `prevState` 类似于一个累加器（accumulator），而 `props` 则像是新的数据源。类似于 Redux 中的 reducers，你可以使用任何标准的 reduce 工具库对该函数进行 reduce（包括 `Array.prototype.reduce()`）。同样类似于 Redux，reducer 应该是 [纯函数](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976) 。

> 注意：企图直接修改 `prevState` 通常都是初学者困惑的根源。

相关源码：

```js
// 将新的 state 合并到状态队列
var nextState = this._processPendingState(nextProps, nextContext)

// 根据更新队列和 shouldComponentUpdate 的状态来判断是否需要更新组件
var shouldUpdate = this._pendingForceUpdate ||
    !inst.shouldComponentUpdate ||
    inst.shouldComponentUpdate(nextProps, nextState, nextContext)
```



###二、setState 循环调用风险

当调用 setState 时，实际上是会执行 `enqueueSetState` 方法，并会对 `partialState` 及 `_pendingStateQueue` 队列进行合并操作，最终通过 `enqueueUpdate` 执行 state 更新。

而 `performUpdateIfNecessary` 获取 `_pendingElement`、` _pendingStateQueue`、`_pendingForceUpdate`，并调用 `reaciveComponent` 和 `updateComponent` 来进行组件更新。

**但，如果在 `shouldComponentUpdate` 或 `componentWillUpdate` 方法里调用 this.setState 方法，就会造成崩溃。**这是因为在 `shouldComponentUpdate` 或 `componentWillUpdate` 方法里调用 `this.setState` 时，`this._pendingStateQueue!=null`，则 `performUpdateIfNecessary` 方法就会调用 `updateComponent` 方法进行组件更新，而 `updateComponent` 方法又会调用 `shouldComponentUpdate`和`componentWillUpdate` 方法，因此造成循环调用，使得浏览器内存占满后崩溃。

![循环调用](/Users/a123/Desktop/Study/JS/img/循环调用.png)

**图 2-1 循环调用**

setState 源码：

```js
// 更新 state
ReactComponent.prototype.setState = function(partialState, callback) {
    this.updater.enqueueSetState(this, partialState)
    if (callback) {
        this.updater.enqueueCallback(this, callback, 'setState')
    }
}

enqueueSetState: function(publicInstance, partialState) {
    var internalInstance = getInternalInstanceReadyForUpdate(
        publicInstance,
        'setState'
    )
    if (!internalInstance) {
        return
    }
    
    // 更新队列合并操作
    var queue = internalInstance._pendingStateQueue || (internalInstance._pendingStateQueue=[])
    queue.push(partialState)
    enqueueUpdate(internalInstance)
}

// 如果存在 _pendingElement、_pendingStateQueue、_pendingForceUpdate，则更新组件
performUpdateIfNecessary: function(transaction) {
    if (this._pendingElement != null) {
        ReactReconciler.receiveComponent(this, this._pendingElement, transaction, this._context)
    }
    
    if (this._pendingStateQueue != null || this._pendingForceUpdate) {
        this.updateComponent(transaction, this._currentElement, this._currentElement, this._context, this._context)
    }
}
```



### 三、setState 调用栈

既然 setState 是通过 enqueueUpdate 来执行 state 更新的，那 enqueueUpdate 是如何实现更新 state 的喃？![setState简化调用栈](/Users/a123/Desktop/Study/JS/img/setState简化调用栈.jpg)

**图3-1 setState 简化调用栈**

上面这个流程图是一个简化的 setState 调用栈，注意其中核心的状态判断，在[源码(ReactUpdates.js)](http://link.zhihu.com/?target=https%3A//github.com/facebook/react/blob/35962a00084382b49d1f9e3bd36612925f360e5b/src/renderers/shared/reconciler/ReactUpdates.js%23L199)中

```
function enqueueUpdate(component) {
  // ...

  if (!batchingStrategy.isBatchingUpdates) {
    batchingStrategy.batchedUpdates(enqueueUpdate, component);
    return;
  }

  dirtyComponents.push(component);
}
```

若 isBatchingUpdates 为 false 时，所有队列中更新执行 batchUpdate，否则，把当前组件（即调用了 setState 的组件）放入 dirtyComponents 数组中。先不管这个 batchingStrategy，看到这里大家应该已经大概猜出来了，文章一开始的例子中 4 次 setState 调用表现之所以不同，这里逻辑判断起了关键作用。

那么 *batchingStrategy* 究竟是何方神圣呢？其实它只是一个简单的对象，定义了一个 isBatchingUpdates 的布尔值，和一个 batchedUpdates 方法。下面是一段简化的定义代码：

```
var batchingStrategy = {
  isBatchingUpdates: false,

  batchedUpdates: function(callback, a, b, c, d, e) {
    // ...
    batchingStrategy.isBatchingUpdates = true;
    
    transaction.perform(callback, null, a, b, c, d, e);
  }
};
```

注意 batchingStrategy 中的 **batchedUpdates** 方法中，有一个 transaction.perform 调用。这就引出了本文要介绍的核心概念 —— Transaction（事务）。

### 四、初识事物

在 Transaction 的[源码](http://link.zhihu.com/?target=https%3A//github.com/facebook/react/blob/6d5fe44c8602f666a043a4117ccc3bdb29b86e78/src/shared/utils/Transaction.js)中有一幅特别的 ASCII 图，形象的解释了 Transaction 的作用。

```
/*
 * <pre>
 *                       wrappers (injected at creation time)
 *                                      +        +
 *                                      |        |
 *                    +-----------------|--------|--------------+
 *                    |                 v        |              |
 *                    |      +---------------+   |              |
 *                    |   +--|    wrapper1   |---|----+         |
 *                    |   |  +---------------+   v    |         |
 *                    |   |          +-------------+  |         |
 *                    |   |     +----|   wrapper2  |--------+   |
 *                    |   |     |    +-------------+  |     |   |
 *                    |   |     |                     |     |   |
 *                    |   v     v                     v     v   | wrapper
 *                    | +---+ +---+   +---------+   +---+ +---+ | invariants
 * perform(anyMethod) | |   | |   |   |         |   |   | |   | | maintained
 * +----------------->|-|---|-|---|-->|anyMethod|---|---|-|---|-|-------->
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | +---+ +---+   +---------+   +---+ +---+ |
 *                    |  initialize                    close    |
 *                    +-----------------------------------------+
 * </pre>
 */
```

简单地说，一个所谓的 Transaction 就是将需要执行的 method 使用 wrapper 封装起来，再通过 Transaction 提供的 perform 方法执行。而在 perform 之前，先执行所有 wrapper 中的 initialize 方法；perform 完成之后（即 method 执行后）再执行所有的 close 方法。一组 initialize 及 close 方法称为一个 wrapper，从上面的示例图中可以看出 Transaction 支持多个 wrapper 叠加。

具体到实现上，React 中的 Transaction 提供了一个 Mixin 方便其它模块实现自己需要的事务。而要使用 Transaction 的模块，除了需要把 Transaction 的 Mixin 混入自己的事务实现中外，还需要额外实现一个抽象的 getTransactionWrappers 接口。这个接口是 Transaction 用来获取所有需要封装的前置方法（initialize）和收尾方法（close）的，因此它需要返回一个数组的对象，每个对象分别有 key 为 initialize 和 close 的方法。

下面是一个简单使用 Transaction 的例子

```
var Transaction = require('./Transaction');

// 我们自己定义的 Transaction
var MyTransaction = function() {
  // do sth.
};

Object.assign(MyTransaction.prototype, Transaction.Mixin, {
  getTransactionWrappers: function() {
    return [{
      initialize: function() {
        console.log('before method perform');
      },
      close: function() {
        console.log('after method perform');
      }
    }];
  };
});

var transaction = new MyTransaction();
var testMethod = function() {
  console.log('test');
}
transaction.perform(testMethod);

// before method perform
// test
// after method perform
```

当然在实际代码中 React 还做了异常处理等工作，这里不详细展开。有兴趣的同学可以参考源码中 [Transaction](http://link.zhihu.com/?target=https%3A//github.com/facebook/react/blob/401e6f10587b09d4e725763984957cf309dfdc30/src/shared/utils/Transaction.js) 实现。

说了这么多 Transaction，它到底是怎么导致上文所述 setState 的各种不同表现的呢？

### 五、解密 setState

那么 Transaction 跟 setState 的不同表现有什么关系呢？首先我们把 4 次 setState 简单归类，前两次属于一类，因为他们在同一次调用栈中执行；setTimeout 中的两次 setState 属于另一类，原因同上。让我们分别看看这两类 setState 的调用栈：

![img](/Users/a123/Desktop/Study/JS/img/componentDidMount里的setState调用栈.jpg)

**图 5-1 componentDidMount 里的 setState 调用栈**

![img](/Users/a123/Desktop/Study/JS/img/setTimeout里的setState调用栈.jpg)

**图 5-2 setTimeout 里的 setState 调用栈**

很明显，在 componentDidMount 中直接调用的两次 setState，其调用栈更加复杂；而 setTimeout 中调用的两次 setState，调用栈则简单很多。让我们重点看看第一类 setState 的调用栈，有没有发现什么熟悉的身影？没错，就是**batchedUpdates** 方法，原来早在 setState 调用前，已经处于 batchedUpdates 执行的 transaction 中！

那这次 batchedUpdate 方法，又是谁调用的呢？让我们往前再追溯一层，原来是 ReactMount.js 中的**_renderNewRootComponent** 方法。也就是说，整个将 React 组件渲染到 DOM 中的过程就处于一个大的 Transaction 中。

### 六、回到题目

接下来的解释就顺理成章了，因为在 componentDidMount 中调用 setState 时，batchingStrategy 的 isBatchingUpdates 已经被设为 true，所以两次 setState 的结果并没有立即生效，而是被放进了 dirtyComponents 中。这也解释了两次打印this.state.val 都是 0 的原因，新的 state 还没有被应用到组件中。

再反观 setTimeout 中的两次 setState，因为没有前置的 batchedUpdate 调用，所以 batchingStrategy 的 isBatchingUpdates 标志位是 false，也就导致了新的 state 马上生效，没有走到 dirtyComponents 分支。也就是，**setTimeout 中第一次 setState 时，this.state.val 为 1，而 setState 完成后打印时 this.state.val 变成了 2。第二次 setState 同理**。

在上文介绍 Transaction 时也提到了其在 React 源码中的多处应用，想必调试过 React 源码的同学应该能经常见到它的身影，像 initialize、perform、close、closeAll、notifyAll 等方法出现在调用栈里时，都说明当前处于一个 Transaction 中。

**既然事务那么有用，那我们可以用它吗？**

答案是**不能**，但在 React 15.0 之前的版本中还是为开发者提供了 batchedUpdates 方法，它可以解决针对一开始例子中 setTimeout 里的两次 setState 导致 rendor 的情况：

```js
import ReactDom, { unstable_batchedUpdates } from 'react-dom';

unstable_batchedUpdates(() => {
  this.setState(val: this.state.val + 1);
  this.setState(val: this.state.val + 1);
});
```

在 React 15.0 之后的版本已经将 batchedUpdates 彻底移除了，所以，不再建议使用。

