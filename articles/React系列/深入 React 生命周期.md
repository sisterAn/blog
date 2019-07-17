#### 你真的了解 React 生命周期吗？

React 生命周期很多人都了解，但通常我们所了解的都是 **单个组件** 的生命周期，但针对 **多个组件（父子组件，兄弟组件）** 的生命周期又是怎么样的喃？你有思考和了解过吗，接下来我们将完整的了解 React 生命周期，如果你已经了解了单个组件的生命周期，可以选择跳过第二部分。

关于 **组件** ，我们这里指的是 `React.omponent` 以及 `React.PureComponent` ，但 是否包括 Hooks 组件喃？

#### 一、Hooks 组件

Hooks 是一个特殊的函数。在 Hooks 之前，函数组件是没有 state 的概念的，因此也就不存在生命周期一说，就仅仅是一个 render 函数。

引入 Hooks 之后，它让函数组件也可以拥有 state，相应的也就有了生命周期的概念，所谓的生命周期也就是 `useEffect()`和 `useLayoutEffect()`具体何时执行的问题。

但，函数组件的本质是函数，而函数本身是没有生命周期的，因此 Hooks 的出现也无法改变这一点。所以这里我们探讨的准确的来说的，是 使用了`Hooks 的函数组件`。

即：**Hooks 组件（使用了Hooks的函数组件）有生命周期，而函数组件（未使用Hooks的函数组件）是没有生命周期的**

#### 二、单个组件的生命周期

**React v16.3之前：**

我们可以将生命周期分为三个阶段：

- 挂载阶段
- 组件更新阶段
- 卸载阶段

分开来讲：

1. 挂载阶段
   - `constructor`
   - `componentWillMount`
   - `render`：react 最重要的步骤，创建虚拟 dom，进行 diff 算法，更新 dom 树都在此进行
   - `componentDidMount`
2. 组件更新阶段
   - `componentWillReceiveProps`
   - `shouldComponentUpdate`
   - `componentWillUpdate`
   - `render`
   - `componentDidUpdate`
3. 卸载阶段
   - `componentWillUnMount`

