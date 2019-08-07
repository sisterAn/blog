### 一、PureComponent

`PureComponent` 最早在 React v15.3 版本中发布，主要是为了优化 React 应用而产生。

```js
class Counter extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

在这段代码中， `React.PureComponent` 会浅比较 `props.color` 或 `state.count` 是否改变，来决定是否重新渲染组件。

- **实现**

  `React.PureComponent` 和 `React.Component` 类似，都是定义一个组件类。不同是 `React.Component` 没有实现 `shouldComponentUpdate()`，而 `React.PureComponent` 通过 props 和 state 的 **浅比较** 实现了。

- **使用场景**

  当 `React.Component` 的 props 和 state 均为基本类型，使用 `React.PureComponent` 会节省应用的性能

- **可能出现的问题及解决方案**

  当props 或 state 为 **复杂的数据结构** （例如：嵌套对象和数组）时，因为 `React.PureComponent` 仅仅是 **浅比较** ，可能会渲染出 **错误的结果** 。这时有 **两种解决方案** ：

  - 当 **知道** 有深度数据结构更新时，可以直接调用 **forceUpdate**  强制更新
  - 考虑使用  [immutable objects](https://facebook.github.io/immutable-js/) （不可突变的对象），实现快速的比较对象

- **注意**

  `React.PureComponent` 中的 `shouldComponentUpdate()` 将跳过所有子组件树的 prop 更新（具体原因参考 [Hooks 与 React 生命周期](https://github.com/sisterAn/blog/issues/34)：即：更新阶段，由父至子去判断是否需要重新渲染），所以使用 React.PureComponent 的组件，它的所有 **子组件也必须都为 React.PureComponent** 。

### 二、PureComponent 与 Stateless Functional Component 

对于 React 开发人员来说，知道何时在代码中使用 **Component**，**PureComponent ** 和 **Stateless Functional Component** 非常重要。

首先，让我们看一下无状态组件。

#### 无状态组件

输入输出数据完全由 `props` 决定，而且不会产生任何副作用。

```js
const Button = props =>
  <button onClick={props.onClick}>
    {props.text}
  </button>
```

无状态组件可以通过减少继承 `Component` 而来的生命周期函数而达到性能优化的效果。从本质上来说，无状态组件就是一个单纯的 `render` 函数，所以无状态组件的缺点也是显而易见的。因为它没有 `shouldComponentUpdate` 生命周期函数，所以每次 `state` 更新，它都会重新绘制 `render` 函数。

React 16.8 之后，React 引入 Hooks 。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。

#### 何时使用 `PureComponent`？

`PureComponent` 提高了性能，因为它减少了应用程序中的渲染操作次数，这对于复杂的 UI 来说是一个巨大的胜利，因此建议尽可能使用。此外，还有一些情况需要使用 `Component` 的生命周期方法，在这种情况下，我们不能使用无状态组件。

#### 何时使用无状态组件？

无状态组件易于实施且快速实施。它们适用于非常小的 UI 视图，其中重新渲染成本无关紧要。它们提供更清晰的代码和更少的文件来处理。

### 三、PureComponent 与 React.memo

`React.memo` 为高阶组件。它实现的效果与 `React.PureComponent` 相似，不同的是：

- `React.memo` 用于函数组件
- `React.PureComponent` 适用于 class 组件
- `React.PureComponent` 只是浅比较 `props`、`state`，`React.memo` 也是浅比较，但它可以自定义比较函数

#### React.memo

```js
function MyComponent(props) {
  /* 使用 props 渲染 */
}

// 比较函数
function areEqual(prevProps, nextProps) {
  /*
  如果把 nextProps 传入 render 方法的返回结果与
  将 prevProps 传入 render 方法的返回结果一致则返回 true，
  否则返回 false
  返回 true，复用最近一次渲染
  返回 false，重新渲染
  */
}

