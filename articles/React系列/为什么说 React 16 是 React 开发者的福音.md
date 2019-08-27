![React](https://miro.medium.com/max/2400/1*YG3-T77xGBfKDn5SfE6P8w.jpeg)

就像人们对更新他们的移动应用程序和操作系统感到兴奋一样，开发人员也应该对更新他们的框架感到兴奋。不同框架的新版本带有开箱即用的新功能和技巧。

以下是将现有应用从 React 15 迁移到React 16时应考虑的一些优秀功能。

> *现在已经是 React 16.9 了，是时候说再见 React15 👋*



### 错误处理

![img](https://miro.medium.com/freeze/max/60/1*UH_OYTog9NJi3o3kooA_vg.gif?q=20)

错误处理就像:)

React 16 引入了*错误边界*的新概念。

错误边界是**在其子组件树中的任何位置捕获JavaScript错误的** React组件**。****他们记录这些错误，并显示后备UI**而不是崩溃的组件树。错误边界在渲染期间，生命周期方法以及它们下面的整个树的构造函数中捕获错误。

如果类组件定义了一个名为的新生命周期方法，则它将成为错误边界`componentDidCatch(error, info)`：

然后，您可以将其用作常规组件。

```
<ErrorBoundary>   
   <MyWidget /> 
</ ErrorBoundary>
```

该`componentDidCatch()`方法的工作方式类似于JavaScript `catch {}`块，但适用于组件。只有类组件可以是错误边界。实际上，大多数情况下，您只需要声明一次错误边界组件。然后你将在整个应用程序中使用它。

请注意，**错误边界仅捕获树中它们下面的组件中的错误**。错误边界本身无法捕获错误。如果错误边界尝试呈现错误消息失败，则错误将传播到其上方最接近的错误边界。这也类似于`catch {}`JavaScript在JavaScript中的工作方式。

查看现场演示：

ComponentDidCatch

有关错误处理的更多信息，请[访问此处](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)。



### 新的渲染返回类型：片段和字符串

摆脱渲染时将组件包装在div中。

您现在可以从组件的`render`方法返回一个元素数组。与其他数组一样，您需要为每个元素添加一个键以避免键警告：

```
render（）{ 
  //无需在额外元素中包装列表项！
  return [ 
    //不要忘记键:) 
    <li key =“A”>第一项</ li>，
    <li key =“B”>第二项</ li>，
    <li key =“C”>第三项</ li>，
  ]; 
}
```

[从React 16.2.0开始](https://reactjs.org/blog/2017/11/28/react-v16.2.0-fragment-support.html)，它支持JSX的特殊片段语法，不需要键。

支持返回字符串：

```
render（）{ 
  return'看看马，没有跨度！'; 
}
```



### 门户

门户网站提供了一种将子项呈现为存在于父组件的DOM层次结构之外的DOM节点的第一类方法。

```
ReactDOM.createPortal（子，容器）
```

第一个参数（`child`）是任何[可渲染的React子](https://reactjs.org/docs/react-component.html#render)元素，例如元素，字符串或片段。第二个参数（`container`）是一个DOM元素。



### 如何使用它

从组件的render方法返回一个元素时，它作为最近父节点的子节点挂载到DOM中：

```
render（）{ 
  // React安装一个新的div并将子项呈现给它
  返回（
    <div> 
      {this.props.children} 
    </ div> 
  ）; 
}
```

有时将子项插入DOM中的其他位置很有用：

```
render（）{ 
  // React确实*不*创建一个新的div。它将孩子们变成了“domNode”。
  //`domNode`是任何有效的DOM节点，无论它在DOM中的位置如何。
  return ReactDOM.createPortal（
    this.props.children，
    domNode 
  ）; 
}
```

门户网站的典型用例是父组件具有样式`overflow: hidden`或`z-index`样式，但您需要子项在视觉上“突破”其容器。例如，对话框，悬停卡和工具提示。

门户



### 自定义DOM属性

![img](https://miro.medium.com/max/60/1*6h94cJ7rOVdaykMmyhOvhg.png?q=20)

React15用于忽略任何未知的DOM属性。它会跳过它们，因为React不承认它。

```
//你的代码：
<div mycustomattribute =“something”/>
```

使用React 15为DOM渲染一个空div：

```
// React 15输出：
<div />
```

在React16中，输出将如下（*将显示自定义属性，根本不会被忽略*）：

```
// React 16输出：
<div mycustomattribute =“something”/>
```



### 避免在状态中设置NULL时重新渲染

![img](https://miro.medium.com/max/60/1*mDNqHOCtoVeKTPR4gtfP2Q.png?q=20)

使用React16，您可以直接阻止状态更新和重新呈现`setState()`。你只需要让你的函数返回`null`。

```
const MAX_PIZZAS = 20; 

function addAnotherPizza（state，props）{ 
  //如果我有足够的比萨饼，停止更新和重新渲染。
  if（state.pizza === MAX_PIZZAS）{ 
    return null; 
  } 

  //如果没有，请保持比萨饼来！：D 
  返回{ 
    pizza：state.pizza + 1，
  } 
} 

this.setState（addAnotherPizza）;
```

[在这里](https://x-team.com/blog/react-render-setstate/)阅读更多。



### 创建参考

现在使用React16创建refs要容易得多。为什么需要使用refs：

- 管理焦点，文本选择或媒体播放。
- 触发势在必行的动画。
- 与第三方DOM库集成。

Refs是使用属性创建的，`React.createRef()`并通过`ref`属性附加到React元素。在构造组件时，通常将Refs分配给实例属性，以便可以在整个组件中引用它们。

```
class MyComponent扩展了React.Component { 
  constructor（props）{ 
    super（props）; 
    this.myRef = React.createRef（）; 
  } 
  render（）{ 
    return <div ref = {this.myRef} />; 
  } 
}
```



### 访问参考

将ref传递给元素时`render`，可以在`current`ref 的属性中访问对该节点的引用。

```
const node = this.myRef.current;
```

ref的值根据节点的类型而有所不同：

- 当在`ref`HTML元素上使用该属性时，`ref`在构造函数中创建的属性将`React.createRef()`接收底层DOM元素作为其`current`属性。
- 在`ref`自定义类组件上使用该属性时，该`ref`对象将接收组件的已安装实例作为其`current`。
- **您可能无法**`**ref**`**在功能组件上****使用该****属性，**因为它们没有实例。



### Context API

Context提供了一种通过组件树传递数据的方法，而无需在每个级别手动传递props。

### `React.createContext`

```
const {Provider，Consumer} = React.createContext（defaultValue）;
```

创建一`{ Provider, Consumer }`对。当React呈现上下文时`Consumer`，它将从`Provider`树中位于其上方的最接近匹配处读取当前上下文值。

该`defaultValue`参数**仅**在消费者在树中没有匹配的提供者时使用。这可以帮助单独测试组件而不包装它们。注意：`undefined`作为Provider值传递不会导致使用者使用`defaultValue`。

### `Provider`

```
<提供者值= {/ *某些值* /}>
```

一个React组件，允许消费者订阅上下文更改。

接受`value`将要传递给作为此提供者后代的消费者的道具。一个提供商可以连接到许多消费者。可以嵌套提供程序以覆盖树中更深层的值。

### `Consumer`

```
<Consumer> 
  {value => / *根据上下文值* /} 
</ Consumer> 呈现内容
```

订阅上下文更改的React组件。

需要[作为孩子](https://reactjs.org/docs/render-props.html#using-props-other-than-render)的[功能](https://reactjs.org/docs/render-props.html#using-props-other-than-render)。该函数接收当前上下文值并返回React节点。`value`传递给函数的参数将等于`value`树中上述此上下文的最接近的Provider 的prop。如果上面没有此上下文的Provider，则`value`参数将等于`defaultValue`传递给它的参数`createContext()`。

### `static getDerivedStateFromProps()`

`getDerivedStateFromProps`在调用render方法之前调用。初始安装和后续更新。它应返回一个对象来更新状态，或者返回null以不更新任何内容。

此方法适用于[罕见的用例](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#when-to-use-derived-state)，其中状态取决于道具随时间的变化。例如，实现一个`<Transition>`比较其上一个和下一个子节点的组件可能很方便，以决定它们中的哪个动画进出。

导出状态会导致冗长的代码并使您的组件难以思考。

[确保您熟悉更简单的替代方案：](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)

- 如果您需要**执行副作用**（例如，数据提取或动画）以响应道具的更改，请改用`componentDidUpdate`生命周期。
- 如果只想**在prop更改时重新计算某些数据**，请[改用memoization helper](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization)。
- 如果你想**在支柱改变时“重置”一些状态**，可以考虑使一个组件[完全受控制](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-controlled-component)或[完全不受控制](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key)`key`。

此方法无权访问组件实例。如果您愿意，可以`getDerivedStateFromProps()`通过在类定义之外提取组件props和状态的纯函数来重用一些代码和其他类方法。

请注意，无论原因如何，都会在*每个*渲染上触发此方法。这与之形成鲜明对比`UNSAFE_componentWillReceiveProps`。它仅在父项导致重新呈现而不是本地结果时触发`setState`。

我们比较`nextProps.someValue`有`this.props.someValue.`如果两者都不同，那么我们执行一些操作，`setState`

```
static getDerivedStateFromProps（nextProps，prevState）{if（nextProps.someValue！== prevState.someValue）{      
   return {someState：nextProps.someValue};  
} else return null;}
```

它收到两个参数`nextProps`和`prevState`。如前所述，您无法访问`this`此方法。您必须将状态存储在状态中以`nextProps`与之前的道具进行比较。在上面的代码`nextProps`和`prevState`进行比较。如果两者不同，则返回一个对象以更新状态。否则`null`将返回指示不需要状态更新。如果状态发生变化，则会`componentDidUpdate`调用我们可以执行所需操作的位置`componentWillReceiveProps`。



### 奖励：React Lifecycle事件

![img](https://miro.medium.com/max/60/1*6sVjMFCtW_dS_2MserkyQw.jpeg?q=20)

生命周期信用 - <https://twitter.com/dceddia>

这些是使用React16时一定要尝试的一些功能！

快乐的编码💻😀