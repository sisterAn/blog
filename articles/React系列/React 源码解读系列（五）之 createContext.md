## context

### 一、初识 context 

在典型的 React 应用中， **数据** 是通过 props 属性显式的由父及子进行 **传递** 的，但这种方式，对于复杂情况（例如，跨多级传递，多个组件共享）来说，是极其繁琐的。

- 第一种解决方式是： **组件的封装与组合，将组件自身传递下去**

  在项目中，我们在父层获取数据，不同层级的子组件访问时，我们可以使用 **将子组件的公共组件封装，将公共组件传递下去** 。例如

  ```js
  function Page(props) {
    // 你可以传递多个子组件，甚至会为这些子组件（children）封装多个单独的接口（slots）
    const localeCom = (
     	<span>{ props.locale }</span>
    ) 
    return (
      <Content localeCom={localeCom} />
    );
  } // 这种情况下，只有顶层 Page 才知道 localeCom 的具体实现，实现了组件的控制反转
  
  function Content(props) {
    return (
      <div>
        <FirstComponent localeCom={localeCom} />
      </div>
    );
  }
  
  class FirstComponent extends React.Component {
    render() {
      return (
        <div>FirstComponent: {this.props.localeCom}</div>
      );
    }
  }
  ```

  这种对组件的 **控制反转** 减少了在应用中要传递的 props 数量，这在很多场景下会使得你的 **代码更加干净** ，使你对根组件有更多的把控。但是，这并不适用于每一个场景： **将逻辑提升到组件树的更高层次来处理，会使得这些高层组件变得更复杂，并且会强行将低层组件提到高层实现，这很多时候有违常理**。

- 第二种解决方式是：**context**

  **context** 提供了一种在 **组件之间共享此类值** 的方式，使得我们无需每层显式添加 props 或传递组件 ，就能够实现在 **组件树中传递数据** 。

  ```js
  // Context 可以让我们无须显式地传遍每一个组件，就能将值深入传递进组件树。
  // 为当前的 locale 创建一个 context（默认值为 anan）。
  // context 会在每一个创建或使用 context 的组件上引入，所以，最好在单独一个文件中定义
  // 这里只做演示
  const LocaleContext = React.createContext('anan');
  
  class App extends React.Component {
    render() {
      // 使用一个 Provider 来将当前的 name 传递给以下的组件树。
      // 无论多深，任何组件都能读取这个值。
      // 在这个例子中，我们将 “ananGe” 作为当前的值传递下去。
      return ( // Provider 接收一个 value 属性，传递给消费组件
        <LocaleContext.Provider value="ananGe">
          <Content />
        </LocaleContext.Provider>
      );
    }
  }
  
  // 中间的组件再也不必指明往下传递 locale 了。
  // LocaleContext 分别在 FirstComponent 组件与 SecondComponent 的子组件 SubComponent 中使用
  function Content(props) {
    return (
      <div>
        <FirstComponent />
        <SecondComponent />
      </div>
    );
  }
  
  // 第一个子组件
  class FirstComponent extends React.Component {
    // 指定 contextType 读取当前的 locale context。
    // React 会往上找到最近的 locale Provider，然后使用它的值。
    // 在这个例子中，当前的 locale 值为 ananGe
    static contextType = LocaleContext;
    render() {
      return (
        <div>FirstComponent: <span>{ this.context }</span></div>
      );
    }
  }
  
  // 第二个子组件（中间件）
  function SecondComponent(props) {
    return (
      <div>
        <SubComponent />
      </div>
    );
  }
  // SecondComponent 的子组件
  class SubComponent extends React.Component {
    static contextType = LocaleContext;
    render() {
      return (
        <div>SubComponent: <span>{ this.context }</span></div>
      ); // this.context 为传递过来的 value 值
    }
  }
  ```

  **注意：在大多数情况下，context 一般用来做 中间件 的方式使用，例如 redux。**

  - **React.createContext**

    ```js
    const LocaleContext = React.createContext(defaultValue);
    // 创建一个 Context 对象。当 React 渲染一个订阅了这个 Context 对象的组件，这个组件会从组件树中 离自身最近 的那个匹配的 Provider 中读取到当前的 context 值。
    ```

  - **Context.Provider**

    ```js
    <LocaleContext.Provider value={/* 某个值 */}>
    ```

    - Provider 接收一个 **value** 属性，传递给消费组件。
    - 一个 Provider 可以和 **多个消费组件** 有对应关系。多个 Provider 可以 **嵌套使用** ，里层的会覆盖外层的数据。
    - 当 Provider 的 value 值发生变化时，它内部的所有消费组件都会 **重新渲染** 。
    - Provider 及其内部 consumer 组件都 **不受制于 shouldComponentUpdate**  函数，因此当 consumer 组件在其祖先组件退出更新的情况下也能更新。
    - 通过新旧值检测来确定变化，使用了与 **Object.is** （[Object.is MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is)） 相同的算法。

  - **Class.contextType**

    ```js
    // 挂载在 SubComponent 上的 contextType 属性会被重赋值为 LocaleContext
    SubComponent.contextType = LocaleContext;
    // 使用 this.context 来消费最近 Context 上的那个值
    let value = this.context;
    
    // 你可以使用这种方式来获取 context value，也可以使用 Context.Consumer 函数式订阅获取
    ```

  - **Context.Consumer**

    ```js
    // 在函数式组件中完成订阅 context
    <LocaleContext.Consumer>
      {value => /* 基于 context 值进行渲染*/}
    </LocaleContext.Consumer>
    ```

