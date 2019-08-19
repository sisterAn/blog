### 十、Hooks

Hook 是一个特殊的函数，它可以让你“钩入” React 的特性。所有 Hooks 都以 use 一词开头。一些用于为函数组件增加状态（如 `useState` ），一些可用于管理副作用（如useEffect），一些可用于缓存 memoize 函数和对象（如useCallback、 useMemo）。

**React hook 函数只能用于函数组件。你不能在类组件中使用它们。**

下面是一个基本示例：

```js
const Button = () => {
  let count = 0;

  return (
    <button>{count}</button>
  );
};

ReactDOM.render(<Button />, mountNode);
```



#### 1. 响应用户事件

我们给 `button` 组件增加一个 `onClick` 事件：

```js
const Button = () => {
  let count = 0;

  return (
    <button onClick={() => console.log('Button clicked')}>
      {count}
    </button>
  );
};

ReactDOM.render(<Button />, mountNode);
```

每次我们点击 `Button` 按钮时，`onClick` 都会调用内联箭头函数，向控制台输出 `Button clicked`。

> 请注意：React 处理的所有与 DOM 相关的属性都必须采用驼峰式，否则，React 将会报错。同时，React 还支持使用自定义 HTML 属性，并且必须采用全小写格式。
>
> React 中的一些 DOM 属性与它们在原生 DOM API 中的属性略有不同。例如 `onChange` 事件。在原生浏览器中，当你在表单字段中单击时，通常会触发它。在 React 中，`onChange` 只要更改了表单字段的值就会触发。
>
> React 中的某些属性的命名与 HTML 等效的不同。例如 React 属性 `className` ，它相当于 `class` 在HTML中使用属性。有关 React 属性和 DOM 属性之间差异的完整列表，请参阅[jscomplete.com/react-attributes](https://jscomplete.com/react-attributes)。



#### 阅读和更新状态

跟踪状态更新并触发虚拟树协调算法更新到真实 DOM 上，React 需要了解组件中使用的任何状态元素发生的任何更改。为了更有效的执行此操作，React 需要为组件中引入的每个状态元素使用特殊的 `getter` 和 `setter` 。这就是 `useState` 发挥作用的地方。它定义了一个状态元素，并为它提供了一个 `getter` 和 `setter` ！

**`useState` 是允许你在 React 函数组件中添加 state 的 Hook。**

以下是我们尝试实现的 count state 元素所需的内容：

```js
const [count, setCount] = React.useState(0);
```

 `React.useState()` 函数返回一个包含 2 个元素的数组。我们使用数组解构来命名，第一项是 `useState` 返回的第一个值 `count`（getter），第二项是返回的第二个值	`setCount` 函数（setter）。等价于下面的代码：

```js
var countStateVariable = useState(0); // 返回一个有两个元素的数组
var count = countStateVariable[0]; // 数组里的第一个值
var setCount = countStateVariable[1]; // 数组里的第二个值
```

第一项为定义的 state 变量名称，示例中叫 `count`， 但是我们可以叫他任何名字，比如 `hello`。这是一种在函数调用时保存变量的方式 —— `useState` 是一种新方法，它与 class 里面的 `this.state` 提供的功能完全相同。一般来说，在函数退出后变量就就会”消失”，而 state 中的变量会被 React 保留。

第二个项 "function" 将在调用时更改 `state` 元素的值（如果需要，它将触发DOM处理）。每次 `setCount` 调用该函数时，React 都将重新渲染 `Button` 组件，该组件将刷新组件中定义的所有变量（包括 `count` 值）。我们传递给 `setCount` 的参数将成为新的值 `count` 。

 `React.useState()` 方法里面唯一的参数就是初始 state。不同于 class 的是，它可以是字符串，数字，数组等，而不一定是对象。在示例中，在示例中，我们传了 `0` 作为变量的初始 `state` 。（如果我们想要在 state 中存储两个不同的变量，只需调用 `useState()` 两次即可。）

```js
const Button = () => {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
};

ReactDOM.render(<Button />, mountNode);
```

请注意我们不对 UI 本身进行任何操作，我们只是实现了一个 `action`来改变 JavaScript 对象（在内存中）！React 负责将我们的声明性描述转换为浏览器中的实际UI。

> 除了使用别名 `React.useState` ，还可以直接导入 `useState` 使用：
>
> ```js
> import React, { useState } from 'react';
> ```



### 十一、多组件

将 `Button` 组件拆分为以下两个部分：

- 保持 `Button` 组件为按钮元素，并有一个静态标签；
- 添加一个新 `Display` 组件以显示计数值。

新 `Display` 组件将是纯粹的表示组件，没有自己的状态或交互。这很正常。并非每个 React 组件都必须与状态或交互挂钩。

```js
const Display = (props) => (
  <pre>COUNT VALUE HERE...</pre>
);
```

`Display` 组件的职责是简单地显示它将作为 `props` 接收的值。例如，`pre` 元素用于托管值是该职责的一部分。该应用程序中的其他组件对此没有发言权！

#### 渲染兄弟组件

我们现在有两个要渲染的元素：`Button` 和 `Display` 。我们不能将它们直接相互渲染，如下所示：

```js
// This will not work
ReactDOM.render(<Button /><Display />, mountNode);
```

在React中，相邻的元素不能像这样呈现，因为当 JSX 被转换时，它们中的每一个都被转换为函数调用。有以下几个解决方案。

**方案一：数组形式**

```js
ReactDOM.render([<Button />, <Display />], mountNode);
```

当您渲染的所有元素都来自动态源时，这通常是一个很好的解决方案。但是，对于我们在这里做的情况，它并不理想。

**方案二：添加父组件**

为所有 React 元素添加一个共同的父组件。例如，我们可以将它们包含在 `div` 元素中。

```js
ReactDOM.render(
  <div>
    <Button />
    <Display />
  </div>,
  mountNode
);
```

React API 支持此嵌套。事实上，你可以使用 `React.Fragment`，它不会引入任何新的 DOM 父节点。

**方案三：使用 React.Fragment**

```js
ReactDOM.render(
  <React.Fragment>
    <Button />
    <Display />
  </React.Fragment>,
  mountNode
);
```

**方案三+：使用 <></>**

你也可以这样写：

```js
ReactDOM.render(
  <>
    <Button />
    <Display />
  </>,
  mountNode
);
```

React 会将空标记转换为 `React.Fragment` 语法。

但是，我们应该为 React 创建单一根节点，而不是我们刚刚执行的嵌套树。



#### 顶级组件

让我们创建一个顶层组件来托管 `Button` 和 `Display` 组件。现在的问题是：我们应该为这个新的父组件命名什么？

>  命名组件及其 `state` / `props` 元素是一项非常艰巨的任务，会影响这些组件的工作和执行方式。正确的名称将迫使你做出正确的设计决策。花些时间考虑一下你为 React 应用程序引入的每个新名称。

由于这种新的父组件包含一个 `Display` 显示计数值，一个 `Button` 增加计数值，我们可以把它命名为 `CountManager`。

```js
const CountManager = () => {
  return (
    <>
      <Button />
      <Display />
    </>
  );
};

ReactDOM.render(<CountManager />, mountNode);
```

由于我们将在新 `Display` 组件中显示计数值，因此我们不再需要将计数值显示在按钮上。相反，我们可以在按钮上显示 “+1”。

```js
const Button = () => {
  return (
    <button onClick={() => console.log('TODO: Increment counter')}>
      +1
    </button>
  );
};
```

请注意，我还从 `Button` 组件中删除了 `state` 元素，因为我们不能再使用它了。根据新要求，组件 `Button` 和`Display` 组件都需要访问 `count` state 元素。

当组件需要访问其兄弟组件所拥有的状态元素时，一种解决方案是将该状态元素“提升”到其最近共同父组件上。对于这种情况，父级是 `CountManager` 。

通过将状态移动到 `CountManager` ，我们现在可以使用组件 `props` 将数据从父级传到子级。

```js
const Display = ({ content }) => (
  <pre>{content}</pre>
);

const CountManager = () => {
  const [count, setCount] = useState(0);

  return (
    <>
      <Button />
      <Display content={count} />
    </>
  );
};

ReactDOM.render(<CountManager />, mountNode);
```

通过 `props` 将 `count` 值传递给 `Display` 组件时，注意我使用了不同的名称（`content`）。这很正常。你不必使用完全相同的名称。事实上，在某些情况下，引入新的通用名称对于子组件更好，因为它使状态可重用更高。

父组件也可以将行为传递给他们的孩子。

由于 `count` state元素现在位于 `CountManager` 组件中，因此我们需要在 `CountManager` 中处理更新它。我们来命名更新函数为 `incrementCounter` 。该函数的逻辑实际上与我们之前在组件中的 `handleClick` 函数中使用的逻辑相同。新 `incrementCounter` 函数将更新 `CountManager` 组件 `count` 状态：

```js
const CountManager = () => {
  // ....

  const incrementCounter = () => {
    setCount(count + 1);
  }

  // ...
};
```

为了使 `Button` 组件能够调用组件中的 `incrementCounter` 函数，`CountManager` 将 `incrementCounter` 作为 `props` 参数给 `Button` 组件。

我们命名这个 `props` 为 `clickAction`，值为 `incrementCounter` ，它是我们在 `CountManager` 组件中定义的函数的引用。

```js
const Button = ({ clickAction }) => {
  return (
    <button onClick={clickAction}>
      +1
    </button>
  );
};

// ...

const CountManager = () => {
  // ...

  return (
    <div>
      <Button clickAction={incrementCounter} />
      <Display content={count} />
    </div>
  );
};
```

此 `clickAction` 属性允许 `Button` 组件调用 `CountManager` 组件的`incrementCounter` 功能。

我们分析代码就会知道， `Button` 组件是不知道单击它时会发生什么。它只遵循父级定义的规则并调用泛型 `clickAction` 。父组件控制该行为的执行内容。这遵循责任隔离的概念。这里的每个组件都有一定的责任，它们专注于此。

再看一下 `Display` 。从它的角度来看，计数值不是一个状态。它只是一个 `CountManager` 组件传递给它的内容。 `Display` 组件将始终显示该 `prop`。这也是职责分离。

作为这些组件的设计者，你可以选择他们的职责级别。

我们创建 `CountManager` 组件负责管理计数状态。在项目开发中，又该如何做喃？

我遵循的做法是在共享父节点中定义一个状态元素，该元素尽可能接近需要访问该状态元素的所有子节点。对于像这样的小应用程序，这通常意味着顶级组件本身。在较大的应用程序中，子树可以管理自己的状态“分支”，而不是依赖于在顶级根组件上定义的全局状态元素。

> 顶级组件通常用于管理共享应用程序状态和操作，因为它是所有其他组件的父级。
>
> 请注意，更新顶级组件上的state元素意味着将重新呈现整个组件树（在内存中）。

到目前为止，这是此示例的完整代码：

```js
const Button = ({ clickAction }) => {
  return (
    <button onClick={clickAction}>
      +1
    </button>
  );
};

const Display = ({ content }) => (
  <pre>{content}</pre>
);

const CountManager = () => {
  const [count, setCount] = useState(0);

  const incrementCounter = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <Button clickAction={incrementCounter} />
      <Display content={count} />
    </div>
  );
};
```



### 十二、组件可重用

组件都需要考虑可重用性。上栗中，`Button` 组件也可以重用，它可以使用任何值增加 `count` 计数，而不仅仅是 `+1` 。

首先 `Button` 在 `CountManager` 组件中添加更多元素，以便我们可以测试这个新功能：

```js
const CountManager = () => {
  // ..

  return (
    <>
      <Button clickAction={incrementCounter} /> {/* +1 */}
      <Button clickAction={incrementCounter} /> {/* +5 */}
      <Button clickAction={incrementCounter} /> {/* +10 */}
      <Display count={count} />
    </>
  );
};
```

`Button` 上面呈现的所有元素当前都有一个 `+1` 标签，它们将使计数增加1。我们希望使它们显示特定于每个按钮的不同标签，并使它们根据特定于每个按钮的值执行不同的操作。请记住，你可以将任何值作为 `prop` 传递给 React 元素。

例如：

![UI](https://jscomplete.com/images/reads/react/multi-button-ui.png)

> 在我们完成这个练习之前，花点时间尝试自己实现它。



#### 添加 props

我们需要做的第一件事是使组件中的 `+1` 标签 `Button` 成为可自定义的标签。

为了在 React 组件中进行自定义，我们引入了一个新的 prop（父组件可以控制）并使组件使用其值。在我们的例子中，我们可以让 `Button` 组件接收增量（`1`，`5`，`10`），例如 `clickValue` 。我们可以更改 `render` 方法，`CountManager` 将我们想要测试的值传递给这个新的 `prop` 。

```js
return (
  <>
    <Button clickAction={incrementCounter} clickValue={1} />
    <Button clickAction={incrementCounter} clickValue={5} />
    <Button clickAction={incrementCounter} clickValue={10} />
    <Display content={count} />
  </>
);
```

到目前为止，请注意有关此代码的一些事项：

- 我没有用与之相关的任何内容命名新属性 `count`。该 `Button` 组件无需了解其 `click` 事件的含义。它只需要在 `clickValue` 触发 `click` 事件时传递它。例如，命名这个新属性 `countValue` 不是最佳选择，因为现在，在 `Button` 组件中，我们读取代码以了解 `Button` 元素与计数相关。这使得 `Button` 组件不再可重用。例如，如果我想使用相同的 `Button` 组件将字母附加到字符串，则其代码会令人困惑。
- 我使用大括号来传递新 `clickValue` 属性的值 `(clickValue={5})` 。我没有在那里使用字符串 `(clickValue="5")` 。由于我有一个数学运算来处理这些值（每次 `Button` 点击时），我需要这些值为数字。如果我将它们作为字符串传递，我将不得不在执行添加操作时进行一些字符串到数字的转换。

> 将数字作为字符串传递是React中的常见错误。有关更多与React相关的常见错误，请参阅[此文章](https://jscomplete.com/learn/react-beyond-basics/react-cfp)。



#### 自定义行为

我们需要在`CountManager`组件中制作通用的另一件事是`incrementCounter`动作功能。它不能`count + 1`像现在这样进行硬编码操作。与我们为`Button`组件所做的类似，为了使函数通用，我们使它接收一个参数并使用该参数的值。例如： 

```js
incrementCounter = (incrementValue) => {
  setCount(count + incrementValue);
};
```

现在我们需要做的就是让`Button`组件使用`clickValue`prop作为其标签，并使其作为参数调用其`onClick`动作`clickValue`。

```js
const Button = ({ clickValue, clickAction }) => {
  return (
    <button onClick={() => clickAction(clickValue)}>
      +{clickValue}
    </button>
  );
};
```

注意我是如何使用内联箭头函数包装onClick prop以使其绑定到Button的`clickValue`。这个新箭头函数的JavaScript闭包将负责这一点。

现在，三个按钮应以三个不同的点击值递增：

```js
const Button = ({ clickValue, clickAction }) => {
  return (
    <button onClick={() => clickAction(clickValue)}>
      +{clickValue}
    </button>
  );
};

const Display = ({ content }) => (
  <pre>{content}</pre>
);

const CountManager = () => {
  const [count, setCount] = useState(0);

  const incrementCounter = (increment) =>
    setCount(count + increment);

  return (
    <div>
      <Button clickAction={incrementCounter} clickValue={1} />
      <Button clickAction={incrementCounter} clickValue={5} />
      <Button clickAction={incrementCounter} clickValue={10} />
      <Display content={count} />
    </div>
  );
}

ReactDOM.render(<CountManager />, mountNode);
```



### 十三、接受用户的输入

想象一下，我们需要计算文本区域中用户类型的字符，就像Twitter的推文形式一样。对于每个字符的用户类型，我们需要使用新的字符数更新UI。

这是一个显示`textarea`输入元素的组件，其中包含字符数的占位符div：

```js
const CharacterCounter = () => {
  return (
    <div>
      <textarea cols={80} rows={10} />
      <div>Count: X</div>
    </div>
  );
};

ReactDOM.render(<CharacterCounter />, mountNode);
```

要在用户输入时更新计数`textarea`，我们需要自定义用户键入时触发的事件。React中的此事件实现为`onChange`。我们还需要使用state元素来计算字符数，并在`onChange`事件中触发其updater函数。

在`onChange`我们需要提出的新事件处理程序中，我们需要访问在`textarea`元素中键入的文本。我们需要以某种方式阅读它，因为默认情况下React不知道它。当用户键入时，呈现的UI通过浏览器自己的状态管理进行更改。我们没有指示React根据`textarea`元素的值更改UI 。

我们可以使用两种主要方法来读取值。首先，我们可以直接使用DOM API本身来阅读它。我们需要使用DOM选择API“选择”元素，一旦我们这样做，我们就可以使用`element.value`调用来读取它的值。要选择元素，我们只需给它一个ID并使用`document.getElementById`DOM API来选择它。

因为React渲染`textarea`元素，我们实际上可以通过React本身进行元素选择。React有一个特殊的“ref”属性，我们可以将其分配给每个DOM元素，然后使用它来访问它。

我们也可以`onChange`直接通过事件的目标对象访问元素。每个事件都暴露其目标，并且在目标上的`onChange`事件`textarea`是`textarea`元素。

这意味着我们需要做的就是：

```js
const CharacterCounter = () => {
  const [count, setCount] = useState(0);
  
  const handleChange = (event) => {
    const element = event.target;
    setCount(element.value.length);
  };
  
  return (
    <div>
      <textarea cols={80} rows={10} onChange={handleChange} />
      <div>Count: {count}</div>
    </div>
  );
};

ReactDOM.render(<CharacterCounter />, mountNode);
```

这是最简单的解决方案，它实际上工作正常。这个解决方案的不理想之处在于我们正在混淆问题。该`handleChange`事件具有调用`setCount`函数和计算文本长度的副作用。这实际上不是事件处理程序的关注点。

我们需要混淆这些问题的原因是React不知道输入的是什么。这是一个DOM更改，而不是React更改。

我们可以通过覆盖其值`textarea`并通过React将其更新为状态更改来使其成为React更改。在`onChange`处理程序中，我们只是设置在组件状态上键入的值，而不是对字符进行计数。然后，关于如何处理该值的问题成为React UI渲染逻辑的一部分。以下是使用此策略的解决方案的一个版本：

```js
const CharacterCounter = () => {
  const [inputValue, setInputValue] = useState('');
  
  const handleChange = (event) => {
    const element = event.target;
    setInputValue(element.value);
  };
  
  return (
    <div>
      <textarea cols={80} rows={10} value={inputValue} onChange={handleChange} />
      <div>Count: {inputValue.length}</div>
    </div>
  );
};

ReactDOM.render(<CharacterCounter />, mountNode);
```

虽然这是一个更多的代码，但它有明确的关注点分离。React现在知道输入元素状态。它控制它。此模式称为React中的受控组件模式。

此版本也更容易扩展。如果我们要计算用户输入的单词数量，这将成为另一个UI计算值。无需在州上添加任何其他内容。



### 十四、管理副作用

首次在浏览器中渲染React组件称为“安装”，将其从浏览器中删除称为“卸载”。

安装，更新和卸载组件可能需要具有“副作用”。例如，React TODOs应用程序可能需要在浏览器页面的标题中显示活动TODO项目的数量。这不是您可以直接使用React API执行的操作。您需要使用DOM API。同样，在渲染输入表单时，您可能希望自动对焦文本框。这也必须使用DOM API完成。

副作用通常需要在React的渲染任务之前或之后发生。这就是为什么React在类组件中提供“生命周期方法”以允许您在render方法之前或之后执行自定义操作的原因。您可以在组件首次安装在`componentDidMount`类方法中后执行操作，您可以在组件在`componentDidUpdate`类方法中获取更新后执行操作，并且可以在组件从`componentWillUnmount`类方法中的浏览器中删除之前执行操作。

对于函数组件，使用`React.useEffect`hook函数管理副作用，该函数有2个参数：回调函数和依赖项数组。

```js
useEffect(() => {
  // Do something after each render
  // but only if dep1 or dep2 changed
}, [dep1, dep2]);
```

第一次React呈现一个有`useEffect`调用的组件时，它将调用它的回调函数。在每个新组件呈现之后，如果依赖项的值与之前渲染中的值不同，则React将再次调用回调函数。

> 更新或卸载函数组件时，React可以调用副作用“清理”功能。可以从`useEffect`回调函数返回该清理函数。

副作用方法对于分析应用程序中正在发生的事情以及进一步优化React更新的性能也非常方便。



### 十五、下一步

您准备使用React构建简单但非常棒的应用程序。通过此[交互式实验室](https://jscomplete.com/learn/lab-data-with-react)，您可以轻松使用和管理React应用程序中的数据。然后，建立更大的东西。实用的东西。有趣的事。如果您正在迎接挑战，那就建立一个[简单的记忆游戏](https://jscomplete.com/learn/react-beyond-basics/memory-challenge)吧！

在使用React构建内容时，请在[此列表中](https://jscomplete.com/learn/react-beyond-basics/react-cfp)打开浏览器窗口，其中包含初学者在使用React时常遇到的常见问题。当您遇到问题时，请确保它不是其中之一。

如果你有信心采取深入了解的反应更发达的地区，我有一个[Pluralsight当然](https://jscomplete.com/c/reactjs-advanced)，一个[在线研讨会](https://jscomplete.com/learn/1806-react-workshop)，一[本书](https://jscomplete.com/learn/react-beyond-basics)，而在其他一些书面材料[jsComplete库](https://jscomplete.com/learn)。
