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



#### 3. `$typeof` 

React 应用程序代码通常是先构建 HTML，然后把它插入 DOM 中：

```js
const messageElement = document.getElementById('app');
messageElement.innerHTML = '<p>' + message.text + '</p>';
```

这样看起来没什么问题，但当你 `message.text` 的值类似 `'<img src onerror="alert(/xss/)" />'` 时，你就会发现你的项目已经被 XSS 攻击了。

![XSS](https://user-images.githubusercontent.com/19721451/65688093-0fbc5980-e09d-11e9-83d4-08e367fb095b.png)

为什么防止此类攻击，你可以用只处理文本的 `document.createTextNode()` 或者 `textContent` 等安全的 API。你也可以事先将用户输入的内容，用转义符把潜在危险字符（ `<`、`>` 等）替换掉。

但是如果这样处理，这个问题的成本代价就会很高，且很难做到用户每次输入都记得转换一次。**因此像 React 等新库会默认进行文本转义：**

```js
<p>
  {message.text}
</p>
```

如果 `message.text` 是一个带有 `<img>` 或其他标签的恶意字符串，它不会被当成真的 `<img>` 标签处理，React 会先进行转义，然后再插入到 DOM 里。所以 `<img>` 标签会以文本的形式展现出来。

![React createElement](https://user-images.githubusercontent.com/19721451/65688098-13e87700-e09d-11e9-9af4-40764a375a9e.png)

如果，你想要在 React 元素中渲染 HTML，你可以使用 `dangerouslySetInnerHTML={{ __html: message.text }}` ，它是 React 为浏览器 DOM 提供 `innerHTML` 的替换方案，它需要向其传递包含 key 为 `__html` 的对象，以此来警示你。

```js
<p dangerouslySetInnerHTML={{ __html: message.text }} />
```

![XSS](https://user-images.githubusercontent.com/19721451/65688093-0fbc5980-e09d-11e9-83d4-08e367fb095b.png)

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



#### 4. React 元素与组件

这里涉及三种相关的东西，分别是：

- 组件
- 组件实例
- 元素

这有点令人惊讶，因为如果你习惯于其他 UI 框架，你可能认为只有两种，大致对应于类（如 `Widget` ）和实例（如新的 `Widget()` ）。React 的情况并非如此，组件实例与元素不是一回事，它们之间也没有一对一的关系。为了说明这一点，请考虑以下代码：

```js
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
import './style.css';

class Hello extends React.Component {
  constructor(props) {
    super(props);
    console.log('This is a component instance:', this);
  }

  render() {
    const another_element = <div>Hello Bottle!</div>;
    console.log('This is also an element:', another_element);
    return another_element;
  }
}

console.log('This is a component:', Hello)

const element = <Hello/>;

console.log('This is an element:', element);

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

对应上面的实例：

- React 组件是一个模板、蓝图、全球定义，它可以是函数组件或类组件，`Hello` 就是一个组件。
- `element` 是一个元素，它并不是 `Hello` 的实例，相反，它是创建组件实例的简单描述。它是包含 `key` 、 `props` 、 `ref` 和 `type` 属性的对象。此处， `key` 和 `ref` 为 `null` ， `props` 为空对象， `type` 为 `Hello` 
- `Hello` 组件实例将在 `element` 元素被渲染时创建
- `another_element` 也是一个元素，它也和 `element` 一样，拥有 `key` 、 `props` 、 `ref` 、 `type` 属性，但是它的`type` 属性值为 `div`，

[点击查看实例](https://stackblitz.com/edit/react-t3gjmz)

`React.createElement` 创建一个 React 元素，React 元素是从组件返回的元素。它是一个描述虚拟 DOM 节点的对象。对于函数组件，此元素是函数返回的对象，对于类组件，元素是组件的 `render` 方法返回的对象。React 元素不是您在浏览器中看到的真实 DOM。它们存储在内存中的 Virtual DOM 对象，你无法改变它们。

当我们告诉 React 在浏览器中渲染元素树时，它首先生成该树的虚拟表示并将其保留在内存中以供日后使用。然后它将继续执行 DOM 操作，使树显示在浏览器中。

当我们告诉 React 更新它先前渲染的元素树时，它会生成更新树的新虚拟表示。现在 React 在内存中有 2 个版本的树！

要在浏览器中呈现更新的树，React 不会丢弃已呈现的内容。相反，它将比较它在内存中的 2 个虚拟版本，计算它们之间的差异，找出主树中需要更新的子树，并且只在浏览器中更新这些子树。

这个过程就是所谓的树协调算法，它使 React 成为使用浏览器 DOM 树的一种非常有效的方法。

这是 React 的智能**差异**算法。它只在主 DOM 树中更新实际**需要**更新的内容，同时保持其他所有内容相同。这种差异化过程是可能的，因为它在内存中保留了 React 的虚拟 DOM 表示。无论 UI 视图需要重新生成多少次，React 将只向浏览器提供所需的“部分”更新。



### 四、Virtual DOM 与内核

#### 1. Virtual DOM

Virtual DOM 是一种变成概念。在这个概念中，UI 是以一种理想化的，或者说是虚拟化的形式保存在内存中，并通过 React DOM、React Native 等渲染器，使之与真实 DOM 同步，这一过程称为协调。

这一方式赋予了 React 声明式的 API，你只需要告诉 React 你想要做什么，React 负责将这些声明性描述转化为真实的 DOM 或视图层展示给用户。这使得你从属性操作、事件处理以及手动 DOM 更新等一系列操作中解救出来。

与其说 Virtual DOM 是一种技术，不如说，它是一种模式，在不同的平台上表达不同的东西。在 React 的世界里，术语 “Virtual DOM” 通常与 React 元素关联在一起，因为它们都是代表了用户界面的对象。而 React 也使用一个名为 “fibers” 的内部对象来存放组件树的附加信息。上述二者也被认为是 React 中 “Virtual DOM” 实现的一部分。

#### 2. diff

React 协调算法用于比较两棵 Virtual DOM 树，以确定 Actual DOM 树哪些部分需要修改。

计算一个树形结构转换成另一个树形结构的最少操作，是一个复杂且值得研究的问题，传统 diff 算法通过循环递归的方法对节点进行操作，算法复杂度 为 `O(n^3)` ，其中 `n` 为树中节点的总数，这效率太低了，如果 React 只是单纯的引入 diff 算法，而没有任何的优化的话，其效率远远无法满足前端渲染所需要的性能。那么 React 是如何实现一个高效、稳定的 diff 算法。

React 将 Virtual DOM 树转换为 Actual DOM 树的最小操作的过程称为调和， diff 算法便是调和的结果，React 通过制定大胆的策略，将 O(n^3) 的时间复杂度转换成 O(n) 。

协调是人们普遍理解为 Virtual DOM 背后的算法。描述如下所示：

当你呈现 React 应用程序时，会生成描述应用程序的节点树并将其保存在内存中。然后将该树刷新到渲染环境（例如，在浏览器应用程序的情况下，它被转换为一组 DOM 操作）。更新应用程序（通常通过 `setState` ）时，会生成一个新树。新树与旧树进行区分，以计算更新渲染应用程序所需的操作。

虽然 Fiber 是协调器的重新编写，但 React 文档中描述的高级算法将大致相同。关键点是：

- 假设不同的组件类型生成实质上不同的树。 React 不会尝试区分它们，而是完全替换旧树。
- 使用键执行列表的区分。密钥应该“稳定，可预测且独特”。

##### 协调与渲染

DOM 只是 React 可以呈现的应用环境之一，其他主要目标是通过 React Native 的本机 iOS 和 Android 视图。 （这就是 “Virtual DOM” 有点用词不当的原因。）

它可以支持这么多目标的原因是因为 React 的设计使得协调和渲染是分开的阶段。协调者负责计算树的哪些部分已经改变；然后渲染器使用该信息来实际更新渲染的应用程序。

这种分离意味着 React DOM 和 React Native 可以使用自己的渲染器，同时共享由 React 核心提供的相同协调程序。

Fiber 重新实现了协调。它主要不涉及渲染，但渲染器需要更改以支持（并利用）新架构。

调度 

确定何时应该进行工作的过程。

工作 

任何必须执行的计算。工作通常是更新的结果（例如 `setState`）。 

React 的设计原则文档在这个主题上非常好，我在这里引用它： 

>  在其当前实现中，React 以递归方式遍历树，并在单个 tick 中调用整个更新树的呈现函数。但是在将来它可能会开始延迟一些更新以避免丢帧。 这是 React 设计中的常见主题。一些流行的库实现了“推送”方法，其中在新数据可用时执行计算。然而，React 坚持“拉”方法，在这种方法中计算可以延迟到必要时。 React 不是通用的数据处理库。它是用于构建用户界面的库。我们认为它在应用程序中具有独特的位置，可以知道哪些计算现在是相关的，哪些不是。 如果某些东西在屏幕外，我们可以延迟任何与之相关的逻辑。如果数据的到达速度快于帧速率，我们可以合并并批量更新。我们可以优先考虑来自用户交互（例如由按钮点击引起的动画）的工作，而不是重要的背景工作（例如渲染刚刚从网络加载的新内容）以避免丢帧。 

关键点是： 

- 在 UI 中，不必立即应用每个更新;实际上，这样做可能会浪费，导致帧丢失并降低用户体验。 
- 不同类型的更新具有不同的优先级 - 动画更新需要比例如来自数据存储的更新更快地完成。 
- 基于推送的方法需要应用程序（你，程序员）决定如何安排工作。基于拉取的方法允许框架（ React ）变得聪明并为你做出决策。

React 目前没有以显着的方式利用调度;更新导致整个子树立即重新渲染。改造 React 的核心算法以利用调度是 Fiber 背后的驱动理念。 

现在我们已经准备好深入了解 Fiber 的实施。下一节比我们到目前为止所讨论的更具技术性。在继续之前，请确保你对之前的材料感到满意。

#### 3. Fiber

Fiber 是 React 16 中新的协调引擎，是 React 核心算法的持续重新实现。它的主要目的是使 Virtual DOM 可以进行增量式渲染：能够将渲染工作分割成块并将其分散到多个帧中。

React Fiber 的目标是增加其对动画，布局和手势等区域的适用性。 其他主要功能包括在新更新进入时暂停，中止或重复工作的能力；为不同类型的更新分配优先级的能力；和新的并发原语。



 我们即将讨论React Fiber架构的核心。纤维是一种比应用程序开发人员通常想到的更低级别的抽象。如果您发现自己在理解它时感到沮丧，请不要气馁。继续尝试，它最终会有意义。 （当你最终得到它时，请建议如何改进这一部分。） 

开始了！

我们已经确定，Fiber 的主要目标是使 React 能够利用调度。具体来说，我们需要能够

- 暂停工作，稍后再回来。
- 为不同类型的工作分配优先权。
- 重用以前完成的工作。
- 如果不再需要，则中止工作。

为了做到这一点，我们首先需要一种方法将工作分解为单元。从某种意义上说，这就是 Fiber 。Fiber 代表一种工作单元。

更进一步，让我们回到 React 组件的概念作为数据的函数，通常表示为

```js
v = f(d)
```

因此，呈现 React 应用程序类似于调用其主体包含对其他函数的调用的函数，依此类推。在考虑 Fiber 时，这种类比很有用。

计算机通常跟踪程序执行的方式是使用调用堆栈。执行函数时，会向堆栈添加新的堆栈帧。该堆栈帧表示该功能执行的工作。

在处理 UI 时，问题在于如果同时执行太多工作，则可能导致动画丢帧并且看起来不稳定。更重要的是，如果它被更新的更新所取代，那么其中一些工作可能是不必要的。这是 UI 组件和功能之间的比较分解的地方，因为组件比一般的功能具有更多的特定问题。

较新的浏览器（和 React Native ）实现了有助于解决这个问题的 API：requestIdleCallback 调度在空闲期间调用的低优先级函数，requestAnimationFrame 调度在下一个动画帧上调用的高优先级函数。问题是，为了使用这些API，您需要一种方法将渲染工作分解为增量单元。如果仅依赖于调用堆栈，它将继续工作直到堆栈为空。

如果我们可以自定义调用堆栈的行为以优化渲染UI，那不是很好吗？如果我们可以随意中断调用堆栈并手动操作堆栈帧，那不是很好吗？

这就是 React Fiber 的目的。Fiber 是堆栈的重新实现，专门用于 React 组件。你可以将单个 Fiber 视为虚拟堆栈帧。

重新实现堆栈的优点是，你可以将堆栈帧保留在内存中，然后执行它们（无论何时）。这对于实现我们的调度目标至关重要。

除了调度之外，手动处理堆栈帧还可以释放并发和错误边界等功能。我们将在以后的部分中介绍这些主题。

在下一节中，我们将更多地关注 Fiber 的结构。



##### `type` 和 `key`



##### `child` 和 `sibling`



##### `return`



##### `pendingProps` 和 `memoizedProps`



##### `pendingWorkPriority`



##### `alternate`



##### `output`



### 五、渲染

#### 1. 渲染器

将 React 渲染在不同的平台展示给用户，这就是渲染器，React DOM、React Native 都是 React 渲染器。那么， React 是如何渲染宿主实例的喃？主要有两种模式：

- 突变模式，更新子节点，不需要重新创建替换父节点，仅仅需要删除重新创建子节点。
- 不变模式，更新子节点，始终替换掉顶级子树的宿主环境。

#### 2. 多次 ReactDOM.render 

如果我们在同一个 `domContainer` 中，调用多次 `ReactDOM.render` ，React 是如何处理的喃？

```js
// let domContainer = document.getElementById('root');
// let domNode = document.createElement('p');
// domNode.textContent='Hello Bottle!';
// domContainer.appendChild(domNode);
ReactDOM.render(
  <p>hello Bottle!</p>,
  document.getElementById('root')
);

// 能重用宿主实例吗？能！(p → p)
// domNode.textContent='Hello An!';
ReactDOM.render(
  <p>hello An!</p>,
  document.getElementById('root')
);

// 能重用宿主实例吗？不能！(p → button)
// domContainer.removeChild(domNode);
// domNode = document.createElement('button');
// domNode.className = 'add';
// domNode.textContent = 'Add';
// domContainer.appendChild(domNode);
ReactDOM.render(
  <button className='add'>Add</button>,
  document.getElementById('root')
);

// 能重用宿主实例吗？能！(button → button)
// domNode.className = 'sub';
// domNode.textContent = 'Sub';
ReactDOM.render(
  <button className='sub'>Sub</button>,
  document.getElementById('root')
);
```

即：**如果相同的元素类型在同一个地方先后出现两次，React 会重用已有的宿主实例。**

同样的处理方式也适用于 React 子树。

#### 3. 条件渲染

```js
// 第一次渲染
ReactDOM.render(
  <div>
    <input />
  </div>,
  domContainer
);

// 下一次渲染
ReactDOM.render(
  <div>
    <p>Hello Bottle!</p>
    <input />
  </div>,
  domContainer
);
```

在这个例子中，`<input>` 宿主实例会被重新创建。React 会遍历整个元素树，并将其与先前的版本进行比较：

- `dialog → dialog` ：能够重用宿主实例吗？**能 — 因为类型匹配。**
  - `(null) → p` ：需要插入一个新的 `p` 宿主实例。
  - `input → input` ：能够重用宿主实例吗？**能 — 因为类型匹配。**

因此，React 这样执行更新：

```js
let inputNode = divNode.firstChild;
let pNode = document.createElement('p');
pNode.textContent = 'Hello Bottle';
divNode.insertBefore(pNode, inputNode);
```

#### 4. 列表渲染

```js
function ShoppingList({ list }) {
  return (
    <form>
      {list.map(item => (
        <p key={item.productId}>
          You bought {item.name}
          <br />
          Enter how many do you want: <input />
        </p>
      ))}
    </form>
  )
}
```



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

**函数组件** 的本质是函数，没有 state 的概念的，因此**不存在生命周期**一说，仅仅是一个 **render 函数**而已。

但是引入 **Hooks** 之后就变得不同了，它能让组件在不使用 class 的情况下使用 state 以及其他的 React特性，相比与 class 的生命周期概念来说，它更接近于实现状态同步，而不是响应生命周期事件。但我们可以利用 `useState`、 `useEffect()` 和 `useLayoutEffect()` 来模拟实现生命周期。

即：**Hooks 组件更接近于实现状态同步，而不是响应生命周期事件**。

#### 2. 顺序

#### 3. setInterval

#### 4. useState

#### 5. useEffect

##### 5.1 `[]`

##### 5.2 在 useEffect 中网络请求

##### 5.3 无限重复请求问题

##### 5.4 useEffect 里获取 state、props

### 九、副作用



### 十、上下文



### 十一、react-router



### 十一、react-redux







