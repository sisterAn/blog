### 一、forwardRef 用法

在 `React.createRef` 中已经介绍过，有三种方式可以使用 React 元素的 ref

ref 是为了获取某个节点的实例，但是 函数式组件（PureComponent） 是没有实例的，不存在 this的，这种时候是拿不到函数式组件的 ref 的。

为了解决这个问题，由此引入 `React.forwardRef`， **`React.forwardRef` 允许某些组件接收 ref，并将其向下传递给 子组件**

```js
const ForwardInput = React.forwardRef((props, ref) => (
  <input ref={ref} />
));

class TestComponent extends React.Component {
  constructor(props) {
    super(props);
    this.inputRef = React.createRef(); // 创建 ref 存储 textRef DOM 元素
  }
  componentDidMount() {
    this.inputRef.current.value = 'forwardRef'    
  }
  render() {
    return ( // 可以直接获取到 ForwardInput input 的 ref：
      <ForwardInput ref={this.inputRef}>
    )
  }
}
```

- 只在使用 `React.forwardRef` 定义组件时，**第二个参数 ref** 才存在

- 在项目中组件库中尽量不要使用  `React.forwardRef` ，因为它可能会导致子组件被 **破坏性更改**

- **函数组件 和 class 组件均不接收 `ref` 参数** ，即 props 中不存在 `ref`，**ref 必须独立 props** 出来，否则会被 React 特殊处理掉。

- **通常在 高阶组件 中使用 `React.forwardRef`**

  ```js
  function enhance(WrappedComponent) {
    class Enhance extends React.Component {
      componentWillReceiveProps(nextProps) {
        console.log('Current props: ', this.props);
        console.log('Next props: ', nextProps);
      }
      render() {
        const {forwardedRef, ...others} = this.props;
        // 将自定义的 prop 属性 “forwardedRef” 定义为 ref
        return <WrappedComponent ref={forwardedRef} {...others} />;
      }
    }
    // 注意 React.forwardRef 回调的第二个参数 “ref”。
    // 我们可以将其作为常规 prop 属性传递给 Enhance，例如 “forwardedRef”
    // 然后它就可以被挂载到被 Enhance 包裹的子组件上。
    return React.forwardRef((props, ref) => {
      return <Enhance {...props} forwardedRef={ref} />;
    });
  }
  
  // 子组件
  class MyComponent extends React.Component {
    focus() {
      // ...
    }
    // ...
  }
  
  // EnhancedComponent 会渲染一个高阶组件 enhance(MyComponent)
  const EnhancedComponent = enhance(MyComponent);
  
  const ref = React.createRef();
  
  // 我们导入的 EnhancedComponent 组件是高阶组件（HOC）Enhance。
  // 通过React.forwardRef 将 ref 将指向了 Enhance 内部的 MyComponent 组件
  // 这意味着我们可以直接调用 ref.current.focus() 方法
  <EnhancedComponent
    label="Click Me"
    handleClick={handleClick}
    ref={ref}
  />;
  ```

### 二、forwardRef 与 useImperativeHandle

`useImperativeHandle` 可以让你在使用 `ref` 时自定义暴露给父组件的实例值。在大多数情况下，应当避免使用 ref 这样的命令式代码。`useImperativeHandle` 应当与 `forwardRef`一起使用：

```
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

在本例中，渲染 `<FancyInput ref={fancyInputRef} />` 的父组件可以调用 `fancyInputRef.current.focus()`。

### 三、源码解读

```js
export default function forwardRef<Props, ElementType: React$ElementType>(
  render: (props: Props, ref: React$Ref<ElementType>) => React$Node,
) {
  if (__DEV__) {
    if (render != null && render.$$typeof === REACT_MEMO_TYPE) {
      warningWithoutStack(
        false,
        'forwardRef requires a render function but received a `memo` ' +
          'component. Instead of forwardRef(memo(...)), use ' +
          'memo(forwardRef(...)).',
      );
    } else if (typeof render !== 'function') {
      warningWithoutStack(
        false,
        'forwardRef requires a render function but was given %s.',
        render === null ? 'null' : typeof render,
      );
    } else {
      warningWithoutStack(
        // Do not warn for 0 arguments because it could be due to usage of the 'arguments' object
        render.length === 0 || render.length === 2,
        'forwardRef render functions accept exactly two parameters: props and ref. %s',
        render.length === 1
          ? 'Did you forget to use the ref parameter?'
          : 'Any additional parameter will be undefined.',
      );
    }

    if (render != null) {
      warningWithoutStack(
        render.defaultProps == null && render.propTypes == null,
        'forwardRef render functions do not support propTypes or defaultProps. ' +
          'Did you accidentally pass a React component?',
      );
    }
  }

  /**
   * REACT_FORWARD_REF_TYPE 并不是 React.forwardRef 创建的实例的 $$typeof
   * React.forwardRef 返回的是一个对象，而 ref 是通过实例的参数形式传递进去的，
   * 实际上，React.forwardRef 返回的是一个 ReactElement，它的 $$typeof 也就是 REACT_ELEMENT_TYPE
   * 而 返回的对象 是作为 ReactElement 的 type 存在
   */
  return { // 返回一个对象
    $$typeof: REACT_FORWARD_REF_TYPE, // 并不是 React.forwardRef 创建的实例的 $$typeof
    render, // 函数组件
  };
}
```
