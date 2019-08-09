### 一、createRef

**React v16.3**新增的API，允许我们访问 DOM 节点或在 render 方法中创建的 React 元素。

React 的**核心思想**是每次对于界面 state 的改动，都会重新渲染整个Virtual DOM，然后新老的两个 Virtual DOM 树进行 diff（**协调算法**），对比出变化的地方，然后通过 renderer 渲染到实际的UI界面，

在项目开发中，如果我们可以使用 声明式 或 提升 state 所在的组件层级（状态提升） 的方法来更新组件，最好不要使用 refs。

Refs 有 三种实现：

- 方法一：通过 stringRef 实现

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

- 方法二：通过 createRef 实现

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

- 方法三：回调 Refs

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

  React 将在组件挂载时将 DOM 元素传入`ref` 回调函数并调用，当卸载时传入 `null` 并调用它。`ref` 回调函数会在 `componentDidMount` 和 `componentDidUpdate` 生命周期函数前被调用

注意：

- 当 `ref` 属性被用于一个普通的 HTML 元素时，`React.createRef()` 将接收底层 DOM 元素作为它的 `current`属性以创建 `ref` 。
- 当 `ref` 属性被用于一个自定义类组件时，`ref` 对象将接收该组件已挂载的实例作为它的 `current` 。
- ref 是为了获取某个节点是实例，所以 **你不能在函数式组件上使用 ref 属性**，因为它们没有实例。
- 推荐使用 **回调形式的 refs**， `stringRef` 将会废弃，`React.createRef()` API 是 React v16.3 引入的更新。
- 避免使用 refs 来做任何可以通过 **声明式** 实现来完成的事情

### 二、createRef 与 Hook useRef

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

### 三、createRef 源码解析

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