![1551681166170-c5db89e3-0cc4-4bdd-9698-a993b849fa91](https://user-images.githubusercontent.com/19721451/53748451-54ee2380-3ee0-11e9-9713-5cd6da8dbd47.png)

但是，当更新复杂组件的最上层组件时，调用栈会很长，如果再进行复杂的操作，就可能长时间阻塞主线程，带来不好的用户体验，Fiber 就是为了解决该问题而生。

**V16.3之后**

Fiber 本质上是一个虚拟的堆栈帧，新的调度器会按照优先级自由调度这些帧，从而将之前的同步渲染改成了异步渲染，在不影响体验的情况下去分段计算更新。

对于异步渲染，分为两阶段：

- `reconciliation`：
  - `componentWillMount`
  - `componentWillReceiveProps`
  - `shouldConmponentUpdate`
  - `componentWillUpdate`
- `commit`
  - `componentDidMount`
  - `componentDidUpdate`

其中，`reconciliation`阶段是可以被打断的，所以`reconcilation`阶段执行的函数就会出现多次调用的情况，显然，这是不合理的。

所以V16.3引入了新的API来解决这个问题：

1. `static getDerivedStateFromProps`： 该函数会在挂载阶段和组件更新阶段都会执行被调用，即每次获取新的`props`或`state`之后都会被执行，在挂载阶段用来代替`componentWillMount`；在组件更新阶段配合`componentDidUpdate`，可以覆盖`componentWillReceiveProps`的所有用法。同时它是一个静态函数，所以函数体内不能访问`this`，会根据`nextProps`和`prevState`计算出预期的状态改变，返回结果会被送给`setState`，返回`null`则说明不需要更新`state`，并且这个返回是必须的。
2. `getSnapshotBeforeUpdate`: 该函数会在 `render` 之后， DOM 更新前被调用，用于读取最新的 DOM 数据，返回一个值，作为`componentDidUpdate`的第三个参数；配合`componentDidUpdate`, 可以覆盖`componentWillUpdate`的所有用法。

**注意：V16.3中只用在组件挂载或组件`props`更新过程才会调用，即如果是因为自身setState引发或者forceUpdate引发，而不是不由父组件引发，那么`static getDerivedStateFromProps`也不会被调用，在 V16.4中更正为都调用。**

即更新后的生命周期为：

1. 挂载阶段
   - `constructor`
   - `static getDerivedStateFromProps`
   - `render`
   - `componentDidMount`
2. 更新阶段
   - `static getDerivedStateFromProps`
   - `shouldComponentUpdate`
   - `render`
   - `getSnapshotBeforeUpdate`
   - `componentDidUpdate`
3. 卸载阶段
   - `componentWillUnmount`

![1551680577970-8dd9ac01-2e01-4025-8110-b857f246c694](https://user-images.githubusercontent.com/19721451/53748559-849d2b80-3ee0-11e9-9e57-42f090d88c05.png)

#### 三、多个组件的生命周期

##### 1. 父子组件

- **挂载阶段**

  分 **两个** 阶段：

  - 第 **一** 阶段，由父组件开始执行到自身的 `render`，解析其下有哪些子组件需要渲染，并对其中 **同步的子组件** 进行创建，按 **递归顺序** 挨个执行各个子组件至 `render`，生成到父子组件对应的 Virtual DOM 树，并 commit 到 DOM。
  - 第 **二** 阶段，此时 DOM 节点已经生成完毕，组件挂载完成，开始后续流程。先依次触发同步子组件各自的 `componentDidMount`，最后触发父组件的。

  **注意**：如果父组件中包含异步子组件，则会在父组件挂载完成后被创建。

  所以**执行顺序**是：

  **父组件componentWillMount —> 同步子组件 componentWillMount —> 同步子组件 componentDidMount —> 父组件 componentDidMount —> 异步子组件 componnetWillMount —> 异步子组件 componentDidMount**

- **更新阶段**

  更新阶段，均是同步组件

  **React 的设计遵循单向数据流模型** ，也就是说，数据均是由父组件流向子组件。

  - 第 **一** 阶段，由父组件开始，执行

    1. `static getDerivedStateFromProps`
    2. `shouldComponentUpdate`

    更新到自身的 `render`，解析其下有哪些子组件需要渲染，并对 **子组件** 进行创建，按 **递归顺序** 挨个执行各个子组件至 `render`，生成到父子组件对应的 Virtual DOM 树，并与已有的 Virtual DOM 树 比较，计算出 **Virtual DOM 真正变化的部分** ，并只针对该部分进行的原生DOM操作。

  - 第 **二** 阶段，此时 DOM 节点已经生成完毕，组件挂载完成，开始后续流程。先依次触发同步子组件以下函数，最后触发父组件的。

    1. `getSnapshotBeforeUpdate()`
    2. `componentDidUpdate()`

    React 会按照上面的顺序依次执行这些函数，每个函数都是各个子组件的先执行，然后才是父组件的执行。

    所以**执行顺序**是：

    **父组件 getDerivedStateFromProps —> 父组件 shouldComponentUpdate —> 子组件 getDerivedStateFromProps —> 子组件 shouldComponentUpdate —> 子组件 getSnapshotBeforeUpdate —>  父组件 getSnapshotBeforeUpdate —> 子组件 componentDidUpdate —> 父组件 componentDidUpdate**

- **卸载阶段**

  `componentWillUnmount()`，顺序为 **父组件的先执行，子组件按照在 JSX 中定义的顺序依次执行各自的方法**。

  **注意** ：如果卸载旧组件的同时伴随有新组件的创建，新组件会先被创建并执行完 `render`，然后卸载不需要的旧组件，最后新组件执行挂载完成的回调。

##### 2. 兄弟组件

- **挂载阶段**

  若是同步路由，它们的创建顺序和其在共同父组件中定义的先后顺序是 **一致** 的。

  若是异步路由，它们的创建顺序和 js 加载完成的顺序一致。

- **更新阶段、卸载阶段**

  兄弟节点之间的通信主要是经过父组件（Redux 和 Context 也是通过改变父组件传递下来的 `props` 实现的），**满足React 的设计遵循单向数据流模型**， **因此任何两个组件之间的通信，本质上都可以归结为父子组件更新的情况** 。

  所以，兄弟组件更新、卸载阶段，请参考 **父子组件**。

##### 3. Hooks 组件

- **挂载阶段**

  Hooks 组件和 React.Componet 在挂载阶段唯一的不同是，它包含第三个阶段：

  第三阶段，如果组件使用了 `useEffect`，则会在第二阶段之后触发 `useEffect`。如果父子组件都使用了 `useEffect`，那么子组件先触发，然后是父组件。

- **更新阶段**

  Hooks 组件和 React.Componet 在更新阶段唯一的不同是，它在第二个阶段执行的函数不同：

  1. `useLayoutEffect() 的 Cleanup`
  2. `useLayoutEffect()`

- **卸载阶段**

  卸载过程涉及到 `useEffect()`的 Cleanup、`useLayoutEffect()`的 Cleanup 这两种函数，顺序固定为父组件的先执行，子组件按照在 JSX 中定义的顺序依次执行各自的方法。

  注意，此时的 Cleanup 函数会按照在代码中定义的顺序先后执行，与函数本身的特性无关。

##### 4. 总结

- 同步路由，父组件在 `render` 阶段创建子组件。
- 异步路由，父组件在挂载之后才开始创建子组件。
- 挂载完成之后，在更新时，同步组件和异步组件是一样的。
- 无论是挂载还是更新，以 `render`完成为界，之前父组件先执行，之后子组件先执行。
- 兄弟组件大体上按照在父组件中的出场顺序执行。
- `useEffect`会在挂载/更新完成之后，延迟执行。
- 异步请求（如访问 API）何时得到响应与组件的生命周期无关，即父组件中发起的异步请求不保证在子组件挂载完成前得到响应。

 

 

 

 

