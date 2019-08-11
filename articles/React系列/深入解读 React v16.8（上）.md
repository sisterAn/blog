### 一、React

React 官网上定义：

**React 是一个用于构建用户界面的 JavaScript 库。**

首先，让我们看一下这个定义的两个不同部分：

#### 1. React 是一个 JavaScript 库

这意味着它不完全是一个 **框架** 。它不是一个完整的解决方案，你经常需要使用更多的库来辅助 React 形成一套完整的解决方案。React 不对解决方案中的其他部分作任何假设。

框架是一个伟大的目标，特别是对于年轻的团队和初创公司。在使用框架时，已经为你做出了许多明智的设计决策，这为你提供了一条清晰的道路，可以专注于编写良好的应用程序逻辑。但是，框架存在一些缺点。对于从事大型代码库开发工作的，并且经验丰富的开发人员来说，这些缺点有时会极具破坏性的。

尽管有些人声称，框架并不灵活。框架通常希望你以某种方式编码所有内容。如果你试图偏离这种方式，框架经常会为此与你发生冲突。框架通常很大并且功能齐全，如果你只需要使用它们中的一小部分，你必须要引入整个框架。不可否认今天这一点正在改变，但仍然不理想，一些框架正在模块化，我认为这很棒，但我是纯 Unix 哲学的忠实粉丝：

> 编写做一件事并做得好的程序。编写程序以协同工作。 - 道格麦克罗伊

React 遵循 Unix 哲学，因为它是一个小型库，专注于一件事并且非常好地完成这件事。“一件事” 是React定义的第二部分：**构建用户界面**。

#### 2. React 用于构建用户界面

用户界面（UI）是展现在用户面前，用于与机器交互的媒介。用户界面无处不在，从微波炉上的简单按钮到航天飞机的仪表板。如果我们尝试连接的设备可以识别 JavaScript ，我们就可以使用 React 来描述它的 UI 。由于 Web 浏览器识别 JavaScript ，我们可以使用 React 来描述 Web UI 。

我们只需要告诉浏览器我们想要什么！React 将代表我们在 Web 浏览器中构建实际的 UI。如果没有React或类似的库，我们需要使用原生 Web API 和 JavaScript 手动构建 UI，这并不容易。

当你听到 **React 是声明** 的陈述时，这正是它的含义。我们用 React 描述 UI 并告诉它我们想要什么（而不是如何做）。React将负责“how”并将我们的声明性描述（我们用React语言编写）转换为浏览器中的实际UI。React 与 HTML 本身共享这种简单的声明能力，但是使用React，我们可以声明代表动态数据的HTML UI，而不仅仅是静态数据。

当 React 发布时，有很多关于它性能的质疑，因为它引入了一个虚拟 DOM 的聪明想法，可以用来协调实际的DOM（我们将在下一节讨论）。

**DOM是文档对象模型（Document Object Model）。它是HTML（和XML）文档的浏览器编程接口，将它们视为树结构。DOM API可用于更改文档结构，样式和内容。**

虽然今天 React 非常流行的最重要原因之一就是 React 的高性能，但我并没有把它归类为 React 的最好的一点。我认为 React 是一个游戏规则改变者，因为它在开发人员和浏览器之间创建了一种通用语言，允许开发人员以声明方式描述UI并管理其状态（state）上的操作，而不是对 DOM 元素的操作。它只是用户界面“结果”的语言。开发人员只是根据“最终”状态（如函数）来描述接口，而不是采用步骤来描述接口上的操作。当更新该状态时，React会根据它来更新 DOM 中的 UI（高效更新）。

如果有人要求你给出一个 React 为什么值得学习的原因，就是它是一个基于结果的 UI 语言。我们可以将这种语言称为 **React语言** 。

### 二、React 语言

假设我们有一个像这样的 `todos` 列表：

```js
const todos: [
  { body: 'Learn React Fundamentals', done: true },
  { body: 'Build a TODOs App', done: false },
  { body: 'Build a Game', done: false },
];
```

