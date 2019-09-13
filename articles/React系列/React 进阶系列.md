该系列文章是适合有一定 React 开发经验的人员，如果你对 React 了解并不深入，请先前往 深入解读 React 系列。

本系列文章不会介绍如何利用 React 构建用户界面，而是帮助你更深入的了解 React 编程模式。

### 一、树

#### 1. 宿主树

一段特定的代码，在浏览器或移动端输出为特定的内容（例如图片或一段文字等），React 也是如此。

React 程序在运行时会输出一个随时间变化的树，在浏览器上，是 DOM 树，在 iOS 上是视图层，我们希望平台根据这个树展示特定的 UI，因此我们称这个树为宿主树。

#### 2. 宿主实例

树由节点构成，我们称这些构成树的节点称为实例。

在 DOM 环境中，实例就是我们通常所说的 DOM 节点，和我们平时通过 `document.createElement('div')` 时获取的对象一致，在 iOS 中，实例可以是从 JavaScript 到原生视图的唯一标识。

宿主环境都有各自的实例以及对应的实例属性，例如 DOM 环境的 `domNode.className` 以及 iOS 环境的 `view.tintColor` 。这些实例属性是由原生的 API 来控制，在 React 应用程序中，你通常不会调用这些实例属性，这是 React 的工作。

### 三、元素

在宿主环境中，一个宿主实例是最小的构建单位，而在 React 中，元素是最小的构建单位。

```js
const element = <p>Hello, bottle!</p>;
```

React 元素是一个普通的 JavaScript 对象，它用来描述一个宿主实例：

```js
// JSX 是用来描述这些对象的语法糖。
{
  type: 'p',
  props: { 
    children: 'Hello, bottle!', 
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element'),
}
```

React 是声明式的描述，我们仅仅需要告诉 React 我们需要什么，React 会承担将这种声明式描述转化为我们在屏幕上真实看到的内容。在浏览器中，React DOM 会负责更新 Actual DOM 来与 React 元素保持一致，在移动端，由 React Native 负责。

注意：React 元素是不可变对象。一旦被创建，你就无法更改它的子元素或者属性。如果我们想要更新 React 元素的子元素或属性，都需要创建新的 React 元素树来描述它。

即 React 元素是不可变的，也不是永远存在的，它是在不断的删除和重新创建中。

一个元素就像电影的单帧：它代表了某个特定时刻的 UI。

#### 1. ReactDOM.render

每个 React 项目都有一个入口，通过这个入口将 React 元素树渲染到真实的宿主环境中，这就是 `ReactDOM.render` ：

```js
ReactDOM.render(
  <p>Hello, bottle!</p>,
  document.getElementById('root')
);
```

当我们调用 `ReactDOM.render(reactElement, domContainer[, callback])` 时，它意思是：**“亲爱的 React ，将 reactElement 映射到 domContaienr 的宿主树上去吧。“**

React 会查看 `reactElement.type` （在我们的例子中是 `p` ）然后告诉 React DOM 渲染器创建对应的宿主实例并设置正确的属性：

```js
// 在 ReactDOM 渲染器内部（简化版）
function createHostInstance(reactElement) {
  let domNode = document.createElement(reactElement.type);  
  domNode.className = reactElement.props.className;  
  return domNode;
}
```

在我们的例子中，React 会这样做：

```js
let domNode = document.createElement('p');
domNode.children = 'Hello, bottle!';
domContainer.appendChild(domNode);
```

如果 React 元素在 `reactElement.props.children` 中含有子元素，React 会在第一次渲染中递归地为它们创建宿主实例。

#### 2. React.createElement

你觉得你在写 JSX：

```js
<div className="bottle">hello bottle!</div>
```

其实，你在调用一个方法：

```js
React.createElement(
  'div', /* type */
  { 
    className: 'bottle',
  }, /* props */
  'hello bottle!', /* children */
)
```

之后方法会返回一个对象给你，我们称此对象为 React 元素（ `element` ）。

当你用 JSX 渲染一个组件时：

```js
class Hello extends React.Component {
  render() {
    return <div class='bottle'>Hello {this.props.nickname}</div>;
  }
}

ReactDOM.render(
  <Hello nickname="bottle" />,
  document.getElementById('root')
);
```

实际上，你在这么做：

```js
class Hello extends React.Component {
  render() {
    return React.createElement(
      'div', 
      {
        className: 'bottle',
      }, 
      `Hello ${this.props.nickname}`
    );
  }
}

ReactDOM.render(
  React.createElement(Hello, {nickname: 'bottle'}, null),
  document.getElementById('root'),
);
```

`React.createElement` 会告诉 React 下一个要渲染什么。 实际上它返回一个对象：

```js
// <div className="bottle">hello bottle!</div>
{
  type: 'div',
  props: {
    children: 'hello bottle!',
    className: 'bottle',
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element'), // 🧐是谁
}
```

如果你用过 React，对 `type`、 `props`、 `key`、 和 `ref` 应该熟悉。 **但 $$typeof 是什么？为什么用 Symbol() 作为它的值**？



