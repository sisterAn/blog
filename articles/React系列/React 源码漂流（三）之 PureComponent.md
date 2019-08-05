### 一、PureComponent 用法

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

  `React.PureComponent` 中的 `shouldComponentUpdate()` 将跳过所有子组件树的 prop 更新（具体原因参考 [Hooks 与 React 生命周期](https://github.com/sisterAn/blog/issues/34)：即：更新阶段，由父至子去判断是否需要重新渲染），所以使用 React.PureComponent 的组件，它的所有 **子组件也必须都为 React.PureComponent** 。

### 二、误区

#### 误区一：在渲染中绑定函数中的值

这是React中的常见场景：您正在映射数组，并且您需要每个项目调用单击处理程序并传递一些相关数据。

这是一个例子。我正在遍历用户列表并将userId传递给第31行的deleteUser函数。

```js
import React from "react";
import { render } from "react-dom";
import User from "./User";

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      users: [
        { id: 1, name: "Cory" },
        { id: 2, name: "Meg" },
        { id: 3, name: "Bob" }
      ]
    };
  }

  deleteUser = id => {
    this.setState(prevState => {
      return {
        users: prevState.users.filter(user => user.id !== id)
      };
    });
  };

  render() {
    return (
      <div>
        <h1>Users</h1>
        <ul>
          {this.state.users.map(user => {
            return (
              <User
                key={user.id}
                name={user.name}
                id={user.id}
                onDeleteClick={() => this.deleteUser(user.id)}
              />
            );
          })}
        </ul>
      </div>
    );
  }
}

export default App;

render(<App />, document.getElementById("root"));
```

原因如下：父组件向下传递 `props` 上的箭头函数。每个渲染都会重新分配箭头函数（使用bind的故事相同）。因此，虽然我已将 `User` 声明为 `PureComponent`，但 `User` 的父级中的箭头函数会导致 `User` 组件看到所有用户在 `props` 上发送的新函数。因此，每个用户在单击**任何**删除按钮时都会重新渲染。👎

**避免使用箭头函数并在渲染中绑定。它打破了像shouldComponentUpdate和PureComponent这样的性能优化。**

要解决这个问题，只需将对父级原型方法的引用传递给子级。孩子的`likeComment`将始终具有相同的 `props`，并且永远不会导致不必要的重新渲染。

```js
import React from 'react';
import { render } from 'react-dom';
import User from './User';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      users: [
        { id: 1, name: 'Cory' }, 
        { id: 2, name: 'Meg' }, 
        { id: 3, name: 'Bob'}
      ],
    };
  }

  deleteUser = id => {
    this.setState(prevState => {
      return { 
        users: prevState.users.filter(user => user.id !== id) 
      };
    });
  };

  renderUser = user => {
    return <User key={user.id} user={user} onClick={this.deleteUser} />;
  }

  render() {
    return (
      <div>
        <h1>Users</h1>
        <ul>
          {this.state.users.map(this.renderUser)}
        </ul>
      </div>
    );
  }
}

render(<App />, document.getElementById('root'));

```

在此示例中，index.js在渲染中没有箭头函数。而是将相关数据传递给User.js. 在User.js中，onDeleteClick使用相关的user.id调用在props上传递的onClick函数。

通过此更改，当您单击“删除”时，请注意不会为其他用户调用渲染！👍

为获得最佳性能，

1. 避免使用箭头函数并在渲染中绑定。
2. 怎么样？[提取子组件](https://medium.freecodecamp.org/react-pattern-extract-child-components-to-avoid-binding-e3ad8310725e)，或[传递HTML元素上的数据](https://medium.com/@mgnrsb/another-way-to-avoid-binding-in-render-in-simple-cases-like-this-where-all-you-need-is-to-remember-68af83da0258)。

#### 误区二：在渲染方法中派生数据

考虑一个文章列表，您的个人资料组件将从中显示用户最喜欢的10个作品。

```js
render() {
  const { posts } = this.props
  const topTen = [...posts].sort((a, b) => 
    b.likes - a.likes).slice(0, 9)
  return //...
}
```

`topTen`每次组件重新渲染时都会有一个全新的引用，即使`posts`没有更改，派生数据也是相同的。然后，这将不必要地重新呈现列表。

您可以通过缓存派生数据来解决此问题。例如，在组件的状态中设置派生数据，并仅在`posts`已更新时更新。

```js
componentWillMount() {
  this.setTopTenPosts(this.props.posts)
}
componentWillReceiveProps(nextProps) {
  if (this.props.posts !== nextProps.posts) {
    this.setTopTenPosts(nextProps.posts)
  }
}
setTopTenPosts(posts) {
  this.setState({
    topTen: [...posts].sort((a, b) => b.likes - a.likes).slice(0, 9)
  })
}
```

#### 总结

只要遵循两个简单的规则，使用 `PureComponent` 而不是 `Component` 是安全的：

- 突变一般是不好的,但在使用 `PureComponent` 时,问题会更加复杂。
- 如果要在渲染方法中创建新函数、对象或数组，这可能会导致项目出现问题。

### 三、PureComponent 源码解析

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