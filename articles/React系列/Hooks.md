## 4）Hooks 的特别之处

根据 React 的官方文档，`useEffect()`和 `useLayoutEffect()`都是等效于 `componentDidUpdate()`/ `componentDidMount()`的存在，但实际上两者在一些细节上还是有所不同：

### 4.1）先来未必先走

`useLayoutEffect()`永远比 `useEffect()`先执行，即便在你的代码中 `useEffect()`是写在前面的。所以 `useLayoutEffect()`才是事实上和 `componentDidUpdate()`/ `componentDidMount()`平起平坐的存在。

`useEffect()`会在父子组件的 `componentDidUpdate()`/ `componentDidMount()`都触发之后才被触发。当父子组件都用到 `useEffect()`时，子组件中的会比父组件中的先触发。

### 4.2）不团结的 Cleanup

同样都拥有 Cleanup 函数，`useLayoutEffect()`和它的 Cleanup 未必是挨着的。

当父组件是 Hooks、子组件是 Class 时，能够很明显看出，`useLayoutEffect()`的 Cleanup 会在 `getSnapshotBeforeUpdate()`和 `componentDidUpdate()`之间被调用，而 `useLayoutEffect()`则是和 `componentDidUpdate()`同级，按照更新过程的顺序被调用。

Hooks 作为子组件时也是这么个过程，只是没有了子组件，看上去不那么明显罢了。

而 `useEffect()`就不一样，它和它的 Cleanup 紧密团结在一起，每次执行都是前后脚一起的，从不分离。



### Hooks 与 React Component

- 您不必使用类实例及其 state 状态。您仅仅需要使用在每个渲染上刷新的简单函数。state 被明确声明，没有任何隐藏。这一切意味着你将在代码中遇到更少的惊喜。
- 您可以将相关的有状态逻辑分组，并将其分为独立的可组合和可共享单元。这使得更容易将复杂组件分解为更小的部件。它还使测试组件更容易。
- 您可以以声明方式使用任何有状态逻辑，而无需在组件树中使用任何分层“嵌套”。

```js
const Button =（{clickAction}）=> { 
  return（
    <button onClick = {clickAction}> 
      +1 
    </ button> 
  ）; 
}; 

const Display =（{content}）=>（
  <pre> {content} </ pre> 
）; 

const CountManager =（）=> { 
  const [count，setCount] = useState（0）; 

  const incrementCounter =（）=> { 
    setCount（count + 1）; 
  }; 

  return（
    <div> 
      <Button clickAction = {incrementCounter} /> 
      <Display content = {count} /> 
    </ div> 
  ）; 
};
```

