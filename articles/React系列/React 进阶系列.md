该系列文章是适合有一定 React 开发经验的人员，如果你对 React 了解并不深入，请先前往 深入解读 React 系列。

本系列文章不会介绍如何利用 React 构建用户界面，而是帮助你更深入的了解 React 编程模式。

### 一、树

#### 宿主树

一段特定的代码，在浏览器或移动端输出为特定的内容（例如图片或一段文字等），React 也是如此。

React 程序在运行时会输出一个随时间变化的树，在浏览器上，是 DOM 树，在 iOS 上是视图层，我们希望平台根据这个树展示特定的 UI，因此我们称这个树为宿主树。

#### 宿主实例

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

即 React 元素是不可变的，也不是永远存在的，

一个元素就像电影的单帧：它代表了某个特定时刻的 UI。

#### 1. ReactDOM.render

每个 React 项目都有一个入口，通过这个入口将 React 元素树渲染到真实的宿主环境中，这就是 `ReactDOM.render` ：

```js
ReactDOM.render(
  <button className="bottle" />,
  document.getElementById('root')
);
```

当我们调用 `ReactDOM.render(reactElement, domContainer)` 时，我们的意思是：**“亲爱的 React ，将我的 reactElement 映射到 domContaienr 的宿主树上去吧。“**



#### 2. React.createElement

React 会查看 `reactElement.type` （在我们的例子中是 `button` ）然后告诉 React DOM 渲染器创建对应的宿主实例并设置正确的属性：

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
let domNode = document.createElement('button');
domNode.className = 'blue';
domContainer.appendChild(domNode);
```

如果 React 元素在 `reactElement.props.children` 中含有子元素，React 会在第一次渲染中递归地为它们创建宿主实例。

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

之后方法会返回一个对象给你，我们称此对象为 React 元素（ `element` ），它告诉 React 下一个要渲染什么。你的组件（ `component` ）返回一个它们组成的树（tree）。

```js
{
  type: 'div',
  props: {
    className: 'bottle',
    children: 'hello bottle!',
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element'), // 🧐是谁
}
```

如果你用过 React，对 `type`、 `props`、 `key`、 和 `ref` 应该熟悉。 **但 $$typeof 是什么？为什么用 Symbol() 作为它的值**？



#### 3. $typeof 

这又是一个与你学习使用 React 不相关的点，但了解后你会觉得舒坦。这篇文章里也提到了些关于安全的提示，你可能会感兴趣。也许有一天你会有自己的 UI 库，这些都会派上用场的，我真的希望如此。

在客户端 UI 库变得普遍且具有基本保护作用之前，应用程序代码通常是先构建 HTML，然后把它插入 DOM 中：

```js
const messageEl = document.getElementById('message');
messageEl.innerHTML = '<p>' + message.text + '</p>';
```

这样看起来没什么问题，但当你 `message.text` 的值类似 `'<img src onerror="stealYourPassword()">'` 时，**你不会希望别人写的内容在你应用的 HTML 中逐字显示的。**

（有趣的是：如果你只是在前端渲染，这里为 `<script>` 标签，JavaScript 代码不会被运行。但[不要因此](https://gomakethings.com/preventing-cross-site-scripting-attacks-when-using-innerhtml-in-vanilla-javascript/)让你陷入已经安全的错觉。）

为什么防止此类攻击，你可以用只处理文本的 `document.createTextNode()` 或者 `textContent` 等安全的 API。你也可以事先将用户输入的内容，用转义符把潜在危险字符（ `<`、`>` 等）替换掉。

尽管如此，这个问题的成本代价很高，且很难做到用户每次输入都记得转换一次。**因此像 React 等新库会默认进行文本转义：**

```js
<p>
  {message.text}
</p>
```

如果 `message.text` 是一个带有 `<img>` 或其他标签的恶意字符串，它不会被当成真的 `<img>` 标签处理，React 会先进行转义 *然后* 插入 DOM 里。所以 `<img>` 标签会以文本的形式展现出来。

要在 React 元素中渲染任意 HTML，你不得不写 `dangerouslySetInnerHTML={{ __html: message.text }}` 。**其实这种愚蠢的写法是一个功能**，在 code reviews 和代码库审核时，你可以非常清晰的定位到代码。

------

**这意味着React完全不惧注入攻击了吗？不**，HTML 和 DOM 暴露了[大量攻击点](https://github.com/facebook/react/issues/3473#issuecomment-90594748)，对 React 或者其他 UI 库来说，要减轻伤害太难或进展缓慢。大部分存在的攻击方向涉及到属性，例如，如果你渲染 `<a href={user.website}`，要提防用户的网址是 `'javascript: stealYourPassword()'` 。 像 `<div {...userData}>` 写法几乎不受用户输入影响，但也有危险。

React [可以](https://github.com/facebook/react/issues/10506)逐步提供更多保护，但在很多情况下，威胁是服务器产生的，这不管怎样都[应该](https://github.com/facebook/react/issues/3473#issuecomment-91327040)要避免。

不过，转义文本这第一道防线可以拦下许多潜在攻击，知道这样的代码是安全的就够了吗？

```js
// 自动转义
<p>
  {message.text}
</p>
```

**好吧，也不总是有效的**。这就是 `$$typeof` 的用武之地了。

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

这是个有效的办法，因为JSON不支持 `Symbol` 类型。**所以即使服务器存在用 JSON 作为文本返回安全漏洞，JSON 里也不包含 Symbol.for('react.element') **。React 会检测 `element.$$typeof`，如果元素丢失或者无效，会拒绝处理该元素。

特意用 `Symbol.for()` 的好处是 **Symbols 通用于 iframes 和 workers 等环境中**。因此无论在多奇怪的条件下，这方案也不会影响到应用不同部分传递可信的元素。同样，即使页面上有很多个 React 副本，它们也 「接受」 有效的 `$$typeof` 值。

如果浏览器不支持 [Symbols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol#Browser_compatibility) 怎么办？

唉，那这种保护方案就无效了。React仍然会加上 `$$typeof` 字段以保证一致性，但只是 [设置一个数字](https://github.com/facebook/react/blob/8482cbe22d1a421b73db602e1f470c632b09f693/packages/shared/ReactSymbols.js#L14-L16) 而已 —— `0xeac7`。

为什么是这个数字？因为 `0xeac7` 看起来有点像 「React」。

### 四、协调



#### 1. diff

#### 2. Fiber

### 五、渲染

#### 1. 渲染器

#### 2. 条件渲染

#### 3. 列表渲染

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








