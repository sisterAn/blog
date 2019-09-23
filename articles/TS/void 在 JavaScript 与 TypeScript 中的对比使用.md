

如果你对强类型有所了解的话，你一定听说过或使用过 `void` ，它代表调用的方法不存在任何返回内容。

`void` 同样在 `JavaScript` 以及 `TypeScript` 中也存在，只不过，在 `JavaScript` 中，`void` 作为一个操作符，而在 `TypeScript` 中，`void` 作为一个基本类型，并且它在 `JavaScript` 和 `TypeScript` 中的使用，也是有所不同的。

### JS 中的 void

在 `JavaScript` 中 `void` 是计算其临近的表达式的运算符。无论计算哪个表达式，`void` 始终返回 `undefined` 。

```js
let i = void 2; // i === undefined
```

直接定义 `let i` 不好吗，为什么要使用 `void 2` ？

首先，在 `JavaScript`早期，人们能够重写 `undefined` 并赋予它一个真实的值。而 `void` 则能返回真正的 `undefined` 。

其次，它是调用立即执行函数的好方法：

```js
void function() {
  console.log('What')
}()
```

在不污染全局命名空间的情况下：

```js
void function aRecursion(i) {
  if(i > 0) {
    console.log(i--)
    aRecursion(i)
  }
}(3)

console.log(typeof aRecursion) // undefined
```

由于 `void` 始终返回 `undefined` ，并且 `void` 始终计算其临近的表达式，因此你可以通过返回一个 `void` 函数，即不会返回任何具体值，但仍调用 `void` 临近函数，例如：

```js
// 返回 undefined，但仍执行了 nextCallback()
function middleware(nextCallback) {
  if(conditionApplies()) {
    return void nextCallback();
  }
}
```

这就是 `void` 最重要的功能：它是应用的安全门。当函数需要始终返回 `undefined` 时，可以使用 `void` 。

```js
button.onclick = () => void doSomething();
```



### TS 中的 void

`TypeScript` 中的 `void` 是 `undefined` 的子类型。 `JavaScript` 中的函数总是返回一个值，或 `undefined` ：

```js
function iHaveNoReturnValue(i) {
  console.log(i)
} // returns undefined
```

由于 `JavaScript` 没有返回值的函数总是返回 `undefined` ，而 `void` 在 `JavaScript` 中又总是返回 `undefined` ，因此 `TypeScript` 中的 `void` 是一种类型，它代表此函数返回 `undefined`：

```js
declare function iHaveNoReturnValue(i: number): void
```

`void` 也可以用于参数列表或其他声明中。 它唯一可以传递的值是 `undefined`：

```js
declare function iTakeNoParameters(x: void): void

iTakeNoParameters() // 👍
iTakeNoParameters(undefined) // 👍
iTakeNoParameters(void 2) // 👍
```

因此 `void` 和 `undefined` 几乎相同。 不过，两者之间也有一点小小的但很重要的差异： **`void` 作为返回类型可以用不同的类型代替，以允许使用高级回调模式** ：

```js
function doSomething(callback: () => void) {
  let c = callback() // 在这里, callback 始终返回 undefined
  //c 也是 undefiend 类型
}

// 这个函数返回一个 number
function aNumberCallback(): number {
  return 2;
}

// 这样使用，确保类型安全 👍 
doSomething(aNumberCallback) 
```

这是期望的行为，经常在 `JavaScript` 应用程序中使用。

如果你想要函数仅仅返回 `undefined` ，请调整你的回调方法签名：

```js
// function doSomething(callback: () => void) {
// 调整为
function doSomething(callback: () => undefined) { /* ... */ }

function aNumberCallback(): number { return 2; }

// 类型不匹配 💥 
doSomething(aNumberCallback) 
```

至此，你将会更好的使用 `void` 。

### 翻译自

https://fettblog.eu/void-in-javascript-and-typescript/