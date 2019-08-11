### 五、React 就是 组件

在React中，我们使用可重用，可组合和有状态的组件来描述 UI 。

我们定义小组件，然后将它们组合在一起形成更大的组件。即使在不同的项目中，所有小型或大型组件都可以重复使用。

你可以将组件视为简单的函数（使用任何编程语言）。我们用一些输入调用函数，它们给我们一些输出。我们可以根据需要重用函数，并从较小的函数中创建更大的函数。

反应组件完全相同; 他们的输入是一组“道具”，他们的输出是UI的描述。我们可以在多个UI中重用单个组件，组件可以包含其他组件。React组件的基本形式实际上是一个普通的JavaScript函数。

一些React组件是纯粹的，但你也可以在组件中引入副作用。例如，组件在浏览器中安装时可能会更改网页的HTML“标题”，或者可能会将浏览器视图滚动到某个位置。

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

```js
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

![JSX](/Users/lilunahaijiao/Desktop/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-08-03%20%E4%B8%8A%E5%8D%8811.51.02.png)

所以，在项目中运用 JSX，我们需要使用像 Babel 或 TypeScript 这样的转换器。例如，当 我们使用 create-react-app 创建项目时，就会在内部使用 Babel 来转换项目中的 JSX。

**JSX 基本上是一种折中**，使我们能够使用与 HTML 非常相似的语法，使用编译器将其转换为`React.createElement`调用，而不是直接使用 `React.createElement` 语法创建 React 组件。

React 组件是一个返回 React 元素的 JS 函数。当使用 JSX 时，`<tag></tag>`语法会被转化为`React.createElement("tag")`。在创建 React 组件时应该牢记这一点。我们不是在写 HTML，而实在使用 JS 扩展来返回创建React元素（实际上是 JS 对象）的函数调用。

因此，**JSX 允许我们类HTML的语法来表示React树，浏览器和React均不需要识别它，只有编译器才有。我们发送给浏览器的是无JSX代码。**

#### 2. 名称必须以大写字母开头

请注意我如何将组件命名为 `Button`。第一个字母是大写字母实际上是一个要求，因为我们将处理混合的 HTML 元素和 React 元素。JSX 编译器（如 Babel ）会将所有以小写字母开头的名称视为 `HTML` 元素的名称。这很重要，因为 `HTML` 元素作为字符串传递给 `React.createElement` 调用，而 React 元素需要作为变量传递：

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

就像可以为HTML元素分配像`id`or 这样的属性一样`title`，React元素也可以在渲染时接收属性列表。`Button`上面的元素（[jsdrops.com/bx2](https://jsdrops.com/bx2)）收到了一个`label`属性。在React中，React元素接收的属性列表称为`props`。React函数组件接收此列表作为其第一个参数。该列表作为对象传递，其中键表示属性名称和表示分配给它们的值的值。

使用函数组件时，你不必将包含属性列表的对象命名为“props”，但这是标准做法。当使用类组件时，我们将在下面执行，相同的属性列表始终显示一个名为的特殊实例属性`props`。

**请注意，接收道具是可选的。有些组件没有任何道具。但是，组件的返回值不是可选的。React组件不能返回“undefined”（显式或隐式）。它必须返回一个值。它可以返回“null”以使渲染器忽略其输出。**

每当我使用组件道具（或状态，真的）时，我喜欢使用对象解构。例如，`Button`组件函数可以使用props destructuring这样编写：

```js
const Button = ({ label }) => (
  <button type="submit">{label}</button>
);
```

这种方法有许多好处，但最重要的是视觉检查组件中使用的道具，并确保组件不会收到任何不需要的额外道具。

注意我是如何使用**箭头函数**而不是常规**函数**。这对我个人来说只是一种风格**偏好**。有些人喜欢常规的功能风格，没有任何问题。我认为重要的是要与你选择的风格**保持一致**。我将在这里使用箭头函数，但不要将其解释为一个要求。



#### 4. JSX 中的表达式

你可以在JSX中的任何位置使用一对大括号包含JavaScript表达式：

```js
const RandomValue = () => (
  <div>
    { Math.floor(Math.random() * 100) }
  </div>
);

