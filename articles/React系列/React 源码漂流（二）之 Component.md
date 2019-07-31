### 一、组件

#### 1. 纯组件

`React.PureComponent` ，和 `React.Component` 类似，都是定义一个组件类。不同是 `React.Component` 没有实现 `shouldComponentUpdate()`，而 `React.PureComponent` 通过 `props` 和 `state` 的**浅比较**实现了。

```js
// React.PureComponent 纯组件
class Counter extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 0};
  }
  render() {
    return (
      <button onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

在下一节中将会详细介绍。



#### 2. 函数组件

定义React组件的**最简单**方式就是定义一个函数组件，它接受单一的 props 并返回一个React元素。

```js
// 函数组件
function Counter(props) {
    return <div>Counter: {props.count}</div>
}
// 类组件
class Counter extends React.Component {
  render() {
    return <div>Counter: {this.props.count}</div>
  }
}
```

- 在 函数组件 中，它的输入输出全部由 props 决定，且不会产生任何副作用，这说明 **函数组件 也是 无状态组件**。
- 在函数组件中，无法修改 props，无法使用 state 及组件的生命周期，说明 **函数组件 也是 展示组件**。
- 函数组件 的功能只是接收 props，渲染页面，它不执行与 UI 无关的逻辑处理，它只是一个**纯函数**。
- 函数组件，相对于类组件来说，更加简洁。无论是复用性还是性能，都**优于类组件**。



#### 3. 受控组件与非受控组件

**受控和非受控主要是取决于组件是否受父级传入的 props 控制**

用 props 传入数据的话，组件可以被认为是**受控**（因为组件被父级传入的 props 控制）。

数据只保存在组件内部的 state 的话，是**非受控**组件（因为外部没办法直接控制 state）。

```js
export default class AnForm extends React.Component {
  state = {
    name: ""
  }
  handleSubmitClick = () => {
    console.log("非受控组件: ", this._name.value);
    console.log("受控组件: ", this.state.name);
  }
  handleChange = (e) => {
    this.setState({
      name: e.target.value
    })
  }

  render() {
    return (
      <form onSubmit={this.handleSubmitClick}>
      <label>
        非受控组件:
        <input 
        	type="text" 
        	defaultValue="default" 
        	ref={input => this._name = input} 
        />
      </label>
      <label>
        受控组件:
        <input 
        	type="text" 
        	value={this.state.name} 
        	onChange={this.handleChange}
        />
      </label>
      <input type="submit" value="Submit" />
    </form>
    );
  }
}
```

##### 受控组件

与 html 不同的是，在 React 中，`<input>`或`<select>`、`<textarea> `等这类组件，不会主动维持自身状态，并根据用户输入进行更新。它们都要绑定一个`onChange`事件；每当状态发生变化时，都要写入组件的 state 中，在 React 中被称为**受控组件**。

```js
export default class AnForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ""};
    this.handleChange = this.handleChange.bind(this);
  }
  handleChange(event) {
    this.setState({value: event.target.value});
  }
  render() {
    return <input 
      			type="text" 
      			value={this.state.value} 
      			onChange={this.handleChange} 
      		/>;
  }
}
```

- **onChange & value 模式**（单选按钮和复选按钮对应的是 checked props）

- react通过这种方式**消除了组件的局部状态，**使得应用的整个**状态可控**。

- 注意 `<input type="file" />`，它是一个**非受控组件**。

- 可以使用计算属性名将多个相似的操作组合成一个。

  ```js
  this.setState({
    [name]: value
  });
  ```

##### 非受控组件

非受控组件不再将数据保存在 state，而使用 refs，将真实数据保存在 DOM 中。

```js
export default class AnForm extends Component {
  handleSubmitClick = () => {
    const name = this._name.value;
  }