此 `todos` 数组是 UI 的起始状态。你需要构建一个 UI 来显示和管理。在这个页面上有三个操作，风别是一个添加新 `todo` 的表单  ，一个将 `todo` 标记为已完成，以及删除所有已完成的 `todo` 。

![todos-ui](/Users/lilunahaijiao/Desktop/todos-ui.png)

这些操作中的每一个都将要求应用程序执行DOM操作以创建，插入，更新或删除 DOM 节点。使用 React ，你不必担心所有这些 DOM 操作。你不必担心何时需要发生或如何有效地执行它们。你只需将 `todos` 数组置于应用程序的 `state` 中，然后使用 React 语言命令 React 在 UI 中以某种方式显示该状态：

```js
<header>TODO List</header>

<ul>
  {todos.map(todo =>
    <li>{todo.body}</li>
  )}
</ul>

// Other form elements...
```

之后，你可以专注于对该`todos` 数组进行数据操作！你可以添加，删除和更新该数组，React 会将你对该对象所做的更改渲染到浏览器上。

这种基于最终状态建模 UI 的心理模型更易于理解和使用，尤其是当视图具有大量数据转换时。例如，考虑一下可以告诉你有多少朋友在线的视图。该视图的 `state` 只是目前有多少朋友在线的一个数字。它并不关心刚才三个朋友上网，然后其中一个断线，然后两个加入。它只知道在这个时刻，有四个朋友在线。



### 三、树协调算法

在 React 之前，当我们需要使用浏览器的API（DOM API）时，我们尽可能避免遍历 DOM 树，那是因为 DOM 上的任何操作都在同一个线程中完成，该线程负责浏览器中发生的所有事情，包括对用户事件的反应：如打字，滚动，调整大小等。

对 DOM 的任何昂贵的操作都可能给用户带来缓慢的操作体验。非常重要的是，你的应用程序执行最小的操作时，应尽可能地批量处理。React 就提出了一个独特的概念来帮助我们做到这一点！

当我们告诉 React 在浏览器中渲染元素树时，它首先生成该树的虚拟表示并将其保存在内存中以供日后使用。然后它将继续执行DOM操作，使树显示在浏览器中。

当我们告诉 React 更新之前渲染的元素树时，它会生成树的新的虚拟表示。现在React在内存中有2个版本的树！

要在浏览器中呈现更新的树，React 不会丢弃已呈现的内容。相反，它将比较它在内存中的2个虚拟版本，计算它们之间的差异，找出主树中需要更新的子树，并且只在浏览器中更新这些子树。

这个过程就是所谓的树协调算法，它是 React 渲染浏览器 DOM 树的一种非常有效的方法。

除了基于声明结果的语言和有效的树协调之外，以下是我认为React获得其广泛流行的其他一些原因：

- 使用 DOM API 很难。React 使开发人员能够使用比真实浏览器更友好的**“虚拟”浏览**器。React将代理你与DOM进行通信。
- React 经常被赋予 **Just JavaScript** 标签。这意味着它有一个非常小的API可供学习，之后你的 JavaScript 技能使你成为更好的 React 开发人员。这比具有更大 API 的库更具优势。此外，React API 主要是函数（如果需要，还可以选择类）。当你听到 UI 视图是你的数据的函数时，在 React 中确实如此。
- 学习 React也为 iOS 和 Android 移动应用程序带来了巨大的回报。**React Native **允许你使用 React 技能来构建本机移动应用程序。你甚至可以在 Web ，iOS 和 Android 应用程序之间共享一些逻辑。
- Facebook 的 React 团队测试了在 **facebook.com** 上引入 React 的所有改进和新功能，这增加了社区对库的信任。React版本中很少见到大而严重的错误，因为它们只有在 Facebook 进行彻底的生产测试后才能发布。React 还支持其他频繁使用的 Web 应用程序，如 Netflix，Twitter，Airbnb 等等。

### 四、React 示例

