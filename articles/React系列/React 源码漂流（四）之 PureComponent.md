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

  `React.PureComponent` 中的 `shouldComponentUpdate()` 将跳过所有子组件树的 prop 更新（具体原因参考 [React 生命周期](https://github.com/sisterAn/blog/issues/34)：即：更新阶段，由父至子去判断是否需要重新渲染），所以使用 React.PureComponent 的组件，它的所有 **子组件也必须都为 React.PureComponent** 。

##### PureComponent 源码解析

```js
// 新建了空方法ComponentDummy ，ComponentDummy 的原型 指向 Component 的原型;
function ComponentDummy() {}
ComponentDummy.prototype = Component.prototype;

/**
 * Convenience component with default shallow equality check for sCU.
 */
function PureComponent(props, context, updater) {
  this.props = props;
  this.context = context;
  // If a component has string refs, we will assign a different object later.
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
} // 解析同 React.Component，详细请看上一章

/**
 * 实现 React.PureComponent 对 React.Component 的原型继承
 */
/**
 * 用 ComponentDummy 的原因是为了不直接实例化一个 Component 实例，可以减少一些内存使用
 *
 * 因为，我们这里只需要继承 React.Component 的 原型，直接 PureComponent.prototype = new Component() 的话
 * 会继承包括 constructor 在内的其他 Component 属性方法，但是 PureComponent 已经有自己的 constructor 了，
 * 再继承的话，造成不必要的内存消耗
 * 所以会新建ComponentDummy，只继承Component的原型，不包括constructor，以此来节省内存。
 */
const pureComponentPrototype = (PureComponent.prototype = new ComponentDummy());

// 修复 pureComponentPrototype 构造函数指向
pureComponentPrototype.constructor = PureComponent;

// Avoid an extra prototype jump for these methods.
// 虽然上面两句已经让PureComponent继承了Component
// 但多加一个 Object.assign()，能有效的避免多一次原型链查找
Object.assign(pureComponentPrototype, Component.prototype);

// 唯一的区别，原型上添加了 isPureReactComponent 属性去表示该 Component 是 PureComponent
// 在后续组件渲染的时候，react-dom 会去判断 isPureReactComponent 这个属性，来确定是否浅比较 props、status 实现更新 
/** 在 ReactFiberClassComponent.js 中，有对 isPureReactComponent 的判断
 if (ctor.prototype && ctor.prototype.isPureReactComponent) {
    return (
      !shallowEqual(oldProps, newProps) || !shallowEqual(oldState, newState)
    );
  }
 */
pureComponentPrototype.isPureReactComponent = true;
```

关于 JS 继承，参考 [ JS基础之原型与原型链](https://github.com/sisterAn/blog/issues/5)