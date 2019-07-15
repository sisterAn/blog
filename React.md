在前端开发过程中，源码解读是必不可少的一个环节，我们直接进入主题，注意当前 React 版本号 **16.8.6**。

**注意：react 包文件仅仅是 React components 的必要的、功能性的定义，它必须要结合 React renderer一起使用（web下是 react-dom，原生app环境下是react-native）。即 react 仅仅是定义节点与表现行为的包，具体如何渲染、如何更新这是与平台相关的，都是放在react-dom、react-native 包里的。这是我们只分析 web 环境的，即我们不会只分析 react 包文件，会结合 react 包与 react-dom、react-reconciler 及其他相关包一起分析。**

React 16.8.6 使用 FlowTypes 静态类型检查器，我们需要在开发工具中支持 Fow（以 vscode 为例）：

- 安装 `Flow Language Support` 插件

- 配置 `workspace/.vscode/settings.json`

  ```js
  {
    "flow.useNPMPackagedFlow": true,
    "javascript.validate.enable": false
  }
  ```

关于 Flow 更多请看 [Flow官网](https://flow.org/)。

### React

首先，从 react 入口，打开 react 源码库 `index.js`:

```js
'use strict';
const React = require('./src/React');
// TODO: 决定顶层文件导出格式
// 虽然是旁门左道，但它可以使 React 在 Rollup 和 Jest 上运行
module.exports = React.default || React;
```

进入 ./src/React。

#### React API

其中 React 完整内容是：

```js
const React = { // React 暴露出来的 API
  Children: { // 提供处理 props.children 的方法，由于 props.children 是一个类数组的类型，可以用 React.Children 来处理
    map,
    forEach,
    count,
    toArray,
    only,
  },

  createRef, // 创建 ref 函数, React.createRef()
  Component, //  React 组件类
  PureComponent, // React 纯组件，和 React.Component类似，都是定义一个组件类。不同是 React.Component 没有实现 shouldComponentUpdate()，而 React.PureComponent 通过props和state的浅比较实现了。

  createContext, // context 创建方法
  forwardRef, // 用来解决 HOC 组件传递 ref 的问题
  lazy, // 实现异步加载的功能模块
  memo, // 也是一个高阶组件，类似于React.PureComponent，不同于React.memo是function组件，React.PureComponent是class组件。

  error,
  warn,

  // Hooks是React v16.7.0-alpha开始加入的新特性,可以让你在class以外使用state和其他React特性
  useCallback, // 回调钩子，当输入对象改变时调用
  useContext, // 可以订阅React context而不用引入嵌套
  useEffect, // 副作用钩子，为函数式组件带来执行副作用的能力
  useImperativeHandle, // 自定义使用 ref 时，公开给父组件的实例值，应和 forwardRef 一起使用
  useDebugValue,
  useLayoutEffect, // api与useEffect相同，使用它从DOM读取布局并同步重新渲染
  useMemo, // 当输入对象改变时，返回一个 memoized 值
  useReducer, // useState的替代方案，允许你使用一个reducer来管理一个复杂组件的局部状态
  useRef, // 返回 ref 对象，可以通过 .current 访问 ref 实例的属性方法
  useState, // 状态钩子，可以在一个函数式组件中调用它，为这个组件增加一些内部的状态

  /** 
   * 用 Symbol 来表示 React 的 Fragment、StrictMode、Suspense 组件
   *    Fragment: 可以聚合一个子元素列表，并且不在DOM中增加额外节点
   *    StrictMode: 可以在开发阶段开启严格模式，发现应用存在的潜在问题，提升应用的健壮性
   *    Suspense: 在 React.lazy 时，import 失败或者异常时，就需要使用 Suspense 给出错误提示
   */
  Fragment: REACT_FRAGMENT_TYPE, // var REACT_FRAGMENT_TYPE = hasSymbol ? Symbol.for('react.fragment') : 0xeacb;
  Profiler: REACT_PROFILER_TYPE,
  StrictMode: REACT_STRICT_MODE_TYPE, // var REACT_STRICT_MODE_TYPE = hasSymbol ? Symbol.for('react.strict_mode') : 0xeacc;
  Suspense: REACT_SUSPENSE_TYPE, // var REACT_SUSPENSE_TYPE = hasSymbol ? Symbol.for('react.suspense') : 0xead1;

  // ReactElement 相关
  createElement: __DEV__ ? createElementWithValidation : createElement, // 创建 ReactElement
  cloneElement: __DEV__ ? cloneElementWithValidation : cloneElement, // 克隆 ReactElement
  createFactory: __DEV__ ? createFactoryWithValidation : createFactory, // 创建一个专门用来创建某一类 ReactElement 的工厂
  isValidElement: isValidElement, // 验证是否是一个 ReactElement

  version: ReactVersion, // React 功能版本号

  unstable_ConcurrentMode: REACT_CONCURRENT_MODE_TYPE,

  __SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED: ReactSharedInternals, // 顾名思义： React 内部元素，不要使用
};

// Note: some APIs are added with feature flags.
// Make sure that stable builds for open source
// don't modify the React object to avoid deopts.
// Also let's not expose their names in stable builds.

if (enableStableConcurrentModeAPIs) {
  React.ConcurrentMode = REACT_CONCURRENT_MODE_TYPE;
  React.unstable_ConcurrentMode = undefined;
}

if (enableJSXTransformAPI) {
  if (__DEV__) {
    React.jsxDEV = jsxWithValidation;
    React.jsx = jsxWithValidationDynamic;
    React.jsxs = jsxWithValidationStatic;
  } else {
    React.jsx = jsx;
    // we may want to special case jsxs internally to take advantage of static children.
    // for now we can ship identical prod functions
    React.jsxs = jsx;
  }
}

export default React;
```

下面逐个击破。

**附 V16 个个版本的更新：**

**React v16.0**

- render 支持返回 Array 和 String 、Error Boundaries、createPortal、支持自定义 DOM 属性、减少文件体积、Fiber；

**React v16.1**

- react-call-return；

**React v16.2**

- Fragment；

**React v16.3**

- createContext、createRef、forwardRef、生命周期函数的更新、Strict Mode；

**React v16.4**

- Pointer Events、update getDerivedStateFromProps；

**React v16.5**

- Profiler；

**React v16.6**

- memo、lazy、Suspense、static contextType、static getDerivedStateFromError()；

**React v16.7**

- Hooks；

**React v16.8**

- Concurrent Rendering；

**React v16.9（~mid 2019）**

- Suspense for Data Fetching；

下面将按照上述的 React16 更新路径对每个新特性进行详细或简短的解析。