为了看到树协调算法的实际好处及其所带来的巨大差异，让我们看一个只关注该概念的简单示例。让我们生成并更新两次HTML元素树，一次使用本机Web API，然后使用React API（及其协调工作）。为了简化这个例子，我不会使用组件或 JSX（与React一起使用的 JavaScript **扩展**）。我还将在 JavaScript **间隔**计时器内执行更新操作。这不是我们编写React应用程序的方式，而是让我们一次关注一个概念。

在此会话中，使用2种方法将简单的HTML元素呈现给显示：

- 方法1：**直接使用 Web DOM API**

  ```js
  document.getElementById('mountNode').innerHTML = `
    <div>
      Hello HTML
    </div>
  `;
  ```

  

- 方法2：**使用 React API** 

  ```js
  ReactDOM.render(
    React.createElement(
      'div',
      null,
      'Hello React',
    ),
    document.getElementById('mountNode2'),
  );
  ```

该 `ReactDOM.render` 方法和 `React.createElement` 方法是 React 应用程序中的核心 API 方法。事实上，如果不使用这两种方法，React Web 应用程序就不可能存在。简要介绍一下： 

#### ReactDOM.render

是 React 应用程序渲染到浏览器 DOM 的**入口点**。它有两个参数：

- 第一个参数是向浏览器呈现的内容。这是一个 React 元素。
- 第二个参数是 React 渲染在浏览器上的位置。这必须是存在于静态的 HTML 中的有效 DOM 节点。上面的示例使用了一个特殊 `mountNode2`元素，该元素存在于playground 的显示区域中（第一个 `mountNode` 用于本机版本）。

**React元素究竟是什么？它是用来描述 Actual DOM 元素的 Virtual 元素。也就是 `React.createElement` API方法返回的内容。**

#### React.createElement

在 React 中，我们不使用字符串来表示 DOM 元素（如上面的 DOM 示例中），而是使用对方法的调用来表示带有**对象的** DOM 元素 `React.createElement` 。这些对象称为 React 元素。

该 `React.createElement` 函数有很多参数：

- 第一个参数是要表示的DOM元素的 HTML 标记，`div` 在此示例中。
- 第二个参数为任何属性（如`id`，`href`，`title`，等），如果没有属性，可以使用 `null` 。
- 第三个参数是 DOM 元素的内容。我们在那里放了一个 `Hello React` 字符串。可选的第三个参数以及它后面的所有可选参数，形成渲染元素的**子**列表。元素可以包含0个或更多子元素。

 **`React.createElement` 也可用于从 React 组件创建元素。**

React 元素在内存中创建。为了实际在真实 DOM 中显示一个 React 元素，我们使用 `ReactDOM.render` 来实现将 React 元素的状态映射到浏览器中的真实 DOM 树中。



#### 3. 嵌套 React 元素

我们有两个节点：一个用 DOM API 直接控制，另一个用 React API 控制。

我们在浏览器中构建这两个节点的方式之间唯一的区别是，在 HTML 版本中，我们使用字符串来表示 DOM 树，而在 React 版本中，我们使用纯 JavaScript 调用并使用对象表示 DOM 树。

无论 HTML UI 有多复杂，使用 React 时，每个 HTML 元素都将用 React 元素表示。

##### 示例一：添加多个 HTML 元素，添加一个文本框来读取用户的输入 

对于 HTML 版本，你可以直接在模板中注入新元素的标记：

```js
document.getElementById('mountNode').innerHTML = `
  <div>
    Hello HTML
    <input />
  </div>
`;
```

而对 React 执行相同操作，就需要在 `React.createElement` 上面的第三个参数之后添加更多参数。为了匹配到在原生 DOM 示例中的内容，我们可以添加第四个参数，这是另一个 `React.createElement` 呈现 `input` 元素的调用：

```js
ReactDOM.render(
  React.createElement(
    "div",
    null,
    "Hello React ",
    React.createElement("input")
  ),
  document.getElementById('mountNode2'),
);
```

##### 示例二：渲染当前时间

可以使用它 `new Date().toLocaleTimeString()` 来显示简单的时间字符串，并把它放在一个 `pre` 标签中。