export default React.memo(MyComponent, areEqual);
```

- `React.memo` 通过记忆组件渲染结果的方式实现 ，提高组件的性能
- 只会对 `props` 浅比较，如果相同，React 将跳过渲染组件的操作并直接复用最近一次渲染的结果。
- 可以将自定义的比较函数作为第二个参数，实现自定义比较
- 此方法仅作为**性能优化**的方式而存在。但请不要依赖它来“阻止”渲染，这会产生 bug。
- 与 class 组件中 `shouldComponentUpdate()` 方法不同的是，如果 props 相等，`areEqual`会返回 `true`；如果 props 不相等，则返回 `false`。这与 `shouldComponentUpdate` 方法的返回值相反。

### 四、使用 PureComponent 常见误区

#### 误区一：在渲染方法中创建函数

如果你在 `render` 方法里创建函数，那么使用 `props` 会抵消使用 `React.PureComponent` 带来的优势。因为每次渲染运行时，都会分配一个新函数，如果你有子组件，即使数据没有改变，它们也会重新渲染，因为浅比较 `props` 的时候总会得到 `false`。

例如：

```js
// FriendsItem 在父组件引用样式
<FriendsItem
  key={friend.id}
  name={friend.name}
  id={friend.id}
  onDeleteClick={() => this.deleteFriends(friend.id)}
/> 
// 在父组件中绑定
// 父组件在 props 中传递了一个箭头函数。箭头函数在每次 render 时都会重新分配（和使用 bind 的方式相同）
```

其中，`FriendsItem` 为 `PureComponent`：

```js
// 其中 FriendsItem 为 PureComponent
class FriendsItem extends React.PureComponent {
  render() {
    const { name, onDeleteClick } = this.props
    console.log(`FriendsItem：${name} 渲染`)
    return (
      <div>
        <span>{name}</span>
        <button onClick={onDeleteClick}>删除</button>
      </div>
    )   
  }
}
// 每次点击删除操作时，未删除的 FriendsItem 都将被重新渲染
```

[点击查看在线实例](https://stackblitz.com/edit/react-ta6tww)

这种在 `FriendsItem` 直接调用 `() => this.deleteFriends(friend.id)`，看起来操作更简单，逻辑更清晰，但它有一个有一个最大的弊端，甚至打破了像 `shouldComponentUpdate` 和 `PureComponent` 这样的性能优化。

这是因为：父组件在 `render` 声明了一个函数`onDeleteClick`，每次父组件渲染都会重新生成新的函数。因此，每次父组件重新渲染，都会给每个子组件 `FriendsItem` 传递不同的 `props`，导致每个子组件都会重新渲染， 即使 `FriendsItem` 为 `PureComponent`。　

**避免在 render 方法里创建函数并使用它。它会打破了像 shouldComponentUpdate 和 PureComponent 这样的性能优化。**

要解决这个问题，只需要将原本在父组件上的绑定放到子组件上即可。`FriendsItem ` 将始终具有相同的 `props`，并且永远不会导致不必要的重新渲染。

```js
// FriendsItem 在父组件引用样式
<FriendsItem 
  key={friend.id} 
  id={friend.id} 
  name={friend.name} 
  onClick={this.deleteFriends} 
