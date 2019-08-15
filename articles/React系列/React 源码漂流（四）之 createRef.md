### 一、Refs

**React v16.3 ** 新增的API，允许我们访问 DOM 节点或在 render 方法中创建的 React 元素。

React 的**核心思想**是每次对于界面 state 的改动，都会重新渲染整个Virtual DOM，然后新老的两个 Virtual DOM 树进行 diff（**协调算法**），对比出变化的地方，然后通过 renderer 渲染到实际的UI界面，

使用 Refs 为我们提供了一种绕过状态更新和重新渲染时访问元素的方法；这在某些用例中很有用，但不应该作为 `props` 和 `state` 的替代方法。

在项目开发中，如果我们可以使用 声明式 或 提升 state 所在的组件层级（状态提升） 的方法来更新组件，最好不要使用 refs。



#### 使用方式

Refs 有 三种实现：

##### 1、方法一：通过 createRef 实现

Refs 是使用 `React.createRef()` 创建的，并通过 `ref` 属性附加到 React 元素。

Refs 通常在 React 组件的构造函数中定义，或者作为函数组件顶层的变量定义，然后附加到 `render()` 函数中的元素。

```react
export default class Hello extends React.Component {
  constructor(props) {
    super(props);
    this.textRef = React.createRef(); // 创建 ref 存储 textRef DOM 元素
  }
  componentDidMount() {
    // 注意：通过 "current" 取得 DOM 节点
    this.textRef.current.focus(); // 直接使用原生 API 使 text 输入框获得焦点
  }
  render() {
    // 把 <input> ref 关联到构造器里创建的 `textRef` 上
    return <input ref={this.textRef} />
  }
}	
```

使用  `React.createRef()` 给组件创建了 Refs 对象。在上面的示例中，ref被命名 `textRef`，然后将其附加到 `<input>` DOM元素。

其中， `textRef` 的属性 `current` 指的是当前附加到 ref 的元素，并广泛用于访问和修改我们的附加元素。事实上，如果我们通过登录 `myRef` 控制台进一步扩展我们的示例，我们将看到该 `current` 属性确实是*唯一*可用的属性：

```js
componentDidMount = () => {
   // myRef only has a current property
   console.log(this.textRef);
   // myRef.current is what we are interested in
   console.log(this.textRef.current);
   // focus our input automatically when component mounts
   this.textRef.current.focus();
}
```

在 `componentDidMount` 生命周期阶段，`myRef.current` 将按预期分配给我们的 `<input>` 元素;  `componentDidMount` 通常是使用 refs 处理一些初始设置的安全位置。 

我们不能在 `componentWillMount` 中更新 Refs，因为此时，组件还没渲染完成， Refs 还为 `null`。

##### 2、方法二：回调 Refs

不同于传递 `createRef()` 创建的 `ref` 属性，你会传递一个函数。这个函数中接受 React 组件实例或 HTML DOM 元素作为参数，以使它们能在其他地方被存储和访问。

```react
import React from 'react';
export default class Hello extends React.Component {
  constructor(props) {
    super(props);
    this.textRef = null; // 创建 ref 为 null
  }
  componentDidMount() {
    // 注意：这里没有使用 "current" 
    this.textRef.focus(); // 直接使用原生 API 使 text 输入框获得焦点
  }
  render() {
    // 把 <input> ref 关联到构造器里创建的 `textRef` 上
    return <input ref={node => this.textRef = node} />
  }
}												
```

React 将在组件挂载时将 DOM 元素传入`ref` 回调函数并调用，当卸载时传入 `null` 并调用它。在 `componentDidMount` 或 `componentDidUpdate` 触发前，React 会保证 refs 一定是最新的。

像上例， `ref` 回调函数是以内联函数的方式定义的，在更新过程中它会被执行两次，第一次传入参数 `null`，然后第二次会传入参数 DOM 元素。

这是因为在每次渲染时会创建一个新的函数实例，所以 React 清空旧的 ref 并且设置新的。我们可以通过将 ref 的回调函数定义成 class 的绑定函数的方式可以避免上述问题，但是大多数情况下它是无关紧要的。

