### 五、React 核心是组件

在 React 中，我们使用组件（有状态、可组合、可重用）来描述 UI 。
在任何编程语言中，你都可以将组件视为简单的函数。

React 组件也一样， 它的输入是  props，输出是关于 UI 的描述。我们可以在多个 UI 中重用单个组件，组件也可以包含其他组件。React 组件的本质上就是一个普通的 JavaScript 函数。
尽管一些 React 组件是纯组件，但也可以在组件中引入副作用。例如，组件在浏览器中渲染时可能会更改网页的标题，或者可能会将浏览器视图滚动到某个位置。
最重要的是，React 组件可以拥有一个私有状态来保存在组件生命周期内可能发生变化的数据。这个私有状态驱动组件输出到原生 DOM 中！

> 为什么将 React 称为响应式设计？
>
> 当 React 组件的状态（它是其输入的一部分）发生更改时，它所代表的 UI （其输出）也会发生更改。UI 描述中的这种变化必须反映在我们正在使用的设备中。在浏览器中，我们需要更新 DOM 树。在 React 应用程序中，我们不会手动执行此操作。 `state` 更新时，React 自动响应，并在需要时自动（并有效）更新到 DOM 上。



### 六、函数组件

React 组件，最简单的形式就是  JavaScript 函数：

```js
function Button (props) {
  // 在这里返回一个DOM / React元素。例如：
  return <button type="submit">{props.label}</button>;
}

// 在浏览器中渲染一个 Button 元素 
ReactDOM.render(<Button label="Save" />, mountNode);
```

我们在 `ReactDOM.render` 中渲染 `Button` 组件，使用了类似 HTML 的样式，但它既不是 HTML，也不是 JS，甚至不是 React。这就是 JSX ，它是 JavaScript 的扩展，允许我们以类似于 HTML 的函数语法编写函数调用。

你可以尝试在 `Button` 函数内返回其他 HTML 元素，看看它们是如何被支持的（例如，返回 `input` 元素或 `textarea` 元素）。



#### 1. JSX 不是 HTML

每个 JSX 元素只是调用 `React.createElement(component, props, ...children)` 的语法糖。因此，使用 JSX 可以完成的任何事情都可以通过纯 JS 完成。

上例就可以编写为不使用 JSX 的代码：

```js
ReactDOM.render(
  React.createElement(
    Hello,
    {name: 'Button'},
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

![JSX](https://user-images.githubusercontent.com/19721451/62879392-991e0380-bd5d-11e9-94a8-b9ecab779928.png)

所以，在项目中运用 JSX，我们需要使用像 Babel 或 TypeScript 这样的转换器。例如，当 我们使用 create-react-app 创建项目时，就会在内部使用 Babel 来转换项目中的 JSX。

**JSX 基本上是一种折中**，使我们能够使用与 HTML 非常相似的语法，使用编译器将其转换为 `React.createElement` 调用，而不是直接使用 `React.createElement` 语法创建 React 组件。

React 组件是一个返回 React 元素的 JS 函数。当使用 JSX 时，`<tag></tag>`语法会被转化为 `React.createElement("tag")` 。在创建 React 组件时应该牢记这一点。我们不是在写 HTML，而实在使用 JS 扩展来创建 React 元素（实际上是 JS 对象）的函数调用。

因此，**JSX 允许我们类 HTML 的语法来表示 React 树，浏览器和 React 均不需要识别它，只有编译器才有。我们发送给浏览器的是无 JSX 代码。**



#### 2. 命名必须以大写字母开头

请注意我们在上面例子中将组件命名为 `Button`。第一个字母是大写字母，这是一个规定，因为我们在处理混合的 HTML 元素和 React 元素时，JSX 编译器（如 Babel ）会将所有以小写字母开头的名称视为 HTML 元素。

- HTML 元素作为字符串传递给 `React.createElement` 调用
- React 元素需要作为变量传递

```js
<button></button> // React.createElement("button", null)
<Button></Button> // React.createElement(Button, null)
```

再看一个例子：

```js
function button () {
  return <div>My Fancy Button</div>;
};