#### 3. $typeof 

React 应用程序代码通常是先构建 HTML，然后把它插入 DOM 中：

```js
const messageElement = document.getElementById('app');
messageElement.innerHTML = '<p>' + message.text + '</p>';
```

这样看起来没什么问题，但当你 `message.text` 的值类似 `'<img src="https://avatars3.githubusercontent.com/u/19721451?s=460&v=4" />'` 时，你就会发现你的项目已经被 XSS 攻击了。

![XSS](https://user-images.githubusercontent.com/19721451/64854799-17b5db80-d651-11e9-9239-8e681b71e73f.png)

为什么防止此类攻击，你可以用只处理文本的 `document.createTextNode()` 或者 `textContent` 等安全的 API。你也可以事先将用户输入的内容，用转义符把潜在危险字符（ `<`、`>` 等）替换掉。

但是如果这样处理，这个问题的成本代价就会很高，且很难做到用户每次输入都记得转换一次。**因此像 React 等新库会默认进行文本转义：**

```js
<p>
  {message.text}
</p>
```

如果 `message.text` 是一个带有 `<img>` 或其他标签的恶意字符串，它不会被当成真的 `<img>` 标签处理，React 会先进行转义，然后再插入到 DOM 里。所以 `<img>` 标签会以文本的形式展现出来。

![React文本转义](https://user-images.githubusercontent.com/19721451/64854808-1a183580-d651-11e9-9858-b10961c8bcfb.png)

如果，你想要在 React 元素中渲染 HTML，你可以使用 `dangerouslySetInnerHTML={{ __html: message.text }}` ，它是 React 为浏览器 DOM 提供 `innerHTML` 的替换方案，它需要向其传递包含 key 为 `__html` 的对象，以此来警示你。

```js
<p dangerouslySetInnerHTML={{ __html: message.text }} />
```

![XSS-dangerouslySetInnerHTML](https://user-images.githubusercontent.com/19721451/64854799-17b5db80-d651-11e9-9239-8e681b71e73f.png)

[点击查看实例](https://stackblitz.com/edit/react-ghsk6h)



由此说，React 就不会惧怕任何攻击了吗？不。但转义文本这第一道防线可以拦下许多潜在攻击。

```js
// 自动转义
<p>
  {message.text}
</p>
```

但仅仅是这样，对项目安全来说，完全是不够的。

HTML 和 DOM 暴露了[大量攻击点](https://github.com/facebook/react/issues/3473#issuecomment-90594748)，大部分存在的攻击方向涉及到属性，例如，

- 如果你渲染 `<a href={user.website}`，要提防用户的网址是 `'javascript: stealYourPassword()'` 。 
- 像 `<div {...userData}>` 写法几乎不受用户输入影响，但也有危险。

对 React 或者其他 UI 库来说，要减轻攻击伤害实在太难。虽然 React [正在](https://github.com/facebook/react/issues/10506)逐步提供更多保护，但很多情况下，威胁是服务器产生的，这不管怎样都[应该](https://github.com/facebook/react/issues/3473#issuecomment-91327040)要避免。

例如：我们使用用户提供的数据作为 `props` 传递，就存在一个 XSS：

```js
var data = JSON.parse(decodeURI(location.search.substr(1)));

function Foo(props) {
  return <div><div {...props} /><span>{props.content}</span></div>;
}

ReactDOM.render(<Foo {...data} />, container);
```

这种情况下，此 URL 就是一个 XSS 漏洞：

```js
?{"content":"Hello","dangerouslySetInnerHTML":{"__html":"<a%20onclick=\"alert(%27p0wned%27)\">Click%20me</a>"}}
```

这是非常罕见的。有许多不同的方法可以防止获取这样的用户数据。然而，这样做也是不寻常的。所以 React 决定为这些类型的错误添加额外的保护层。

要么：

```js
{ $$typeof:Symbol.for('react.rawhtml'), __html: myHTML }
```

或：

```js
{ [Symbol.for('react.rawhtml')]: myHTML }
```

这就是 `$$typeof` 的用武之地了。

React 元素（elements）是设计好的 plain object：

```js
{
  type: 'marquee',
  props: {
    bgcolor: '#ffa7c4',
    children: 'hi',
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element'),
}
```

虽然通常用 `React.createElement()` 创建它，但这不是必须的。有一些 React 用例来证实像上面这样的 plain object 元素是有效的。当然，你不会想这样写的，但这[可以用来](https://github.com/facebook/react/pull/3583#issuecomment-90296667)优化编译器，在 workers 之间传递 UI 元素，或者将 JSX 从 React 包解耦出来。

但是，如果你的服务器有允许用户存储任意 JSON 对象的漏洞，而前端需要一个字符串，这可能会发生一个问题：

```js
// 服务端允许用户存储 JSON
let expectedTextButGotJSON = {  
  type: 'div',  
  props: {    
    dangerouslySetInnerHTML: {      
      __html: '/* 把你想的搁着 */'    
  },  
},  // ...};let message = { text: expectedTextButGotJSON };

// React 0.13 中有风险
<p>{message.text}</p>
```

在这个例子中，React 0.13[很容易](http://danlec.com/blog/xss-via-a-spoofed-react-element)受到 XSS 攻击。再次声明，**这个攻击是服务端存在漏洞导致的**。不过，React 会为了大家的安全做更多工作。从 React 0.14 开始，它做到了。

React 0.14 修复手段是用 Symbol 标记每个 React 元素（element）：

```js
{
  type: 'marquee',
  props: {
    bgcolor: '#ffa7c4',
    children: 'hi',
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element'),
}
```

这是个有效的办法，因为 JSON 不支持 `Symbol` 类型。**所以即使服务器存在用 JSON 作为文本返回安全漏洞，JSON 里也不包含 Symbol.for('react.element') **。React 会检测 `element.$$typeof`，如果元素丢失或者无效，会拒绝处理该元素。

特意用 `Symbol.for()` 的好处是 **Symbols 通用于 iframes 和 workers 等环境中**。因此无论在多奇怪的条件下，这方案也不会影响到应用不同部分传递可信的元素。同样，即使页面上有很多个 React 副本，它们也 「接受」 有效的 `$$typeof` 值。

如果浏览器不支持 [Symbols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol#Browser_compatibility) 怎么办？

唉，那这种保护方案就无效了。React仍然会加上 `$$typeof` 字段以保证一致性，但只是 [设置一个数字](https://github.com/facebook/react/blob/8482cbe22d1a421b73db602e1f470c632b09f693/packages/shared/ReactSymbols.js#L14-L16) 而已 —— `0xeac7`。

为什么是这个数字？因为 `0xeac7` 看起来有点像 「React」。



#### 4. React.createElement 与 React Component

`React.createElement` 是创建一个 React 元素，它与 React 组件 是不同的：

- React Component 是一个模板。蓝图。全球定义。它可以是函数组件或类组件。
- React 元素是从组件返回的元素。它是一个描述虚拟 DOM 节点的对象。对于函数组件，此元素是函数返回的对象，对于类组件，元素是组件的 render 方法返回的对象。React 元素不是您在浏览器中看到的真实 DOM。它们存储在内存中的 Virtual DOM 对象，你无法改变它们。

当我们告诉 React 在浏览器中渲染元素树时，它首先生成该树的虚拟表示并将其保留在内存中以供日后使用。然后它将继续执行 DOM 操作，使树显示在浏览器中。

当我们告诉 React 更新它先前渲染的元素树时，它会生成更新树的新虚拟表示。现在 React 在内存中有 2 个版本的树！

要在浏览器中呈现更新的树，React 不会丢弃已呈现的内容。相反，它将比较它在内存中的 2 个虚拟版本，计算它们之间的差异，找出主树中需要更新的子树，并且只在浏览器中更新这些子树。

这个过程就是所谓的树协调算法，它使 React 成为使用浏览器 DOM 树的一种非常有效的方法。

这是 React 的智能**差异**算法。它只在主 DOM 树中更新实际**需要**更新的内容，同时保持其他所有内容相同。这种差异化过程是可能的，因为它在内存中保留了 React 的虚拟 DOM 表示。无论 UI 视图需要重新生成多少次，React 将只向浏览器提供所需的“部分”更新。



### 四、协调

#### 1. Virtual DOM

#### 2. diff



#### 3. Fiber



### 五、渲染

#### 1. 渲染器

将 React 渲染在不同的平台展示给用户，这就是渲染器，React DOM、React Native 都是 React 渲染器。那么， React 是如何渲染宿主实例的喃？主要有两种模式：

- 突变模式，更新子节点，不需要重新创建替换父节点，仅仅需要删除重新创建子节点。
- 不变模式，更新子节点，始终替换掉顶级子树的宿主环境。

#### 2. ReactDOM.render 

如果我们在同一个 `domContainer` 中，调用多次 `ReactDOM.render` ，React 是如何处理的喃？

```js
ReactDOM.render(
  <p className="bottle">hello Bottle!</p>,
  document.getElementById('root')
);

// ... 之后 ...

// 应该替换掉 p 宿主实例吗？
// 还是在已有的 p 上更新属性？
ReactDOM.render(
  <p className="an">hello An!</p>,
  document.getElementById('root')
);
```



#### 3. 条件渲染

#### 4. 列表渲染

### 六、组件

#### 1. function 组件

#### 2. class 组件

##### 2.1 constructor

##### 2.2 super(props)

#### 3. React 如何识别 组件是 function 组件还是 class 组件

#### 4. function 组件与 class 组件的区别

### 七、setState

#### 1. setState 机制

#### 2. setState 与 useState

### 八、Hooks

#### 1. 状态同步

#### 2. 顺序

#### 3. setInterval

#### 4. useState

#### 5. useEffect

### 九、副作用

### 十、上下文

### 十一、react-router

### 十一、react-redux








