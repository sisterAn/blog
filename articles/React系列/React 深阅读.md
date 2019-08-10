### 一、重新认识 React

React 官网上定义：

**React 是一个用于构建用户界面的 JavaScript 库。**

首先，让我们这个定义的两个不同部分：

#### React 是一个 JavaScript 库

这意味着它不完全是一个 **框架** 。它不是一个完整的解决方案，您经常需要使用更多的 React 库来形成任何解决方案。React 不对任何解决方案中的其他部分做任何假设。

框架是一个伟大的目标，特别是对于年轻的团队和初创公司。在使用框架时，已经为您做出了许多明智的设计决策，这为您提供了一条清晰的道路，可以专注于编写良好的应用程序级逻辑。但是，框架存在一些缺点。对于从事大型代码库开发工作的经验丰富的开发人员来说，这些缺点有时会极具破坏性的。

框架并不灵活，尽管有些人声称。框架通常希望您以某种方式编码所有内容。如果你试图偏离这种方式，框架经常会为此与你发生冲突。框架通常也很大并且功能齐全。如果你只需要使用它们中的一小部分，你必须要引入整个框架。不可否认，今天这一点正在改变，但仍然不理想。一些框架正在模块化，我认为这很棒，但我是纯 Unix 哲学的忠实粉丝：

> 编写做一件事并做得好的程序。编写程序以协同工作。 - 道格麦克罗伊

React遵循 Unix 哲学，因为它是一个小型库，专注于一件事并且非常好地完成这件事。“一件事” 是React定义的第二部分：**构建用户界面**。

#### React 用于构建用户界面

用户界面（UI）是展现在用户面前，用于与机器交互的媒介。用户界面无处不在，从微波炉上的简单按钮到航天飞机的仪表板。如果我们尝试连接的设备可以理解 JavaScript ，我们可以使用 React 来描述它的 UI 。由于Web浏览器理解 JavaScript ，我们可以使用 React 来描述 Web UI 。

我喜欢在这里使用 **describe** 这个词，因为这是**我们**基本上用React做的。我们只是告诉它我们想要什么！然后，React将代表我们在Web浏览器中构建实际的UI。如果没有React或类似的库，我们需要使用原生 Web API 和 JavaScript 手动构建 UI，这并不容易。

当你听到 **React 是声明** 的陈述时，这正是它的含义。我们用 React 描述 UI 并告诉它我们想要什么（而不是如何做）。React将负责“how”并将我们的声明性描述（我们用React语言编写）转换为浏览器中的实际UI。React 与 HTML 本身共享这种简单的声明能力，但是使用React，我们可以声明代表动态数据的HTML UI，而不仅仅是静态数据。

当React发布时，有很多关于它性能的质疑，因为它引入了一个虚拟 DOM 的聪明想法，可以用来协调实际的DOM（我们将在下一节讨论）。

**DOM是文档对象模型（Document Object Model）。它是HTML（和XML）文档的浏览器编程接口，将它们视为树结构。DOM API可用于更改文档结构，样式和内容。**

虽然React的性能仍然是今天 React 非常流行的最重要原因之一，但我并没有把它归类为 React 的最好的一点。我认为 React 是一个游戏规则改变者，因为它在开发人员和浏览器之间创建了一种通用语言，允许开发人员以声明方式描述UI并管理其状态（state）上的操作，而不是对 DOM 元素的操作。它只是用户界面“结果”的语言。开发人员只是根据“最终”状态（如函数）来描述接口，而不是采用步骤来描述接口上的操作。当更新该状态时，React会根据它来更新 DOM 中的 UI（高效更新）。

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

此 `todos` 数组是 UI 的起始状态。您需要构建一个 UI 来显示和管理。用户界面将有一个表单来添加新的 `todo` ，一种将 `todo` 标记为完整的方法，以及一种删除所有已完成的 `todo` 的方法。