// 将会渲染一个 HTML button 组件
// 忽略 React 组件 button
ReactDOM.render(<button />, mountNode);
```



#### 3. 第一个参数是 `props` 的对象

就像可以为 HTML 元素传递 `id` 或 `title` 等属性一样，React 元素在渲染时也可以接收属性列表。例如，上面的 `Button` 元素就接受了 一个 `label` 属性。在 React 中，React 元素接收的属性列表称为 `props` 。

使用函数组件时，你不必将包含属性列表的对象命名为 `props`，但这是标准做法。但当我们使用类组件时，属性列表始终命名为 `props`。

**请注意，`props` 是可选的。有些组件可以没有 `props`。但是，组件必须有返回值。React 组件不能返回 `undefined`（显式或隐式）。它必须返回一个值。它可以返回 `null` 以使渲染器忽略其输出。**

每当我使用 `props`（或 `state`）时，我喜欢使用对象解构。例如，`Button  `组件函数可以使用 `props` 解构写法：

```js
const Button = ({ label }) => (
  <button type="submit">{label}</button>
);
```

这种方法有许多好处，但最重要的是看上去方便，并确保组件不会收到任何其他不需要的额外 `props`。

注意我这里使用的是 **箭头函数** 而不是常规**函数**。这只是我个人的一种风格**偏好**。有些人喜欢常规函数，这没有任何问题。我认为重要的是要与你选择的风格**保持一致**。



#### 4. JSX 中的表达式

你可以在 JSX 中的任何位置使用一对大括号来包含 JavaScript 表达式：

```js
const RandomValue = () => (
  <div>
    { Math.floor(Math.random() * 100) }
  </div>
);

ReactDOM.render(<RandomValue />, mountNode);
```

请注意，**只有表达式** 可以包含在 `{}` 内。

例如，你不能包含常规 ` if ` 语句，但三元表达式是可以的。任何有 **返回值的** 都是可以。

你可以在函数中放入任何代码，使它返回一些值，并在大括号内调用该函数。但是，尽量不要在  `{}` 内进行复杂的逻辑操作。

JavaScript 变量也是表达式，因此当组件收到 `props` 时，你可以在 `{}` 使用 `props`。这就是我们为什么能在 `Button` 函数组件中使用  `{label}` 的原因。

JavaScript 对象也是表达式。我们使用大括号内的 JavaScript 对象,这使得它看起来像双大括号：`{{a:42}}`。但这并不是一个不同的语法，它仅仅表示在常规 JSX 括号内，使用对象而已。

例如，在这些 `{}` 中使用对象的一个用例是将 CSS 样式对象传递给 `style` ：

```js
const ErrorDisplay = ({ message }) => (
  <div style={ { color:'red', backgroundColor:'yellow' } }>
    {message}
  </div>
);

ReactDOM.render(
  <ErrorDisplay
    message="These aren't the droids you're looking for"
  />,
  mountNode
);
```

`style`上面的属性是一个特殊的属性。React 将这些样式对象转换为内联 CSS 样式属性。当然，这不是设置 React 组件样式的最佳方法，但在条件样式中，使用它非常方便。例如，随机将输出的文本设为绿色或红色：

```js
class ConditionalStyle extends React.Component {
  render() {
    return (
      <div style={{ color: Math.random() < 0.5 ? 'green': 'red' }}>
        How do you like this?
      </div>
    );
  }
}

ReactDOM.render(
  <ConditionalStyle />,
  mountNode,
);
```

这比有条件地使用类名更容易使用。 



#### 5. JSX不是模板语言

一些处理 HTML 的库为它提供了模板语言。使用具有循环和条件的"增强"HTML 语法编写动态视图。然后，这些库使用 JavaScript 将模板转换为 DOM 操作。可以在浏览器中使用 DOM 操作来显示增强的 HTML 描述的 DOM 树。

React取消了那一步。我们不会使用 React 应用程序向浏览器发送模板。我们向它发送了一个用 React API 描述的对象树。React 使用这些对象生成显示所需 DOM 树的操作。

**使用 HTML 模板时，库会将你的应用程序解析为字符串，React 应用程序被解析为对象树。**

虽然 JSX 可能看起来像模板语言，但实际上并非如此。它只是一个JavaScript扩展，它允许我们用一个看起来像HTML 模板的语法来表示React的对象树。浏览器根本不需要处理 JSX ，React 也不必处理它！只有编译器才有。我们发送给浏览器的是无模板和无 JSX 代码。

例如，对于`todos`我们上面看到的数组，如果我们要使用模板语言在UI中显示该数组，我们需要执行以下操作：

```js
<ul>
  <% FOR each todo in the list of todos %>
    <li><%= todo.body %></li>
  <% END FOR %>
