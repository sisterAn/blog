### 四、call、apply、bind 区别与实现

`call` 、`apply` 都是为了解决 `this` 的指向。作用是相同的，只是传参的方式不同。

除了第一个参数外，`call` 可以接收一个参数列表，`apply` 只能接收一个参数数组。

```js
let a = {
    value: 1
}
function getValue(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value)
}
getValue.call(a, 'yck', '24')
getValue.apply(a, ['yck', '24'])
```

模拟实现 `call` 、`apply`

可以从一下几点考虑实现

- 不传入第一个参数，那么默认为 `window`
- 改变了 `this` 指向，让新的对象可以执行该函数，那么思路是否可以变成新的对象添加一个函数，然后再执行完成后删除



#### 1. 模拟实现 call

```js
Function.prototype.myCall = function(context) {
    var context = context || windows
    // 给 context 添加一个属性
    // getValue.call(a, 'yck', '24') => a.fn = gatValue
    context.fn = this
    // 将 context 后面的参数取出来
    var args = [...arguments].slice(1)
    // getValue.call(a, 'yck', '24') => a.fn('yck', '24')
    var result = context.fn(...args)
    // 删除 fn
    delete context.fn
    return result
}
```

以上就是 `call` 的思路， `apply` 的实现也类似



#### 2. 模拟实现 apply

```js
Function.prototype.myApply = function(context) {
    var context = context || window
    context.fn = this
    
    var result
    // 需要判断是否存储第二个参数
    // 如果存在，就将第二个参数展开
    if (arguments[1]) {
        result = context.fn(...arguments[1])
    } else {
        result = context.fn()
    }
    
    delete context.fn
    return result
}
```



#### 3. 模拟实现 bind

调用绑定函数通常会导致执行包装函数，绑定函数有以下内部属性：

- `[[BoundTargetFunction]]`：包装的函数（ `function` ）
- `[[BoundThis]]`：调用包装函数的 this 值
- `[[BoundArguments]]`：值列表，其元素用于对包装函数调用的第一个参数
- `[[Call]]`：执行与此对象关联的代码。通过函数调用表达式调用，内部方法的参数是 `this` 值和参数列表

当调用绑定函数时，它调用 `[[BoundTargetFunction]]` 上的内部方法 `[[Call]]` ，后跟参数 **Call(boundThis, args)** 。其中，`boundThis` 是 `[[BoundThis]]` ，`args` 是 `[[BoundArguments]]`，后跟函数调用传递的参数。

```js
Function.prototype.myBind = function(context) {
    if (typeof this !== 'function') {
        throw new TypeError('error')
    }
    var _this = this
    var args = [...arguments].slice(1)
    // 返回一个函数
    return function Fun() {
        // 因为返回一个函数， 我们可以 new Fun(), 所以需要判断
        if (this instanceof Fun) {
            return new _this(...args, ...arguments)
        }
        return _this.call(context, ...args, ...arguments)
    }
}
```

`bind` 和其他两个方法作用是一致的，只是该方法会返回一个函数，并且我们可以通过 `bind` 来实现柯里化。