这些操作中的每一个都将要求应用程序执行DOM操作以创建，插入，更新或删除 DOM 节点。使用 React ，您不必担心所有这些 DOM 操作。您不必担心何时需要发生或如何有效地执行它们。您只需将 `todos` 数组置于应用程序的 `state` 中，然后使用 React 语言命令 React 在 UI 中以某种方式显示该状态：

```js
<header>TODO List</header>

<ul>
  {todos.map(todo =>
    <li>{todo.body}</li>
  )}
</ul>

// Other form elements...
```

不要担心语法，但如果您想知道发生了什么，我们只需将 JavaScript 对象数组映射到 React 元素数组中，就会发现更多。

之后，您可以专注于对该`todos` 数组进行数据操作！您可以添加，删除和更新该数组，React 会将你对该对象所做的更改渲染到浏览器上。

这种基于最终状态建模 UI 的心理模型更易于理解和使用，尤其是当视图具有大量数据转换时。例如，考虑一下可以告诉您有多少朋友在线的视图。该视图的 `state` 只是目前有多少朋友在线的一个数字。它并不关心刚才三个朋友上网，然后其中一个断线，然后两个加入。它只知道在这个时刻，有四个朋友在线。



### 三、树协调算法

在 React 之前，当我们需要使用浏览器的API（DOM API）时，我们尽可能避免遍历 DOM 树，那是因为，DOM 上的任何操作都在同一个线程中完成，该线程负责浏览器中发生的所有事情，包括对用户事件的反应，如打字，滚动，调整大小等。

对 DOM 的任何昂贵的操作都可能给用户带来缓慢而笨拙的操作体验。非常重要的是，您的应用程序执行绝对最小的操作时，应尽可能地批量处理它们。React 就提出了一个独特的概念来帮助我们做到这一点！

当我们告诉 React 在浏览器中渲染元素树时，它首先生成该树的虚拟表示并将其保留在内存中以供日后使用。然后它将继续执行DOM操作，使树显示在浏览器中。

当我们告诉 React 更新它先前渲染的元素树时，它会生成更新树的新虚拟表示。现在React在内存中有2个版本的树！

要在浏览器中呈现更新的树，React 不会丢弃已呈现的内容。相反，它将比较它在内存中的2个虚拟版本，计算它们之间的差异，找出主树中需要更新的子树，并且只在浏览器中更新这些子树。

这个过程就是所谓的树协调算法，它是 React 操纵 浏览器 DOM 树的一种非常有效的方法。

除了基于声明结果的语言和有效的树协调之外，以下是我认为React获得其广泛流行的其他一些原因：

- 使用DOM API很难。React 使开发人员能够使用比真实浏览器更友好的**“虚拟”浏览**器。React将代理您与DOM进行通信。
- React 经常被赋予 **Just JavaScript** 标签。这意味着它有一个非常小的API可供学习，之后您的 JavaScript 技能使您成为更好的 React 开发人员。这比具有更大 API 的库更具优势。此外，React API 主要是函数（如果需要，还可以选择类）。当您听到 UI 视图是您的数据的函数时，在 React 中确实如此。
- 学习 React也为 iOS 和 Android 移动应用程序带来了巨大的回报。**React Native **允许您使用 React 技能来构建本机移动应用程序。您甚至可以在 Web ，iOS 和 Android 应用程序之间共享一些逻辑。
- Facebook 的 React 团队测试了在 **facebook.com** 上引入 React 的所有改进和新功能，这增加了社区对库的信任。React版本中很少见到大而严重的错误，因为它们只有在 Facebook 进行彻底的生产测试后才能发布。React 还支持其他频繁使用的 Web 应用程序，如 Netflix，Twitter，Airbnb 等等。

### 四、React 示例

