在 React 中，当我们需要渲染多个组件时：

```js
const Hello = ({name})=>(
  <h1> Hello {name} </ h1> 
);
const Button = () => {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
};
```

当需要同时渲染以上两个组件时，它们不能直接渲染：

```js
// 渲染失败
ReactDOM.render(<Button /><Hello />, document.getElementById('root'));  
```

解决方法一：元素数组的形式

```js
ReactDOM.render([<Button />, <Hello />], document.getElementById('root'));
```

解决方法二：引入新的 DOM 父节点

```js
ReactDOM.render(
  <div>
    <Button />
    <Hello />
  </div>,
  document.getElementById('root')
); // 需要引入新的 DOM 父节点
```

解决方法三：React.Fragment

```js
ReactDOM.render(
  <React.Fragment>
    <Button />
    <Hello />
  </React.Fragment>,
  document.getElementById('root')
);
```

React.Fragment 简写

```js
ReactDOM.render(
  <>
    <Button />
    <Display />
  </>,
  document.getElementById('root')
);
```