</ul>
```

**这`<% %>`是表示动态增强部分的一种语法。你可能还会看到`{{ }}`语法。某些模板语言使用特殊属性来增强逻辑。一些模板语言使用空格缩进（off-side rule）。**

当 `todos` 数组发生更改时（我们需要使用模板语言更新 DOM 中呈现的内容），我们必须重新呈现该模板或计算DOM树中我们需要反映 `todos` 数组中更改的位置。

在 React 应用程序中，根本没有模板语言。相反，我们使用 JSX ：

```js
<ul>
  {todos.map(todo =>
    <li>{todo.body}</li>
  )}
</ul>
```

在浏览器中使用之前，它被转换为：

```js
React.createElement(
  "ul",
  null,
  todos.map(todo =>
    React.createElement("li", null, todo.body)
  ),
);
```

React 获取这个对象树并将其转换为 DOM 元素树。从我们的角度来看，我们已经完成了这棵树。我们不管理任何行动。我们只管理 `todos` 数组本身的操作。



### 七、class 组件

React 也支持通过 JavaScript  `class` 语法创建组件。这是使用 `class` 编写的相同 `Button` 组件示例：

```js
class Button extends React.Component {
  render() {
    return (
      <button>{this.props.label}</button>
    );
  }
}

// 相同的语法使用它 
ReactDOM.render(<Button label="Save" />, mountNode);
```

在此语法中，你定义了 `Button` 继承自 `React.Component` ，它是 React 顶级 API 中的主要类之一。基于类的 React 组件必须至少定义一个名为的实例方法 `render` 。此 `render` 方法返回表示从组件实例化的对象的输出的元素。每次我们使用 `Button` 组件（通过渲染 `<Button … />`）时，React 将从这个基于类的组件中**实例化**一个对象，并使用该对象来创建一个 DOM 元素。它还会将DOM 呈现的元素与它从类创建的实例相关联。

注意我们在渲染的 JSX 中使用 `this.props.label` 的方式 ，每个组件有 `props` 属性，在组件实例化时，它包含传递给该组件元素的参数。与函数组件不同的是，`class` 组件中的 `render` 函数不接收任何参数。



### 八、函数与类

在 React 中使用函数组件是受限的。因为函数组件没有 `state` 状态。但在 React v16.8 引入 **Hooks** 之后就变得不同了，它能让组件在不使用 `class` 的情况下使用 `state` 以及其他的 React 特性，

我相信新的 API 会慢慢取代旧的 API ，但这并不是我想鼓励你使用它的唯一原因。

我在大型应用程序中使用了这两个 API ，我可以告诉你，新 API 比旧 API 更优越的方面有很多，其中我认为这些是最重要的：

- 你不必使用 `class` 及其 `state`。你仅需要使用在每个渲染上刷新的简单函数。`state` 被明确声明，没有任何隐藏。所有这些基本上意味着你将在代码中遇到更少的惊喜。
- 你可以将相关的 `state` 逻辑分组，并将其分为独立的可组合和可共享单元。这使得我们更容易将复杂组件分解为更小的部件。它还使测试组件更容易。
- 你可以以声明方式使用任何有状态逻辑，而无需在组件树中使用任何分层 “嵌套” 。

虽然在可预见的未来，基于 `class` 的组件将继续成为 React 的一部分，但作为 React 开发人员，我认为开始使用函数（和 Hook ），并专注于学习新 API 是有意义的。

#### 1. 组件与元素

你可能会在 React 指南和教程中找到 `component` 和 `element` 这两个词。我认为 React 学习者需要理解重要的区别。

- React Component 是一个模板，蓝图，全球定义。可以是函数或类（使用render方法）。
- React Element 是从组件返回的元素。它是与真实 DOM 相对应的虚拟节点。对于函数组件，此元素是函数返回的对象，对于类组件，元素是组件的 `render` 方法返回的对象。React 元素不是你在浏览器中看到的，它们只是内存中的对象，你无法改变它们。

React 在内部创建、更新和销毁对象，以找出需要渲染在浏览器的 DOM 元素树。使用类组件时，通常将其浏览器渲染的 DOM 元素称为组件实例。你可以渲染同一组件的许多实例。你不需要手动在类中创建实例，你只需要记住它就在 React 的内存中。对于函数组件，React 只使用函数的调用来确定要渲染的 DOM 实例。



### 九、组件的优点

术语 "组件" 被许多框架和库使用。我们可以使用 HTML5 功能(如自定义元素和 HTML 导入)编写原生 Web 组件。

组件，无论我们是在原生调用还是通过像 React 这样的库调用，都有许多优点。

首先，组件使你的代码更**易读**，更易于使用。考虑这个UI：

```js
<a href=”http://facebook.com”>
 <img src=”facebook.png” />