为了看到树协调算法的实际好处及其所带来的巨大差异，让我们通过一个仅关注该概念的简单示例。让我们生成并更新两次HTML元素树，一次使用本机Web API，然后使用React API（及其协调工作）。为了简化这个例子，我不会使用组件或 JSX（与React一起使用的 JavaScript **扩展**）。我还将在 JavaScript **间隔**计时器内执行更新操作。这不是我们编写React应用程序的方式，而是让我们一次关注一个概念。

在此会话中，使用2种方法将简单的HTML元素呈现给显示：

- 方法1：**直接使用 Web DOM API**

  ```js
  document.getElementById('mountNode').innerHTML = `
    <div>
      Hello HTML
    </div>
  `;
  ```

  

- 方法2：**使用React API** 

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

- 第一个参数是向浏览器呈现的内容。是一个 React 元素。
- 第二个参数是 React 渲染在浏览器上的位置。这必须是存在于静态的 HTML 中的有效 DOM 节点。上面的示例使用了一个特殊 `mountNode2`元素，该元素存在于playground 的显示区域中（第一个 `mountNode` 用于本机版本）。

React元素究竟是什么？它是用来描述 Actual DOM 元素的 Virtual 元素。也就是 `React.createElement` API方法返回的内容。

#### React.createElement

在 React 中，我们不使用字符串来表示 DOM 元素（如上面的本机 DOM 示例中），而是使用对方法的调用来表示带有**对象的** DOM元素 `React.createElement` 。这些对象称为 React 元素。

该`React.createElement`函数有很多参数：

- 第一个参数是要表示的DOM元素的HTML“标记”，`div`在此示例中。
- 第二个参数为任何属性（如`id`，`href`，`title`，等），我们想要的DOM元素有。`div`我们使用的简单没有属性，所以我们`null`在那里使用。
- 第三个参数是DOM元素的内容。我们在那里放了一个“Hello React”字符串。可选的第三个参数以及它后面的所有可选参数，形成渲染元素的**子**列表。元素可以包含0个或更多子元素。

 **`React.createElement` 也可用于从React组件创建元素。**

React元素在内存中创建。为了实际在DOM中显示一个React元素，我们使用这个`ReactDOM.render`方法来做很多事情来找出将React元素的状态反映到浏览器中的实际DOM树中的最佳方法。



#### 3. 嵌套React元素

我们有两个节点：一个用DOM API直接控制，另一个用React API控制（反过来使用DOM API）。我们在浏览器中构建这两个节点的方式之间唯一的主要区别是，在HTML版本中，我们使用字符串来表示DOM树，而在React版本中，我们使用纯JavaScript调用并使用对象表示DOM树而不是一个字符串。

无论HTML UI有多复杂，使用React时，每个HTML元素都将用React元素表示。

让我们在这个简单的UI中添加更多HTML元素。让我们添加一个文本框来读取用户的输入。对于HTML版本，您可以直接在模板中注入新元素的标记：

```js
document.getElementById('mountNode').innerHTML = `
  <div>
    Hello HTML
    <input />
  </div>
`;
```

要对React执行相同操作，可以在`React.createElement`上面的第三个参数之后添加更多参数。为了匹配到目前为止在本机DOM示例中的内容，我们可以添加第四个参数，这是另一个`React.createElement`呈现`input`元素的调用：

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

让我们在两个版本中渲染当前时间。让我们把它放在一个`pre`元素中（只是为了在操场上给它一个等宽字体）。您可以使用它`new Date().toLocaleTimeString()`来显示简单的时间字符串。以下是您需要为本机DOM版本执行的操作：

```js
document.getElementById('mountNode1').innerHTML = `
  <div>
    Hello HTML
    <input />
    <pre>${new Date().toLocaleTimeString()}</p>
  </div>
`;
```

为了在React中做同样的事情，我们在顶层`div`元素中添加第五个参数。这个新的第五个参数是另一个`React.createElement`调用，这次使用`pre`带有`new Date().toLocaleTimeString()`内容字符串的标记：

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

