### 一、createElement 用法

一个简单的 `React.createElement` 例子：

```js
ReactDOM.render(
  React.createElement(
    'div',
    null,
    'Hello Bottle' 
  ),
  document.getElementById('root'),
);
```

`React.render` 与 `React.createElement` 是 React 最核心的 API 方法，每一个 React 项目都必须要引入这两个API。

#### ReactDOM.render

这基本上是React应用程序进入浏览器DOM 的**入口点**。它有两个参数：

1. 第一个参数是向浏览器呈现的内容。
2. 第二个参数是WHERE在浏览器中呈现React元素。这必须是存在于静态呈现的HTML中的有效DOM节点。。

什么是React元素？它是描述DOM元素的 Virtual 元素。是 `React.createElement `API方法返回的内容。

#### React.createElement

在React中，我们不使用字符串来表示DOM元素（如上面的本机DOM示例中），而是使用对方法的调用来表示带有**对象的** DOM元素`React.createElement`。这些对象称为React元素。

该`React.createElement`函数有很多参数：

- 第一个参数是要表示的DOM元素的HTML标记，`div`在此示例中。
- 第二个参数为任何属性（如`id`，`href`，`title`，等），我们想要的DOM元素有。`div`我们使用的简单没有属性，所以我们`null`在那里使用。
- 第三个参数是DOM元素的内容。我们在那里放了一个“Hello React”字符串。可选的第三个参数以及它后面的所有可选参数，形成渲染元素的**子**列表。元素可以包含0个或更多子元素。

**React元素在内存中创建**。为了实际在DOM中显示一个React元素，我们使用这个`ReactDOM.render`方法来做很多事情来找出将React元素的状态反映到浏览器中的实际DOM树中的最佳方法。

##### 1. 标记

所有我们自定义的 React 元素首字母必须是大写，因为，JSX编译器（如Babel）会将所有以小写字母开头的名称视为HTML元素的名称。

```js
// HTML 元素以字符串的形式，传递给 React.createElement 调用
React.createElement("input", null)
// React 元素以变量的形式，传递给 React.createElement 调用
React.createElement(Input, null)
```

##### 2. 参数/属性

使用函数组件时，通常我们将属性列表命名为 props（标准做法） ，同时也可以使用解构的方法来编写：

```js
function Hello({name}) {
  return (
    <div>{name}</div>
  )
}

ReactDOM.render(
  <Hello name='Bottle' />
  document.getElementById('root'),
);
```

请注意，props 是可选的。有些组件没有 props。但是，组件的返回值是必须的。React 组件不能返回 `undefined`（显式或隐式）。它必须返回一个值。它可以返回 `null`以使渲染器忽略其输出。

##### 3. DOM 元素的内容

可选的第三个参数以及它后面的所有可选参数，形成渲染元素的**子**列表。元素可以包含0个或更多子元素。

#### React.createElement 与 React Component

`React.createElement` 是创建一个 React 元素，它与 React 组件 是不同的：

- React Component 是一个模板。蓝图。全球定义。它可以是函数组件或类组件。
- React 元素是从组件返回的元素。它是一个描述虚拟 DOM 节点的对象。对于函数组件，此元素是函数返回的对象，对于类组件，元素是组件的 render 方法返回的对象。React 元素不是您在浏览器中看到的真实 DOM。它们存储在内存中的 Virtual DOM 对象，你无法改变它们。

当我们告诉React在浏览器中渲染元素树时，它首先生成该树的虚拟表示并将其保留在内存中以供日后使用。然后它将继续执行DOM操作，使树显示在浏览器中。

当我们告诉React更新它先前渲染的元素树时，它会生成更新树的新虚拟表示。现在React在内存中有2个版本的树！

要在浏览器中呈现更新的树，React不会丢弃已呈现的内容。相反，它将比较它在内存中的2个虚拟版本，计算它们之间的差异，找出主树中需要更新的子树，并且只在浏览器中更新这些子树。

这个过程就是所谓的树协调算法，它使React成为使用浏览器DOM树的一种非常有效的方法。

这是React的智能**差异**算法。它只在主DOM树中更新实际**需要**更新的内容，同时保持其他所有内容相同。这种差异化过程是可能的，因为它在内存中保留了React的虚拟DOM表示。无论UI视图需要重新生成多少次，React将只向浏览器提供所需的“部分”更新。