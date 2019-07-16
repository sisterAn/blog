#### PureComponent

##### PureComponent 用法

PureComponent 最早在 React v15.3 版本中发布，主要是为了优化 React 应用而产生。

- **实现**

  它和 React.Component类似，都是定义一个组件类。不同是 React.Component 没有实现 `shouldComponentUpdate()`，而 React.PureComponent 通过 props 和 state 的 **浅比较** 实现了。

- **使用场景**

  当 React.Component 的 props 和 state 均为基本类型，使用 React.PureComponent 会节省应用的性能

- **可能出现的问题及解决方案**

  当props 或 state 为 **复杂的数据结构** （例如：嵌套对象和数组）时，因为 React.PureComponent 仅仅是 **浅比较** ，可能会渲染出 **错误的结果** 。这时有 **两种解决方案** ：

  - 当 **知道** 有深度数据结构更新时，可以直接调用 **forceUpdate**  强制更新
  - 考虑使用  [immutable objects](https://facebook.github.io/immutable-js/) （不可突变的对象），实现快速的比较对象

- **注意**

  - 使用 React.PureComponent 的组件，它的所有子组件化也必须都为 React.PureComponent。



##### PureComponent 源码解析