正如你现在想的那样，使用React比简单熟悉的原生方式要困难得多。**什么是React做得很好，值得放弃熟悉的HTML并且必须学习一个新的API来编写可以用HTML编写的内容？**

答案不是渲染第一个HTML视图。这是关于我们需要做什么来**更新** DOM中的任何现有视图。



#### 4. 更新React元素

让我们对目前为止的DOM树进行更新操作。让我们简单地让时间字符串**每秒**都**打勾**。

我们可以使用`setInterval`Web计时器API 轻松地在浏览器中重复JavaScript函数调用。让我们将两个版本的所有DOM操作放入一个函数中，命名它`render`，并在`setInterval`调用中使用它以使其每秒重复一次。

以下是此示例的完整代码：

```js
document.getElementById('mountNode').innerHTML = `
    <div>
      Hello HTML
      <input />
      <pre>${new Date().toLocaleTimeString()}</pre>
    </div>
  `;

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

setInterval(render, 1000);
```

请注意两个版本中的时间字符串如何每秒滴答。我们现在正在更新DOM中的UI。

**这是React可能会让你大吃一惊的时刻。**如果您尝试在本机DOM版本的文本框中键入内容，则无法执行此操作。这是非常期待的，因为我们基本上抛弃了每个tick上的整个DOM节点并重新生成它。但是，如果您尝试在使用React呈现的文本框中键入内容，您当然可以这样做！

虽然整个React渲染代码都在滴答计时器内，但React只更改`pre`元素的内容而不是整个DOM树。这就是文本输入框没有重新生成的原因，我们可以输入它。

如果您检查Chrome DevTools元素面板中的两个DOM节点，您可以看到我们以可视方式更新DOM的不同方式。Chrome DevTools元素面板突出显示所有更新的DOM元素。您将看到本机HTML版本如何`div#mountNode`使用每个刻度重新生成其整个容器，而React巧妙地仅`pre`在其`div#mountNode2`容器中重新生成标记。