  render() {
    return (
      <div>
        <input type="text" ref={input => this._name = input} />
        <button onClick={this.handleSubmitClick}>Sign up</button>
      </div>
    );
  }
}
```

- **非受控组件是最简单快速**的实现方式，项目中出现极简的表单时，使用它，但**受控组件才是是最权威的**。
- 通常指定一个 **defaultValue/defaultChecked** 默认值来控制初始状态，不使用 value。
- 非受控组件相比于受控组件，更容易同时集成 React 和非 React 代码。

- 使用场景

  | 特征                                                         | 非受控组件 | 受控组件 |
  | ------------------------------------------------------------ | ---------- | -------- |
  | one-time value retrieval (e.g. on submit)                    | ✅          | ✅        |
  | [validating on submit](https://goshakkk.name/submit-time-validation-react/) | ✅          | ✅        |
  | [instant field validation](https://goshakkk.name/instant-form-fields-validation-react/) | ❌          | ✅        |
  | [conditionally disabling submit button](https://goshakkk.name/form-recipe-disable-submit-button-react/) | ❌          | ✅        |
  | enforcing input format                                       | ❌          | ✅        |
  | several inputs for one piece of data                         | ❌          | ✅        |
  | [dynamic inputs](https://goshakkk.name/array-form-inputs/)   | ❌          | ✅        |

#### 4. 有状态组件与无状态组件

##### 有状态组件

通过 state 管理状态

```js
export default class Counter extends React.Component {
  constructor(props) {
    super(props)
    this.state = { clicks: 0 }
    this.handleClick = this.handleClick.bind(this)
  }
  handleClick() {
    this.setState(state => ({ clicks: state.clicks + 1 }))
  }
  render() {
    return (
      <Button
        onClick={this.handleClick}
        text={`You've clicked me ${this.state.clicks} times!`}
      />
    )
  }
}
```
##### 无状态组件

输入输出数据完全由props决定，而且不会产生任何副作用。

```js
const Button = props =>
  <button onClick={props.onClick}>
    {props.text}
  </button>
```

- 无状态组件一般会搭配高阶组件（简称：HOC）一起使用，高阶组件用来托管state，Redux 框架就是通过 store 管理数据源和所有状态，其中所有负责展示的组件都使用无状态函数式的写法。
- 一个简单的 无状态(stateless) 按钮组件，仅依赖于 props(属性) ，这也称为**函数式组件**。



#### 5. 展示组件与容器组件

##### 展示组件

展示组件指不关心数据是怎么加载和变动的，只关注于页面展示效果的组件。

```js
class TodoList extends React.Component{
    constructor(props){
        super(props);
    }
    render(){
        const {todos} = this.props;
        return (<div>
                <ul>
                    {todos.map((item,index)=>{
                        return <li key={item.id}>{item.name}</li>
                    })}
                </ul>
            </div>)
    }
}
```

- 只能通过 **props** 的方式**接收数据和进行回调**(callback)操作。
- **很少拥有自己的状态**，即使有也是用于展示UI状态的。
- 通常允许通过 **this.props.children** 方式来包含其他组件。
- **内部可以包含展示组件和容器组件**，通常会包含一些自己的DOM标记和样式(style)
- 对应用程序的其他部分没有依赖关系，例如Flux操作或store。
- 会被写成函数式组件除非该组件需要自己的状态，生命周期或者做一些性能优化。

##### 容器组件

容器组件只关心数据是怎么加载和变动的，而不关注于页面展示效果。

```js
//容器组件
class TodoListContainer extends React.Component{
    constructor(props){
        super(props);
        this.state = {
            todos:[]
        }
        this.fetchData = this.fetchData.bind(this);
    }
    componentDidMount(){
        this.fetchData();
    }
    fetchData(){
        fetch('/api/todos').then(data =>{
            this.setState({
                todos:data
            })
        })
    }
    render(){
        return (<div>
                <TodoList todos={this.state.todos} />    
            </div>)
    }
}
```

- **内部可以包含容器组件和展示组件**，但通常没有任何自己的DOM标记，除了一些包装divs，并且从不具有任何样式。
- 提供数据和行为给其他的展示组件或容器组件。
- 可以调用 Flux 操作并将它们作为回调函数（callback）提供给展示组件。
- 往往是**有状态**的，因为它们倾向于**作为数据源**
- 通常使用**高阶组件**生成，例如React Redux的connect()



#### 6. 高阶组件

**高阶函数**的定义：接收函数作为输入，或者输出另一个函数的一类函数，被称作高阶函数。

对于**高阶组件**，它描述的便是接受 React 组件作为输入，输出一个新的 React 组件的组件。

更通俗的描述为，高阶组件通过包裹（wrapped）被传入的 React 组件，经过一系列处理，最终返回一个**相对增强（enhanced）的 React 组件**，供其他组件调用。使我们的代码更具有复用性、逻辑性和抽象特性，它可以对 render 方法做劫持，也**可以控制  props 、state**。

实现高阶组件的方法有以下两种：

- **属性代理（props proxy）**，高阶组件通过被包裹的 React 组件来操作 props。
- **反向继承（inheritance inversion）**，高阶组件继承于被包裹的 React 组件。

```js
// 属性代理
export default function withHeader(WrappedComponent) {
  return class HOC extends React.Component { // 继承与 React.component
    render() {
      const newProps = {
        test:'hoc'
      }
      // 透传props，并且传递新的newProps
      return <div>
        <WrappedComponent {...this.props} {...newProps}/> 
      </div>
    }
  }
}