ReactDOM.render(<RandomValue />, mountNode);
```

请注意，**只有表达式**可以包含在这些大括号内。例如，你不能包含常规if语句，但三元表达式是可以的。任何**返回值的**东西都可以。你总是可以在函数中放入任何代码，使它返回一些东西，并在大括号内调用该函数。但是，请将放在这些花括号中的逻辑保持在最低限度。

JavaScript变量也是表达式，因此当组件收到道具列表时，你可以在大括号内使用这些道具。这就是我们`{label}`在`Button`示例中使用的方式。

JavaScript对象文字也是表达式。有时我们在大括号内使用JavaScript对象，这使它看起来像双花括号：`{{a:42}}`。这不是一个不同的语法; 它只是在常规JSX花括号中定义的对象文字。

例如，在这些花括号中使用对象文字的一个用例是将CSS样式对象传递给`style`React中的特殊属性：

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

`style`上面的属性是一个特殊的属性。我们使用一个对象作为其值，该对象定义样式，就好像我们通过JavaScript DOM API（camel-case属性名称，字符串值）设置它们一样。React将这些样式对象转换为内联CSS样式属性。这不是设置React组件样式的最佳方法，但我发现在将条件样式应用于元素时使用它非常方便。例如，这是一个组件，它将以大约一半的时间随机输出绿色或红色文本：

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

这种样式的逻辑就在组件中。我喜欢！这比有条件地使用类名更容易使用，然后跟踪该类名在全局CSS样式表中的作用。 



#### 5. JSX不是模板语言

一些处理HTML的库为它提供了一种模板语言。你使用具有循环和条件的“增强”HTML语法编写动态视图。然后，这些库将使用JavaScript将模板转换为DOM操作。然后可以在浏览器中使用DOM操作来显示增强型HTML描述的DOM树。

React取消了那一步。我们不会使用React应用程序向浏览器发送模板。我们向它发送了一个用React API描述的对象树。React使用这些对象生成显示所需DOM树所需的DOM操作。

**使用HTML模板时，库会将你的应用程序解析为字符串。React应用程序被解析为对象树。**

虽然JSX可能看起来像模板语言，但实际上并非如此。它只是一个JavaScript扩展，它允许我们用一个看起来像HTML模板的语法来表示React的对象树。浏览器根本不需要处理JSX，React也不必处理它！只有编译器才有。我们发送给浏览器的是无模板和无JSX代码。

例如，对于`todos`我们上面看到的数组，如果我们要使用模板语言在UI中显示该数组，我们需要执行以下操作：

```js
<ul>
  <% FOR each todo in the list of todos %>
    <li><%= todo.body %></li>
  <% END FOR %>
</ul>
```

**这`<% %>`是表示动态增强部分的一种语法。你可能还会看到`{{ }}`语法。某些模板语言使用特殊属性来增强逻辑。一些模板语言使用空格缩进（off-side rule）。**

当`todos`数组发生更改时（我们需要使用模板语言更新DOM中呈现的内容），我们必须重新呈现该模板或计算DOM树中我们需要反映`todos`数组中更改的位置。

在React应用程序中，根本没有模板语言。相反，我们使用JSX：

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

React获取这个对象树并将其转换为DOM元素树。从我们的角度来看，我们已经完成了这棵树。我们不管理任何行动。我们只管理`todos`数组本身的操作。

### 七、class 组件

React也支持通过JavaScript类语法创建组件。这是使用类语法编写的相同Button组件示例：

```js
class Button extends React.Component {
  render() {
    return (
      <button>{this.props.label}</button>
    );
  }
}

// Use it (same syntax)
ReactDOM.render(<Button label="Save" />, mountNode);
```

在此语法中，你定义了一个扩展的类`React.Component`，它是React顶级API中的主要类之一。基于类的React组件必须至少定义一个名为的实例方法`render`。此`render`方法返回表示从组件实例化的对象的输出的元素。每次我们使用`Button`基于类的组件（通过渲染a `<Button … />`）时，React 将从这个基于类的组件中**实例化**一个对象，并使用该对象的表示来创建一个DOM元素。它还会将DOM呈现的元素与它从类创建的实例相关联。

注意我们如何`this.props.label`在渲染的JSX中使用。每个组件都获得一个名为的特殊实例属性`props`，该属性包含在实例化时传递给该组件元素的所有值。与函数组件不同，`render`基于类的组件中的函数不接收任何参数。



### 八、函数与类

使用函数创建的组件在React中受到限制。使组件“有状态”的唯一方法是使用类语法。自从2019年初发布的React版本16.8开始发布“React Hooks”后，这已经发生了变化.React hooks发布引入了一个新的API，使函数组件有状态（并赋予它许多其他功能）。

使用这个新API，通常使用React完成的大部分工作都可以通过函数完成。只有高级和非常罕见的情况才需要基于类的语法。

我相信新的API会慢慢取代旧的API，但这并不是我想鼓励你使用它的唯一原因（如果可以的话）。

我在大型应用程序中使用了这两个API，我可以告诉你，由于很多原因，新API比旧API更优越，但我认为这些是最重要的：

- 你不必使用类“实例”及其隐式状态。你使用在每个渲染上刷新的简单函数。状态被明确声明，没有任何隐藏。所有这些基本上意味着你将在代码中遇到更少的惊喜。
- 你可以将相关的有状态逻辑分组，并将其分为独立的可组合和可共享单元。这使得更容易将复杂组件分解为更小的部件。它还使测试组件更容易。
- 你可以以声明方式使用任何有状态逻辑，而无需在组件树中使用任何分层“嵌套”。

虽然在可预见的未来，基于类的组件将继续成为React的一部分，但作为生态系统的新手，我认为让你纯粹只使用函数（和钩子）开始并专注于学习新API是有意义的（除非你必须使用已经使用类的代码库）。

##### 组件与元素

你可能会在React指南和教程中找到“组件”和“元素”这两个词。我认为React学习者需要理解重要的区别。

- React Component是一个模板。蓝图。全球定义。这可以是函数或类（使用render方法）。
- React元素是从组件返回的元素。它是一个虚拟描述组件所代表的DOM节点的对象。对于函数组件，此元素是函数返回的对象，对于类组件，元素是组件的render方法返回的对象。React元素不是你在浏览器中看到的。它们只是内存中的对象，你无法改变它们。

React在内部创建，更新和销毁对象，以找出需要呈现给浏览器的DOM元素树。使用类组件时，通常将其浏览器呈现的DOM元素称为组件实例。你可以渲染同一组件的许多实例。该实例是你在基于类的组件中使用的“this”关键字。你不需要手动从类创建实例。你只需要记住它就在React的内存中。对于函数组件，React只使用函数的调用来确定要呈现的DOM元素。



### 九、组件的优点

许多其他框架和库使用术语“组件”。我们甚至可以使用自定义元素和HTML导入等HTML5功能本地编写Web组件。

组件，无论我们是在本地使用它们还是通过像React这样的库，都有许多优点。

首先，组件使你的代码更**易读**，更易于使用。考虑这个UI：

```js
<a href=”http://facebook.com”>
 <img src=”facebook.png” />