### 二、深入 context

- 当 Provider 的 value 值发生变化时，它内部的所有消费组件都会 **重新渲染**
- 当需要在 **Consumer 中触发 Provider 执行更新 context value 操作** 时，可以通过 context 传递一个 **函数** ，使得 consumer 组件触发更新 context
- 多个 context 可以 **嵌套使用**
- 注意： **不要在 Provider value 直接赋值** （`<LocaleProvider.Provider value={{name: 'AnGe'}}>`），因为这样会导致，每次 Provider 的父组件进行重渲染时，都会导致 Consumer 组件中重新渲染，因为 `value` 属性总是被赋值为新的对象（Object.is 新旧值检测）
- 

**locale-context.js**

```js
export const locales = {
  An: {
    name: 'an',
    color: 'red',
  },
  AnGe: {
    name: 'anGe',
    color: 'green',
  },
}

export const LocaleContext = React.createContext(
  locales.An // 默认值
)

// 确保传递给 createContext 的默认值数据结构是调用的组件（consumers）所能匹配的！
export const AddressContext = React.createContext({
  address: 'Shanghai',
  updateAddress: () => {}, // Consumer 更新 Provider value 函数
})
```

**app.js**

```js
import { locales, LocaleContext, AddressContext } from './locale-context';
import SubComponent from './SubComponent';

class App extends React.Component {
  state = {
    locale: locales.An,
    address: 'Beijing',
  }
  // 更新 locale 函数
  changePerson = () => {
    this.setState(state => ({
      locale:
        state.locale === locales.An
          ? locales.AnGe
          : locales.An,
    }));
  }
  // 更新 address 函数
  updateAddress = () => {
    this.setState(state => ({
      address:
        state.address === 'Beijing'
          ? 'Shanghai'
          : 'Beijing',
    }));
  }

  render() {
    const {
      locale,
      address,
    } = this.state
    
    // addressValue 包含了 updateAddress 更新函数
    const addressValue = {
      address: 'Beijing',
      updateAddress: this.updateAddress
    }
    return (
      <div>
        // 在 LocaleProvider 内部的 SubComponent 组件使用 state 中的 locale 值
        // 当 LocaleProvider 的 value 值发生变化时，它内部的所有消费组件都会重新渲染
        <LocaleProvider.Provider value={locale}>
          // addressValue 都被传递进 AddressContext.Provider
          <AddressContext.Provider value={addressValue}>
            <Toolbar changePerson={this.changePerson} />
          </AddressContext.Provider>
        </LocaleProvider.Provider>
        // 而外部的组件，没有被 LocaleProvider.Provider 包裹，则使用默认的 locale 值
        <div>
          <SubComponent />
        </div>
      </div>
    );
  }
}

// 一个使用 SubComponent 的中间组件
function Toolbar(props) {
  return (
    <SubComponent onClick={props.changePerson}>
      Change Person
    </SubComponent>
  );
}

ReactDOM.render(<App />, document.root);
```

**SubComponent.js**

```js
import { LocaleContext, AddressContext } from './locale-context';

class SubComponent extends React.Component {
  
  render() {
    const props = this.props;
    return ( // 一个组件可能会消费多个 context
      <LocaleContext.Consumer>
        {locale => (
          <div
            {...props}
            style={{color: locale.color}}
          >
            {locale.name}
            <AddressContext.Consumer> // AddressContext.Consumer 可以从 context 中获取到 address 值 与 updateAddress 函数
        	  {(address, updateAddress) => ( // 点击 button，执行 AddressContext.Provider 的 updateAddress 函数，更新 address
        	    <button onClick={updateAddress}>{address}</button>
    		  )}
            </AddressContext.Consumer>
          </div>
        )}
      </LocaleContext.Consumer>
    );
  }
}

export default SubComponent;
```

