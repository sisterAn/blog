Hooks 与 React Component：

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