</a>
```

这个 UI 代表什么？如果你说 HTML ，你可以在这里快速解析并说 “这是一个可点击的图像”。如果我们要将这个 UI 转换成一个组件，我们可以命名它 `ClickableImage` ！

```js
<ClickableImage />
```

当事件变得更复杂时，HTML 变得更加困难，此时，组件允许我们使用我们熟悉的语言快速理解 UI 所代表的内容。这是一个更大的例子：

```js
<TweetBox>
  <TextAreaWithLimit limit="280" />
  <RemainingCharacters />
  <TweetButton />
</TweetBox>
```

在不查看实际 HTML 代码的情况下，我们确切地知道此 UI 表示的内容。此外，如果我们需要修改剩余字符部分的输出，我们必须知道确切要去哪里修改。

React 组件也可以在同一个应用程序中和多个应用程序中**重用**。例如，以下是 `ClickableImage` 组件：

```js
const ClickableImage = ({ href, src }) => {
 return (
   <a href={href}>
     <img src={src} />
   </a>
 );
};
```

拥有 `href` 和 `src` 属性的变量是使该组件可重用的原因。例如，要使用此组件，我们可以使用一组 `props` 渲染它：

```js
<ClickableImage href="http://google.com" src="google.png" />
```

我们可以通过使用不同的 `props` 重用它：

```js
<ClickableImage href="http://bing.com" src="bing.png" />
```

在函数式编程中，我们有纯函数的概念。 如果我们给纯函数相同的输入，我们将始终获得相同的输出。

如果 React 组件不依赖于其定义之外的任何内容，我们也可以将该组件标记为纯组件。纯组件在没有任何问题的情况下更有可能被重用。

我们可以将 HTML 元素视为浏览器中的内置组件。我们也可以使用自己的自定义组件来**组成**更大的组件。例如，让我们编写一个显示搜索引擎列表的组件。

```js
const SearchEngines = () => {
  return (
    <div className="search-engines">
      <ClickableImage href="http://google.com" src="google.png" />
      <ClickableImage href="http://bing.com" src="bing.png" />
    </div>
  );
};
```

注意我是如何使用 `ClickableImage` 组件来组成 `SearchEngines` 组件的！

我们还可以 `SearchEngines` 通过将数据提取到变量中并将其设计为使用该变量来使组件可重用。

例如，我们可以采用以下格式引入数据数组：

```js
const data = [
  { href: "http://google.com", src: "google.png" },
  { href: "http://bing.com", src: "bing.png" },
  { href: "http://yahoo.com", src: "yahoo.png" }
];
```

然后，为了 `<SearchEngines data={data} />` 能成功渲染，我们需要将 `data` 数组从对象列表映射到 `ClickableImage` 组件列表：

```js
const SearchEngines = ({ engines }) => {
  return (
    <List>
      {engines.map(engine => <ClickableImage {...engine} />)}
    </List>
  );
};

ReactDOM.render(
 <SearchEngines engines={data} />,
 document.getElementById("mountNode")
);
```

 `SearchEngines` 适用于我们提供给它的任何搜索引擎列表。



