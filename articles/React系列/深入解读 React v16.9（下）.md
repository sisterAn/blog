### 十、Hooks

Hook 是一个特殊的函数，它可以让你“钩入” React 的特性。所有 Hooks 都以 use 一词开头。`useState` 是允许你在 React 函数组件中添加 state 的 Hook，一些可用于管理副作用（如useEffect），一些可用于缓存 memoize 函数和对象（如useCallback、 useMemo）。钩子非常强大，天空是你可以用它们做的事情的极限。

**React hook 函数只能用于函数组件。你不能在类组件中使用它们。**

下面看一个 `useState` 示例，创建  `Button` 并响应 `click` 事件，并在 `count` 中保存点击它的次数。

```js
const Button = () => {
  let count = 0;

  return (
    <button>{count}</button>
  );
};

ReactDOM.render(<Button />, mountNode);
```

我们声明了一个叫 `count` 的 state 变量。React 会在重复渲染时记住它当前的值，并且提供最新的值给我们的函数。我们可以通过调用 `setCount` 来更新当前的 `count`。

#### 1. 响应用户事件

我们给 `button` 组件增加一个`onClick` 事件：

```js
const Button = () => {
  let count = 0;

  return (
    <button onClick={() => console.log('Button clicked')}>
      {count}
    </button>
  );
};

ReactDOM.render(<Button />, mountNode);
```

与 HTML DOM 版本的 `onClick` 不同的是，React 版本的 `onClick` 使用的是一个函数引用，使用 `{}` 引入。

```js
function func() {}

<button onClick={func} />
```



### 十一、多组件



### 十二、组件可重用



### 十三、接受用户的输入



### 十四、管理副作用



### 十五、下一步
