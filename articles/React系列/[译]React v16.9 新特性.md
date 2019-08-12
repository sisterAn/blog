今天我们发布了 React 16.9。它包含了一些新特性、bug修复以及新的弃用警告，以便与筹备接下来的主要版本。



## 新弃用

### 重命名 Unsafe 生命周期方法

一年前，我们宣布 unsafe 生命周期方法重命名为：

- `componentWillMount` → `UNSAFE_componentWillMount`
- `componentWillReceiveProps` → `UNSAFE_componentWillReceiveProps`
- `componentWillUpdate` → `UNSAFE_componentWillUpdate`

**React v16.9 不包含破坏性更改，而且旧的生命周期方法在此版本依然沿用**。但是，当你在新版本中使用旧的生命周期方法时，会提示如下警告：

![Warning:componentWillMount has been renamed,and is not recommended for use.](https://i.imgur.com/sngxSML.png)

正如警告所示，对于每种 unsafe 的方法，通常有更好的解决方案。但你可能没有过多时间去迁移或测试这些组件。在这种情况下，我们建议运行一个自动重命名它们的 **codemod** 脚本：

```js
cd your_project
npx react-codemod rename-unsafe-lifecycles
```

（注意：这里使用的是 `npx`，不是 `npm` ，`npx` 是 Node 6+ 默认提供的实用程序。）

运行 **codemod** 将会替换旧的生命周期，如 `componentWillMount` 将会替换为 `UNSAFE_componentWillMount` :

![Codemode in action](https://i.imgur.com/Heyvcyi.gif)

新命名的生命周期（例如：`UNSAFE_componentWillMount`）在 React 16.9 和 React 17.x 继续使用，但是，新的 `UNSAFE_` 前缀将帮助具有问题的组件在代码 review 和 debugging 期间脱颖而出。（如果你不喜欢，你可以引入 严格模式（Strict Mode）来进一步阻止开发人员使用它 。）

> 点击此链接，学习更多关于 [版本策略以及稳定性承诺](https://zh-hans.reactjs.org/docs/faq-versioning.html#commitment-to-stability)



### 弃用：javascript: URLs

以 `javascript:` 开头的 URL 很容易遭受攻击，因为它很容易意外在标签中（`<a href>`）引入未经处理的输出，造成安全漏洞。

```js
const userProfile = {
  website: "javascript: alert('you got hacked')",
};
// This will now warn:
<a href={userProfile.website}>Profile</a>
```

在 React 16.9 中，这种模式将继续有效，但它将输出一个警告，如果你逻辑上需要使用 `javascript:` 开头的 URL，请尝试使用 React 事件处理程序代替。（万不得已，你可以使用 `dangerouslySetInnerHTML` 来规避保护，但仍然是不鼓励使用的并且往往会导致安全漏洞。）

在未来的主要版本中，如果遇到 `javascript:` 形式的 URL，React 将抛出错误。



### 弃用 “Factory” 组件

在用 Babel 编译 JavaScript 类流行前，React 支持 “factory” 组件，它使用 `render` 方法返回一个对象。

```js
function FactoryComponent() {
  return { render() { return <div />; } }
}
```

这种模式令人困惑，因为它看起来太像一个函数组件，但它不是一个。(函数组件只会返回像上述示例中的 `<div />` ）。

这种模式几乎从未在外部使用过，并且支持它会导致 React 变大、变慢。因此，我们在 16.9 中弃用此模式，并且遇到时，输出警告。如果你在项目中依赖此组件，可以添加 `FactoryComponent.prototype = React.Component.prototype` 作为解决方法。或者，你可以将它转换为 class 组件或函数组件。

我们预计大多数代码库不会受此影响。



## 新特性

### 用于测试的一部函数 `act()`

React 16.8 引入了名为 `act()` 的新测试实用程序来帮助你编写更匹配浏览器行为的测试代码。例如，对单个 `act()` 中的多个状态更新进行批处理。这与 React 已有的处理真实浏览器事件时的工作方式相匹配，并有助于为将来 React 组件更频繁地批处理更新做准备。

然而，React v16.8 中的 `act()` 仅支持同步函数，有时，你可能在测试环境下看到以下警告，但无法轻易修复：

```js
An update to SomeComponent inside a test was not wrapped in act(...).
```

**在 React 16.9 中 `act()` 支持异步函数** ，你可以在调用它时，使用 `await` ：

```js
await act(async () => {
  // ...
});
```

这将解决以前无法使用 `act()` 的情况，例如当 state 更新位于异步函数中时。因此，**你现在应该能够测试中修复所有关于 `act()` 的警告了** 。

我们听说，现在还没有足够的信息关于如何使用 `act()` 编写测试用例。新的测试技巧指南介绍了一些常见方案，以及 `act()` 如何帮助您编写良好的测试。这些示例使用原生 DOM API，但您也可以使用 [React Testing Library](https://testing-library.com/docs/react-testing-library/intro) 来减少样板代码。它的许多方法已经在内部使用 `act()` 。

如果你遇到 `act()` 的相关问题，请在[问题跟踪器上](https://github.com/facebook/react/issues)告知我们，我们会尽力提供帮助。



### 使用 `<React.Profiler>` 进行性能评估

在 React 16.5 中，我们介绍了新的 [React Profiler for DevTools](https://zh-hans.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html) 来帮助开发人员发现项目中的性能瓶颈。在 React 16.9 中，我们提供了一种编程的方式来收集测量你的代码，这就是 `<React.Profiler>` ，我们预计大多数较小的应用不会使用它，但在大型应用中跟踪性能回归会很方便。

`<Profiler>` 测量 React 应用程序渲染的频率以及渲染的 "成本" 。其目的是帮助识别应用程序中渲染缓慢的部分，并且可能更益与 [memoization 等优化](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-to-memoize-calculations) 。

可以将 `<Profiler>` 添加到 React 项目中的任意一个子树上，来测量该子树的渲染成本。它需要两个 `props` ：`id` (`string`) 和 `onRender` 回调(`function`)，当树中的组件"提交"更新时，React 将调用它。

```js
render(
  <Profiler id="application" onRender={onRenderCallback}>
    <App>
      <Navigation {...props} />
      <Main {...props} />
    </App>
  </Profiler>
);
```

要了解关于 `Profiler` 和传递给 `onRender` 回调的参数的更多详细信息，请查看 [`Profiler` 文档](https://zh-hans.reactjs.org/docs/profiler.html)。

> 注意：
>
> `Profiling` 会增加一些额外的开销，因此在生产构建中禁止使用它。
>
> 如果想要在生产环境中进行性能分析，React 提供了特殊的生产构建，并启用了分析模式。在 [fb.me/react-profiling](https://fb.me/react-profiling) 阅读更多关于如何使用此构建的更多信息。



## 显著的 bug 修复

此版本包含一些一些其他显著的提升：

- 在 `<Suspense>` 组件中调用 `findDOMNode()`  造成崩溃，已修复
- 保存已删除的子树导致内存泄漏，已修复
- 在 `useEffect` 中，使用 `setState` 引起的循环引用，现在会输出错误（这与在 class 组件中的 `componentDidUpdate` 使用 `setState` 导致的错误一致）

感谢所有帮助解决这些问题的贡献者，你可以在[此处](https://zh-hans.reactjs.org/blog/2019/08/08/react-v16.9.0.html#changelog)找到完整的日志。

本文翻译自：https://reactjs.org/blog/2019/08/08/react-v16.9.0.html