##### 3、方法三：通过 stringRef 实现

```js
export default class Hello extends React.Component {
  constructor(props) {
    super(props);
  }
  componentDidMount() {
    // 通过 this.refs 调用
    this.refs.textRef.focus(); // 直接使用原生 API 使 text 输入框获得焦点
  }
  render() {
    // 把 <input> ref 关联到构造器里创建的 `textRef` 上
    return <input ref='textRef' />
  }
}
```

尽管字符串 stringRef 使用更方便，但是它有[一些缺点](https://github.com/facebook/react/issues/1373)，因此严格模式使用 stringRef 会报警告。官方推荐采用回调 Refs。

#### 使用场景

- **管理焦点（如文本选择）或处理表单数据：** Refs 将管理 `input` 当前焦点选中，或 `input` 其它属性。
- **媒体播放：**基于 React 的音乐或视频播放器可以利用 Refs 来管理其当前状态（播放/暂停），或管理播放进度等。这些更新不需要进行状态管理。
- **触发强制动画：**如果要在元素上触发过强制动画时，可以使用 Refs 来执行此操作。



#### 注意

- 当 `ref` 属性被用于一个普通的 HTML 元素时，`React.createRef()` 将接收底层 DOM 元素作为它的 `current` 属性以创建 `ref` ，我们可以通过 Refs 访问 DOM 元素属性。
- 当 `ref` 属性被用于一个自定义 class 组件时，`ref` 对象将接收该组件已挂载的实例作为它的 `current`，与 `ref` 用于 HTML 元素不同的是，我们能够通过 `ref` 访问该组件的props，state，方法以及它的整个原型 。
- ref 是为了获取某个节点是实例，所以 **你不能在函数式组件上使用 ref 属性**，因为它们没有实例。
- 推荐使用 **回调形式的 refs**， `stringRef` 将会废弃（严格模式下使用会报警告），`React.createRef()` API 是 React v16.3 引入的更新。
- 避免使用 refs 来做任何可以通过 **声明式** 实现来完成的事情



### 三、createRef 与 Hook useRef

#### useRef

`useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的值。返回的 ref 对象在组件的整个生命周期内保持不变。

```js
function Hello() {
  const textRef = useRef(null)
  const onButtonClick = () => {
    // 注意：通过 "current" 取得 DOM 节点
    textRef.current.focus();
  };
  return (
    <>
      <input ref={textRef} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  )
}
```



####区别

`useRef()` 比 `ref` 属性更有用。`useRef()` Hook 不仅可以用于 DOM refs。「ref」 对象是一个 `current` 属性可变且可以容纳任意值的通用容器，类似于一个 class 的实例属性。

```js
function Timer() {
  const intervalRef = useRef();

  useEffect(() => {
    const id = setInterval(() => {
      // ...
    });
    intervalRef.current = id;
    return () => {
      clearInterval(intervalRef.current);
    };
  });

  // ...
}
```



这是因为它创建的是一个普通 Javascript 对象。而 `useRef()` 和自建一个 `{current: ...}`对象的唯一区别是，`useRef` 会在每次渲染时返回同一个 ref 对象。

请记住，当 ref 对象内容发生变化时，`useRef` 并*不会*通知你。变更 `.current` 属性不会引发组件重新渲染。如果想要在 React 绑定或解绑 DOM 节点的 ref 时运行某些代码，则需要使用[回调 ref](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-can-i-measure-a-dom-node) 来实现。



### 四、createRef 源码解析

```react
// ReactCreateRef.js 文件
import type {RefObject} from 'shared/ReactTypes';

// an immutable object with a single mutable value
export function createRef(): RefObject {
  const refObject = {
    current: null,
  };
  if (__DEV__) {
    Object.seal(refObject); // 封闭对象，阻止添加新属性并将所有现有属性标记为不可配置。当前属性的值只要可写就可以改变。
  }
  return refObject;
}
```

其中 `RefObject` 为： 

```js
export type RefObject = {|
  current: any,
|};
```

这里只是简单的 createRef API 介绍，具体如何使用，后期将详细介绍。