### 三、源码解读

```js
export function createContext<T>(
  defaultValue: T, // context 默认值
  calculateChangedBits: ?(a: T, b: T) => number, // 计算新老 context 变化函数
): ReactContext<T> {
  if (calculateChangedBits === undefined) {
    calculateChangedBits = null;
  } else {
    if (__DEV__) {
      warningWithoutStack(
        calculateChangedBits === null ||
          typeof calculateChangedBits === 'function',
        'createContext: Expected the optional second argument to be a ' +
          'function. Instead received: %s',
        calculateChangedBits,
      );
    }
  }

  // 声明了一个 context 对象
  const context: ReactContext<T> = {
    $$typeof: REACT_CONTEXT_TYPE,
    _calculateChangedBits: calculateChangedBits,
    // As a workaround to support multiple concurrent renderers, we categorize
    // some renderers as primary and others as secondary. We only expect
    // there to be two concurrent renderers at most: React Native (primary) and
    // Fabric (secondary); React DOM (primary) and React ART (secondary).
    // Secondary renderers store their context values on separate fields.
    _currentValue: defaultValue, // 用来记录 context 最新值，当 Provider value 更新时，同步到 _currentValue 上
    _currentValue2: defaultValue,
    // Used to track how many concurrent renderers this context currently
    // supports within in a single renderer. Such as parallel server rendering.
    _threadCount: 0,
    // These are circular
    Provider: (null: any), // context Provider
    Consumer: (null: any), // context Consumer
  };

  context.Provider = { // context.Provider 的 _context 为 context
    $$typeof: REACT_PROVIDER_TYPE,
    _context: context, 
  };

  let hasWarnedAboutUsingNestedContextConsumers = false;
  let hasWarnedAboutUsingConsumerProvider = false;

  if (__DEV__) {
    // A separate object, but proxies back to the original context object for
    // backwards compatibility. It has a different $$typeof, so we can properly
    // warn for the incorrect usage of Context as a Consumer.
    const Consumer = { //Consumer 的 _context 也为 context
      $$typeof: REACT_CONTEXT_TYPE,
      _context: context, 
      _calculateChangedBits: context._calculateChangedBits,
    };
    // $FlowFixMe: Flow complains about not setting a value, which is intentional here
    Object.defineProperties(Consumer, {
      Provider: {
        get() {
          if (!hasWarnedAboutUsingConsumerProvider) {
            hasWarnedAboutUsingConsumerProvider = true;
            warning(
              false,
              'Rendering <Context.Consumer.Provider> is not supported and will be removed in ' +
                'a future major release. Did you mean to render <Context.Provider> instead?',
            );
          }
          return context.Provider;
        },
        set(_Provider) {
          context.Provider = _Provider;
        },
      },
      _currentValue: {
        get() {
          return context._currentValue;
        },
        set(_currentValue) {
          context._currentValue = _currentValue;
        },
      },
      _currentValue2: {
        get() {
          return context._currentValue2;
        },
        set(_currentValue2) {
          context._currentValue2 = _currentValue2;
        },
      },
      _threadCount: {
        get() {
          return context._threadCount;
        },
        set(_threadCount) {
          context._threadCount = _threadCount;
        },
      },
      Consumer: {
        get() {
          if (!hasWarnedAboutUsingNestedContextConsumers) {
            hasWarnedAboutUsingNestedContextConsumers = true;
            warning(
              false,
              'Rendering <Context.Consumer.Consumer> is not supported and will be removed in ' +
                'a future major release. Did you mean to render <Context.Consumer> instead?',
            );
          }
          return context.Consumer;
        },
      },
    });
    // $FlowFixMe: Flow complains about missing properties because it doesn't understand defineProperty
    context.Consumer = Consumer;
  } else {
    context.Consumer = context;
  }
  // Provider 与 Consumer 均指向 context，也就是说，Provider 与 Consumer 使用同一个变量 _currentValue，当 Consumer 需要渲染时，直接从自身取得 context 最新值 _currentValue 去渲染
  if (__DEV__) {
    context._currentRenderer = null;
    context._currentRenderer2 = null;
  }

  return context;
}
```