/>
```

`FriendsItem`:

```js
class FriendsItem extends React.PureComponent {
  onDeleteClick = () => {
    this.props.onClick(this.props.id)
  } // 在子组件中绑定
  render() {
    const { name } = this.props
    console.log(`FriendsItem：${name} 渲染`)
    return (
      <div>
        <span>{name}</span>
        <button onClick={this.onDeleteClick}>删除</button>
      </div>
    )   
  }
}
// 每次点击删除操作时，FriendsItem 都不会被重新渲染
```

[点击查看在线实例](https://stackblitz.com/edit/react-sr5yvp)

通过此更改，当单击删除操作时，其他 `FriendsItem` 都不会被重新渲染了 👍

#### 误区二：在渲染方法中派生 state

考虑一个文章列表，您的个人资料组件将从中显示用户最喜欢的 10 个作品。

```js
render() {
  const { posts } = this.props
  // 在渲染函数中生成 topTen，并渲染
  const topTen = [...posts].sort((a, b) => 
    b.likes - a.likes).slice(0, 9)
  return //...
}
// 这会导致组件每次重新渲染，都会生成新的 topTen，导致不必要的渲染
```

`topTen`每次组件重新渲染时都会有一个全新的引用，即使 `posts` 没有更改，派生 `state` 也是相同的。

这个时候，我们应该将 `topTen` 的判断逻辑提取到 `render` 函数之外，通过缓存派生 `state` 来解决此问题。

例如，在组件的状态中设置派生 `state`，并仅在 `posts` 已更新时更新。

```js
componentWillMount() {
  this.setTopTenPosts(this.props.posts)
}
componentWillReceiveProps(nextProps) {
  if (this.props.posts !== nextProps.posts) {
    this.setTopTenPosts(nextProps.posts)
  }
}
// 每次 posts 更新时，更新派生 state，而不是在渲染函数中重新生成
setTopTenPosts(posts) {
  this.setState({
    topTen: [...posts].sort((a, b) => b.likes - a.likes).slice(0, 9)
  })
}
```

#### 总结

在使用 `PureComponent` 时，请注意：

- 突变一般是不好的，但在使用 `PureComponent` 时，问题会更加复杂。
- 不要在渲染方法中创建新函数、对象或数组，这不导致项目性能显著降低。

### 五、PureComponent 源码解析

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



这里只是 `PureComponent` 的声明创建，至于如何实现 `shouldComponentUpdate()` ，核心代码在：

```js
// ReactFiberClassComponent.js
function checkShouldComponentUpdate(
  workInProgress,
  ctor,
  oldProps,
  newProps,
  oldState,
  newState,
  nextContext,
) {
  // ...
  if (ctor.prototype && ctor.prototype.isPureReactComponent) {
    // 如果是纯组件，比较新老 props、state
    // 返回 true，重新渲染，
    // 即 shallowEqual props 返回 false，或 shallowEqual state 返回 false
    return (
      !shallowEqual(oldProps, newProps) || !shallowEqual(oldState, newState)
    );
  } 
  return true;
}
```



**shallowEqual.js**

```js
/**
 * 通过遍历对象上的键并返回 false 来执行相等性
 * 在参数列表中，当任意键对应的值不严格相等时，返回 false。
 * 当所有键的值严格相等时,返回 true。
 */
function shallowEqual(objA: mixed, objB: mixed): boolean {
  // 通过 Object.is 判断 objA、objB 是否相等
  if (is(objA, objB)) {
    return true;
  }

  if (
    typeof objA !== 'object' ||
    objA === null ||
    typeof objB !== 'object' ||
    objB === null
  ) {
    return false;
  }
    
  // 参数列表
  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);
    
  // 参数列表长度不相同
  if (keysA.length !== keysB.length) {
    return false;
  }

  // 比较参数列表每一个参数，但仅比较一层
  for (let i = 0; i < keysA.length; i++) {
    if (
      !hasOwnProperty.call(objB, keysA[i]) ||
      !is(objA[keysA[i]], objB[keysA[i]])
    ) {
      return false;
    }
  }

  return true;
}
```

#### 附：Object.is（来自MDN）

`Object.is()` 判断两个值是否[相同](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness)。

这种相等性判断逻辑和传统的 [`==`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Equality) 运算不同，[`==`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Equality) 运算符会对它两边的操作数做隐式类型转换（如果它们类型不同），然后才进行相等性比较，（所以才会有类似 `"" == false` 等于 `true` 的现象），但 `Object.is` 不会做这种类型转换。

这与 [`===`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Identity) 运算符的判定方式也不一样。[`===`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Identity) 运算符（和[`==`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Equality) 运算符）将数字值 `-0` 和 `+0` 视为相等，并认为 [`Number.NaN`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/NaN) 不等于 [`NaN`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/NaN)。

如果下列任何一项成立，则两个值相同：

- 两个值都是 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)
- 两个值都是 [`null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null)
- 两个值都是 `true` 或者都是 `false`
- 两个值是由相同个数的字符按照相同的顺序组成的字符串
- 两个值指向同一个对象
- 两个值都是数字并且
  - 都是正零 `+0`
  - 都是负零 `-0`
  - 都是 [`NaN`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/NaN)
  - 都是除零和 [`NaN`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/NaN) 外的其它同一个数字
  
