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

### 一、JSX 不是 HTML

浏览器不识别 JSX。我们在浏览器中运行 JSX，会报错：

![JSX](/Users/lilunahaijiao/Desktop/屏幕快照 2019-08-03 上午11.51.02.png)

所以，在项目中运用 JSX，我们需要使用像 Babel 或 TypeScript 这样的转换器。例如，当 我们使用 create-react-app 创建项目时，就会在内部使用 Babel 来转换项目中的 JSX。

**JSX 基本上是一种折中**，使我们能够使用与 HTML 非常相似的语法，使用编译器将其转换为`React.createElement`调用，而不是直接使用 `React.createElement` 语法创建 React 组件。

React 组件是一个返回 React 元素的 JS 函数。当使用 JSX 时，`<tag></tag>`语法会被转化为`React.createElement("tag")`。在创建 React 组件时应该牢记这一点。我们不是在写 HTML，而实在使用 JS 扩展来返回创建React元素（实际上是 JS 对象）的函数调用。

因此，**JSX 允许我们类HTML的语法来表示React树，浏览器和React均不需要识别它，只有编译器才有。我们发送给浏览器的是无JSX代码。**