// 反向继承
export default function (WrappedComponent) {
  return class Inheritance extends WrappedComponent { // 继承于被包裹的 React 组件
    componentDidMount() {
      // 可以方便地得到state，做一些更深入的修改。
      console.log(this.state);
    }
    render() {
      return super.render();
    }
  }
}
```

- 注意：不要在 HOC 内修改一个组件的原型（或以其它方式修改组件）
- 贯穿传递不相关props属性给被包裹的组件，帮助确保高阶组件最大程度的灵活性和可重用性
- 应该使用**最大化的组合性**
- 为了便于调试，可以选择一个显示名字，传达它是一个高阶组件的结果，`WrappedComponent.displayName || WrappedComponent.name || 'Component';`
- **不要在 render() 方法中创建 HOC**，否则，每一次渲染，都会重新创建渲染 HOC
- 必须将原始组件的静态方法在 HOC 中做拷贝，否则 HOC 将没有原始组件的任何静态方法
- Refs 属性不能贯穿传递，我们可以使用 React.forwardRef 解决



#### 7. Hook 组件

Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。

但与 class 生命周期不同的是，Hook 更接近于实现状态同步，而不是响应生命周期事件。

```js
import React, { useState, useEffect } from 'react';

function Example() {
  // 声明一个叫 "count" 的 state 变量
  const [count, setCount] = useState(0);
    
  useEffect(()=>{
    // 需要在 componentDidMount 执行的内容
    return function cleanup() {
      // 需要在 componentWillUnmount 执行的内容      
  	}
  }, [])

  useEffect(() => { 
    // 在 componentDidMount，以及 count 更改时 componentDidUpdate 执行的内容
    document.title = 'You clicked ' + count + ' times'; 
    return () => {
      // 需要在 count 更改时 componentDidUpdate（先于 document.title = ... 执行，遵守先清理后更新）
      // 以及 componentWillUnmount 执行的内容       
    } // 当函数中 Cleanup 函数会按照在代码中定义的顺序先后执行，与函数本身的特性无关
  }, [count]); // 仅在 count 更改时更新

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

- Hooks 组件更接近于实现状态同步，而不是响应生命周期事件
- 只能在**函数最外层**调用 Hook。只能在 **React 的函数组件**中调用 Hook。
- `useLayoutEffect` 与 `componentDidMount`、`componentDidUpdate` 的调用阶段是一样的。但是，我们推荐你**一开始先用 useEffect**，只有当它出问题的时候再尝试使用 `useLayoutEffect`
- 与 `componentDidMount` 或 `componentDidUpdate` 不同的是，Hook 在浏览器完成布局与绘制**之后**，传给 `useEffect` 的函数会延迟调用，但会保证在任何新的渲染前执行
- effect 的清除（cleanup）并不会读取“最新”的 props 。它只能读取到定义它的那次渲染中的 props 值
- effect 中可以读取到最新的 count 状态值，并不是 count 的值在“不变”的effect中发生了改变，而是effect 函数本身在每一次渲染中都不相同
- 在 class 组件生命周期的思维模型中，副作用的行为和渲染输出是不同的。UI渲染是被 props 和 state 驱动的，并且能确保步调一致，但副作用并不是这样。这是一类常见问题的来源。
- 而在 `useEffect` 的思维模型中，默认都是同步的。副作用变成了 React 数据流的一部分。对于每一个 `useEffect` 调用，一旦你处理正确，你的组件能够更好地处理边缘情况。



### 二、Component 源码解读

首先看一下 React.Component 结构

```js
// ReactBaseClasses.js 文件
/**
 * Base class helpers for the updating state of a component.
 */
function Component(props, context, updater) {
  this.props = props; // 属性 props
  this.context = context; // 上下文 context
  // If a component has string refs, we will assign a different object later.
  // 初始化 refs，为 {}，主要在 stringRef 中使用，将 stringRef 节点的实例挂载在 this.refs 上
  this.refs = emptyObject; 
  // We initialize the default updater but the real one gets injected by the
  // renderer.
  this.updater = updater || ReactNoopUpdateQueue; // updater
}

Component.prototype.isReactComponent = {};

/**
 * 设置 state 的子集，使用该方法更新 state，避免 state 的值为可突变的状态
 * `shouldComponentUpdate`只是浅比较更新，
 * 可突变的类型可能导致 `shouldComponentUpdate` 返回 false，无法重新渲染
 * Immutable.js 可以解决这个问题。它通过结构共享提供不可突变的，持久的集合：
 * 不可突变: 一旦创建，集合就不能在另一个时间点改变。
 * 持久性: 可以使用原始集合和一个突变来创建新的集合。原始集合在新集合创建后仍然可用。
 * 结构共享: 新集合尽可能多的使用原始集合的结构来创建，以便将复制操作降至最少从而提升性能。
 *
 * 并不能保证 `this.state` 通过 `setState` 后不可突变的更新，它可能还返回原来的数值
 * 不能保证 `setrState` 会同步更新 `this.state`
 * `setState` 是通过队列形式来更新 state ，当 执行 `setState` 时，
 * 会把 state 浅合并后放入状态队列，然后批量执行，即它不是立即更新的。
 * 不过，你可以在 callback 回调函数中获取最新的值
 * 
 * 注意：对于异步渲染，我们应在 `getSnapshotBeforeUpdate` 中读取 `state`、`props`,
 * 而不是 `componentWillUpdate`
 *
 * @param {object|function} partialState Next partial state or function to
 *        produce next partial state to be merged with current state.
 * @param {?function} callback Called after state is updated.
 * @final
 * @protected
 */
Component.prototype.setState = function(partialState, callback) {
  // 当 partialState 状态为 object 或 function类型 或 null 时，
  // 执行 this.updater.enqueueSetState 方法，否则报错
  invariant(
    typeof partialState === 'object' ||
      typeof partialState === 'function' ||
      partialState == null,
    'setState(...): takes an object of state variables to update or a ' +
      'function which returns an object of state variables.',
  );
  // 将 `setState` 事务放入队列中
  this.updater.enqueueSetState(this, partialState, callback, 'setState'); 
};

/**
 * 强制更新，当且仅当当前不处于 DOM 事物（transaction）中才会被唤起
 * This should only be invoked when it is known with
 * certainty that we are **not** in a DOM transaction.
 * 
 * 默认情况下，当组件的state或props改变时，组件将重新渲染。
 * 如果你的`render()`方法依赖于一些其他的数据，
 * 你可以告诉React组件需要通过调用`forceUpdate()`重新渲染。 
 * 调用`forceUpdate()`会导致组件跳过 `shouldComponentUpdate()`,
 * 直接调用 `render()`。但会调用 `componentWillUpdate` 和 `componentDidUpdate`。
 * 这将触发组件的正常生命周期方法,包括每个子组件的 shouldComponentUpdate() 方法。 
 * forceUpdate 就是重新 render 。
 * 有些变量不在 state 上，当时你又想达到这个变量更新的时候，刷新 render ；
 * 或者 state 里的某个变量层次太深，更新的时候没有自动触发 render 。
 * 这些时候都可以手动调用 forceUpdate 自动触发 render
 * 
 * @param {?function} callback 更新完成后的回调函数.
 * @final
 * @protected
 */
Component.prototype.forceUpdate = function(callback) {
  // updater 强制更新
  this.updater.enqueueForceUpdate(this, callback, 'forceUpdate'); 
};
```

其中 `this.refs`  值 `emptyObject` 为：

```js
// 设置 refs 初始值为 {}
const emptyObject = {};
if (__DEV__) {
  Object.freeze(emptyObject); // __DEV__ 模式下， 冻结 emptyObject
}
// Object.freeze() 冻结一个对象，被冻结的对象不能被修改（添加，删除，
// 修改已有属性的可枚举性、可配置性、可写性与属性值，原型）；返回和传入的参数相同的对象。

```

`ReactNoopUpdateQueue ` 为：

```js
// ReactNoopUpdateQueue.js 文件
/**
 * 这是一个关于 更新队列(update queue) 的抽象 API
 */
const ReactNoopUpdateQueue = {
  /**
   * 检查复合组件是否装载完成（被插入树中）
   * @param {ReactClass} publicInstance 测试实例单元
   * @return {boolean} 装载完成为 true，否则为 false
   * @protected
   * @final
   */
  isMounted: function(publicInstance) {
    return false;
  },

  /**
   * 强制更新队列，当且仅当当前不处于 DOM 事物（transaction）中才会被唤起
   *
   * 当 state 里的某个变量层次太深，更新的时候没有自动触发 render 。
   * 这些时候就可以调用该方法强制更新队列
   *
   * 该方法将跳过 `shouldComponentUpdate()`, 直接调用 `render()`, 但它会唤起
   * `componentWillUpdate` 和 `componentDidUpdate`.
   *
   * @param {ReactClass} publicInstance 将被重新渲染的实例
   * @param {?function} callback 组件更新后的回调函数.
   * @param {?string} callerName 在公共 API 调用该方法的函数名称.
   * @internal
   */
  enqueueForceUpdate: function(publicInstance, callback, callerName) {
    warnNoop(publicInstance, 'forceUpdate');
  },

  /**
   * 完全替换state，与 `setState` 不同的是，`setState` 是以修改和新增的方式改变 `state `的，
   * 不会改变没有涉及到的 `state`。
   * 而 `enqueueReplaceState` 则用新的 `state` 完全替换掉老 `state`
   * 使用它或 `setState` 来改变 state，并且应该把 this.state 设置为不可突变类型对象，
   * 并且this.state不会立即更改
   * 我们应该在回调函数 callback 中获取最新的 state
   *
   * @param {ReactClass} publicInstance The instance that should rerender.
   * @param {object} completeState Next state.
   * @param {?function} callback Called after component is updated.
   * @param {?string} callerName name of the calling function in the public API.
   * @internal
   */
  enqueueReplaceState: function(
    publicInstance,
    completeState,
    callback,
    callerName,
  ) {
    warnNoop(publicInstance, 'replaceState');
  },

  /**
   * 设置 state 的子集
   * 它存在的唯一理由是 _pendingState 是内部方法。
   * `enqueueSetState` 实现浅合并更新 `state`
   *
   * @param {ReactClass} publicInstance The instance that should rerender.
   * @param {object} partialState Next partial state to be merged with state.
   * @param {?function} callback Called after component is updated.
   * @param {?string} Name of the calling function in the public API.
   * @internal
   */
  enqueueSetState: function(
    publicInstance,
    partialState,
    callback,
    callerName,
  ) {
    warnNoop(publicInstance, 'setState');
  },
};

export default ReactNoopUpdateQueue;
```

注意，React API 只是简单的功能介绍，具体的实现是在 react-dom 中，这是因为不同的平台，React API 是一致的，但不同的平台，渲染的流程是不同的，具体的 Component 渲染流程不一致，会根据具体的平台去定制。

组件生命周期请参考  [Hooks 与 React 生命周期的关系](https://github.com/sisterAn/blog/issues/34)