原生 DOM 版本执行的操作：

```js
document.getElementById('mountNode1').innerHTML = `
  <div>
    Hello HTML
    <input />
    <pre>${new Date().toLocaleTimeString()}</pre>
  </div>
`;
```

在 React 中，我们需要在顶层 `div` 元素中添加第五个参数。并且，这个新的第五个参数是另一个 `React.createElement` 调用，创建一个 `pre` 标签，并且内容为 `Date().toLocaleTimeString()` ：

```js
ReactDOM.render(
  React.createElement(
    'div',
    null,
    'Hello React ',
    React.createElement('input'),
    React.createElement(
      'pre',
      null,
      new Date().toLocaleTimeString()
    )
  ),
  document.getElementById('mountNode2')
);
```



因此，你可能认为使用 React 比使用简单熟悉的原生方式要困难得多。**那么为什么我们要放弃熟悉的 HTML 并且必须学习 React  API 来编写可以用 HTML 编写实现的内容喃？**

答案不在于渲染第一个 HTML 视图，这是在于我们如何**更新**已渲染的 DOM 视图。



#### 4. 更新React元素

我们对 DOM 树进行更新操作。例如：简单地让时间字符串**每秒**更新。

我们可以使用 `setInterval` Web 计时器 API 轻松地在浏览器中重复 JavaScript 函数调用。将两个版本的所有 DOM 操作放入一个函数中，命名它 `render` ，并在 `setInterval` 调用中使用它以使其每秒重复一次。

以下是此示例的完整代码：

```js
const render = () => {
  // HTML DOM
  document.getElementById('mountNode').innerHTML = `
    <div>
      Hello HTML
      <input />
      <pre>${new Date().toLocaleTimeString()}</pre>
    </div>
  `;
    
  // React DOM
  ReactDOM.render(
    React.createElement(
      'div',
      null,
      'Hello React',
      React.createElement('input', null),
      React.createElement('pre', null, new Date().toLocaleTimeString())
    ),
    document.getElementById('mountNode2')
  );
};

// 每秒更新一次
setInterval(render, 1000);
```

[点击查看实例](https://stackblitz.com/edit/react-txxnva?file=index.js)

请注意两个版本中的时间字符串如何每秒更新。我们现在正在更新 DOM 中的 UI 。

**这是React可能会让你大吃一惊的时刻。**如果你尝试在原生 DOM 版本的文本框中键入内容，则无法执行此操作。这是因为我们每秒都会抛弃原有的整个DOM节点并重新生成它。

但是，如果你尝试在 React 版本中的文本框中键入内容，却可以执行。

虽然整个 React 渲染代码都在计时器内，但 React只更改 `pre` 元素的内容而不是整个 DOM 树。这就是文本输入框没有重新生成的原因，我们可以输入它。

如果你检查 Chrome DevTools 元素面板中的两个DOM节点，你可以看到这两种方式更新 DOM 的不同。

- 原生 HTML 版本：`div#mountNode` 每秒重新生成其整个 DOM 树
- React 版本：`div#mountNode2` 容器中仅 `pre` 每秒重新生成

![更新DOM](https://jscomplete.com/images/reads/react/highlights.gif)

这是 React 的智能**差异**算法。它只在主 DOM 树中更新实际**需要**更新的内容，同时保持其他所有内容相同。这种差异化过程是可行的，因为它在内存中保留了 React 的虚拟 DOM 表示。无论 UI 视图需要重新生成多少次， React 将只向浏览器提供所需的更新部分。

这种方法不仅效率更高，而且消除了我们考虑更新 UI 的方式的复杂性。让 React 处理关于是否需要更新 DOM 的所有计算模块，使我们能够专注于思考我们的数据（`state` 状态）以及 UI 展示。

然后，我们根据需要管理数据状态的更新，而不必担心在浏览器的实际 UI 中渲染这些更新所需的步骤（因为我们知道 React 将完全执行此操作并且以更有效的方式执行！）