![更新DOM](https://jscomplete.com/images/reads/react/highlights.gif)

这是React的智能**差异**算法。它只在主DOM树中更新实际**需要**更新的内容，同时保持其他所有内容相同。这种差异化过程是可能的，因为它在内存中保留了React的虚拟DOM表示。无论UI视图需要重新生成多少次，React将只向浏览器提供所需的“部分”更新。

这种方法不仅效率更高，而且在我们**考虑**更新UI 的方式上也消除了很大的复杂性。让React做关于我们是否应该更新DOM的所有计算使我们能够专注于思考我们的数据（状态）以及为其描述UI的方式。然后，我们根据需要管理数据状态的更新，而不必担心在浏览器的实际UI中反映这些更新所需的步骤（因为我们知道React将完全执行此操作并且它将以有效的方式执行此操作！）

### 五、React 就是 组件

在React中，我们使用可重用，可组合和有状态的组件来描述UI。

我们定义小组件，然后将它们组合在一起形成更大的组件。即使在不同的项目中，所有小型或大型组件都可以重复使用。

您可以将组件视为简单的函数（使用任何编程语言）。我们用一些输入调用函数，它们给我们一些输出。我们可以根据需要重用函数，并从较小的函数中创建更大的函数。

反应组件完全相同; 他们的输入是一组“道具”，他们的输出是UI的描述。我们可以在多个UI中重用单个组件，组件可以包含其他组件。React组件的基本形式实际上是一个普通的JavaScript函数。

一些React组件是纯粹的，但您也可以在组件中引入副作用。例如，组件在浏览器中安装时可能会更改网页的HTML“标题”，或者可能会将浏览器视图滚动到某个位置。

最重要的是，React组件可以拥有一个私有状态来保存可能在组件生命周期内发生变化的数据。这个私有状态是驱动组件输出的输入的隐式部分，实际上是React的名字！

> 为什么React无论如何命名为“React”？
>
> 当React组件的状态（它是其输入的一部分）发生更改时，它所代表的UI（其输出）也会发生更改。UI描述中的这种变化必须反映在我们正在使用的设备中。在浏览器中，我们需要更新DOM树。在React应用程序中，我们不会手动执行此操作。React将简单地对状态更改**做出反应**，并在需要时自动（并有效）更新DOM。



### 六、函数组件

React组件 - 最简单的形式 - 是一个简单的JavaScript函数：

```js
// jsdrops.com/bx1

function Button (props) {
  // Returns a DOM/React element here. For example:
  return <button type="submit">{props.label}</button>;
}

// To render a Button element in the browser
ReactDOM.render(<Button label="Save" />, mountNode);
```

请注意我是如何在`Button`上面函数的返回输出中编写看起来像HTML的内容。这既不是HTML也不是JavaScript，甚至不是React。这是**JSX**。它是JavaScript 的**扩展**，允许我们以类似HTML的语法编写函数调用。

继续尝试返回`Button`函数内的任何其他HTML元素，看看它们是如何被支持的（例如，返回`input`元素或`textarea`元素）。

#### 1. JSX 不是 HTML

一个最简单 React 组件（函数组件）：

```JS
// 返回一个 React 组件
function Hello (props) {
  return <div>Hello {this.props.name}</div>;;
}

// 在浏览器中渲染一个 Hello 元素
ReactDOM.render(<Hello name="Bottle" />, document.getElementById('root'));
```

我们在 `ReactDOM.render` 中渲染 `Button` 组件，使用了类似 HTML 的样式，但它既不是 HTML，也不是 JS，甚至不是 React。这就是 JSX，它是 JavaScript 的扩展，允许我们以类似于 HTML 的函数语法编写函数调用。

每个 JSX 元素只是调用 `React.createElement(component, props, ...children)` 的语法糖。因此，使用 JSX 可以完成的任何事情都可以通过纯 JS 完成。

上例就可以可以编写为不使用 JSX 的代码：

```js
ReactDOM.render(
  React.createElement(
    Hello,
    {name: 'Bottle'},
    React.createElement(
      'div',
       null,
       `Hello ${this.props.name}`,
    )
  ),
  document.getElementById('root'),
); // 在浏览器中渲染一个简单的 div 元素，显示 Hello Bottle
```

`React.render` 与 `React.createElement` 是 React 最核心的 API 方法，每一个 React 项目都必须要引入这两个API。

浏览器不识别 JSX。我们在浏览器中运行 JSX，会报错：

![JSX](/Users/lilunahaijiao/Desktop/屏幕快照 2019-08-03 上午11.51.02.png)

所以，在项目中运用 JSX，我们需要使用像 Babel 或 TypeScript 这样的转换器。例如，当 我们使用 create-react-app 创建项目时，就会在内部使用 Babel 来转换项目中的 JSX。

**JSX 基本上是一种折中**，使我们能够使用与 HTML 非常相似的语法，使用编译器将其转换为`React.createElement`调用，而不是直接使用 `React.createElement` 语法创建 React 组件。

React 组件是一个返回 React 元素的 JS 函数。当使用 JSX 时，`<tag></tag>`语法会被转化为`React.createElement("tag")`。在创建 React 组件时应该牢记这一点。我们不是在写 HTML，而实在使用 JS 扩展来返回创建React元素（实际上是 JS 对象）的函数调用。

因此，**JSX 允许我们类HTML的语法来表示React树，浏览器和React均不需要识别它，只有编译器才有。我们发送给浏览器的是无JSX代码。**

#### 2. 名称必须以大写字母开头



#### 3. 第一个参数是“道具”的对象

### 七、class 组件



### 八、函数与类



### 九、组件的优点



### 十、Hooks



### 十一、多组件



### 十二、组件可重用



### 十三、接受用户的输入



### 十四、管理副作用



### 十五、下一步