</a>
```

这个UI代表什么？如果你说HTML，你可以在这里快速解析并说“这是一个可点击的图像。”如果我们要将这个UI转换成一个组件，我们可以命名它`ClickableImage`！

```js
<ClickableImage />
```

当事情变得更复杂时，HTML的解析变得更加困难，因此组件允许我们使用我们熟悉的语言快速理解UI所代表的内容。这是一个更大的例子：

```js
<TweetBox>
  <TextAreaWithLimit limit="280" />
  <RemainingCharacters />
  <TweetButton />
</TweetBox>
```

在不查看实际HTML代码的情况下，我们确切地知道此UI表示的内容。此外，如果我们需要修改剩余字符部分的输出，我们确切知道要去哪里。

React组件也可以在同一个应用程序中和多个应用程序中**重用**。例如，以下是`ClickableImage`组件的可能实现：

```js
const ClickableImage = ({ href, src }) => {
 return (
   <a href={href}>
     <img src={src} />
   </a>
 );
};
```

拥有`href`和`src`道具的变量是使该组件可重用的原因。例如，要使用此组件，我们可以使用一组props渲染它：

```js
<ClickableImage href="http://google.com" src="google.png" />
```

我们可以通过使用不同的道具集重用它：

```js
<ClickableImage href="http://bing.com" src="bing.png" />
```

在函数式编程中，我们有纯函数的概念。这些基本上不受任何外部国家的保护; 如果我们给他们相同的输入，我们将始终获得相同的输出。

如果React组件不依赖于（或修改）其定义之外的任何内容（例如，如果它不使用全局变量），我们也可以将该组件标记为纯。纯组件更有可能在没有任何问题的情况下重复使用。

我们创建表示视图的组件。因为`ReactDOM`，我们定义的React组件将代表HTML DOM节点。`ClickableImage`上面的组件由两个嵌套的HTML元素组成。

我们可以将HTML元素视为浏览器中的内置组件。我们也可以使用自己的自定义组件来**组成**更大的组件。例如，让我们编写一个显示搜索引擎列表的组件。

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

注意我是如何使用`ClickableImage`组件来组成`SearchEngines`组件的！

我们还可以`SearchEngines`通过将数据提取到变量中并将其设计为使用该变量来使组件可重用。

例如，我们可以采用以下格式引入数据数组：

```js
const data = [
  { href: "http://google.com", src: "google.png" },
  { href: "http://bing.com", src: "bing.png" },
  { href: "http://yahoo.com", src: "yahoo.png" }
];
```

然后，为了完成`<SearchEngines data={data} />`工作，我们只是将`data`数组从对象列表映射到`ClickableImage`组件列表：

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

这`SearchEngines`适用于我们提供给它的任何搜索引擎列表。



### 十、Hooks

React组件中的一个钩子是对特殊函数的调用。所有钩子函数都以“use”一词开头。其中一些可用于为函数组件提供有状态元素（如`useState`），其他可用于管理副作用（如`useEffect`）或缓存/ memoize函数和对象（如`useCallback`）。钩子非常强大，天空是你可以用它们做的事情的极限。

**React钩子函数只能用于函数组件。你不能在类组件中使用它们。**

要查看基本`useState`钩子的示例，让我们使`Button`上面的组件响应click事件。让我们保持在“count”变量中点击它的次数，并将该变量的值显示为它呈现的按钮的标签。

```js
const Button = () => {
  let count = 0;

  return (
    <button>{count}</button>
  );
};

ReactDOM.render(<Button />, mountNode);
```

这个`count`变量将是我们需要引入示例的state元素。这是UI将依赖的一段数据（因为我们正在显示它），它是一个状态元素，因为它会随着时间的推移而改变。

每次在代码中定义变量时，你都将引入一个状态，每次更改该变量的值时，你都会改变该状态。记在脑子里。

在我们改变国家的价值之前`count`，我们需要了解事件。

#### 1. 响应用户事件



### 十一、多组件



### 十二、组件可重用



### 十三、接受用户的输入



### 十四、管理副作用



### 十五、下一步