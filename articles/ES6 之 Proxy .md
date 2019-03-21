Proxy，代理，是ES6新增的功能，可以理解为代理器（即由它代理某些操作）。

Proxy 对象用于定义或修改某些操作的自定义行为，可以在外界对目标对象进行访问前，对外界的访问进行改写。

### 1. Proxy 定义

```js
var proxy = new Proxy(target, handler)
```

`new Proxy()`表示生成一个 Proxy 实例

- target：目标对象
- handler：一个对象，其属性是当执行一个操作时定义代理的行为的函数。

**注意：要实现拦截操作，必须是对 Proxy 实例进行操作，而不是针对目标对象 target 进行操作。**

首先，看个例子：

```js
let handler = {
    get: function(target, key, receiver) {
        console.log(`getter ${key}!`)
        return Reflect.get(target, key, receiver)
    },
    set: function(target, key, value, receiver) {
    	console.log(`setter ${key}=${value}`)
		return Reflect.set(target, key, value, receiver)
	}
}
var obj = new Proxy({}, handler)
obj.a = 1 // setter a=1
obj.b = undefined // setter b=undefined

console.log(obj.a, obj.b) 
// getter a!
// getter b!
// 1 undefined

console.log('c' in obj, obj.c)	
// getter c!
// false undefined
```

在这个例子中，proxy 拦截了get和set操作。

再看一个例子：

```js
let handler = {
    get: function(target, key, receiver) {
        return 1
    },
  	set: function (target, key, value, receiver) {
    	console.log(`setting ${key}!`);
    	return Reflect.set(target, key, value, receiver);
  	}
}
var obj = new Proxy({}, handler)
obj.a = 5 // setting a!
console.log(obj.a) // 1
```

则由上面代码看出：**Proxy 不仅是拦截了行为，更是用自己定义的行为覆盖了组件的原始行为**。

**若`handler = {}`，则代表 Proxy 没有做任何拦截，访问 Proxy 实例就相当于访问 target 目标对象。**这里不再演示，有兴趣的可以自己举例尝试。

### 2. Proxy handler方法（拦截方法）

- `get(target, key, receiver)`：拦截 target 属性的读取
- `set(target, key, value, receiver)`：拦截 target 属性的设置
- `has(target, key)`：拦截 `key in proxy` 的操作，并返回是否存在（boolean值）
- `deleteProperty(target, key)`：拦截 `delete proxy[key]`的操作，并返回结果（boolean值）
- `ownKeys(target)`：拦截`Object.getOwnPropertyName(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for ... in`循环。并返回目标对象所有自身属性的属性名数组。注意：**`Object.keys()`的返回结果数组中只包含目标对象自身的可遍历属性**
- `getOwnPropertyDescriptor(target, key)`：拦截 `Object.getOwnPropertyDescriptor(proxy, key)`，返回属性的描述对象
- `defineProperty(target, key, desc)`：拦截`Object.defineProperty(proxy, key, desc)`、`Object.defineProperties(proxy, descs)`，返回一个 boolean 值
- `preventExtensions(target)`：拦截`Object.preventExtensions(proxy)`，返回一个 boolean 值
- `getPrototypeOf(target)`：拦截`Object.getPrototypeOf(proxy)`，返回一个对象
- `isExtensible(target)`：拦截`Object.isExtensible(proxy)`，返回一个 boolean 值
- `setPrototypeOf(target, key)`：拦截`Object.setPrototypeOf(proxy, key)`，返回一个 boolean 值。如果目标对象是函数，则还有两种额外操作可以被拦截
- `apply(target, object, args)`：拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`
- `construct(target, args)`：拦截 Proxy 实例作为构造函数调用的操作，比如`new proxy(...args)`

总共 13 个拦截方法，下面进行简要举例说明，更多可见阮一峰老师的 [《ECMAScript 6 入门》](https://link.juejin.im/?target=http%3A%2F%2Fes6.ruanyifeng.com%2F%23docs%2Fproxy)

#### 1. get，set 

`get`方法用于拦截某个属性的读取操作，可以接受三个参数，依次为目标对象、属性名和 proxy 实例本身（严格地说，是操作行为所针对的对象），其中最后一个参数可选。

`get`方法可以继承。

```js
let proxy = new Proxy({foo: 1}, {
    get(target, key, receiver) {
        if (key in target) {
            console.log(`getter ${key}!`)
            return target[key]
        } else {
            throw new ReferenceError(`Prpperty ${key} does not exist.`)
        }
    },
    set: function(target, key, value, receiver) {
        console.log(`setter ${key}!`)
        target[key] = value;
    }
})

let obj = Object.create(proxy)
console.log(obj.foo) 
// getter foo!
// 1
```

上面代码中，拦截操作定义在 Prototype 对象上，所以如果读取 obj 对象继承的属性，拦截将会生效。

```js
var pipe = (function () {
    return function (value) {
        var funcStack = []
        var proxy = new Proxy({}, {
            get: function (target, key) {
                if (key === 'get') {
                    return funcStack.reduce(function (val, fn) {
                        return fn(val)
                    }, value)
                }
                funcStack.push(window[key])
                return proxy
            }
        })
        return proxy
    }
}())

var double = n => n * 2
var pow = n => n * n
var reverseInt = n => n.toString().split("").reverse().join("") | 0
pipe(3).double.pow.reverseInt.get
```

#### 2. has

拦截 propKey in proxy 的操作，返回一个布尔值。

```js
// 使用 has 方法隐藏某些属性，不被 in 运算符发现
var handler = {
    has (target, key) {
        if (key.startsWith('_')) {
            return false;
        }
        return key in target;
    }
};
var foo = { _name: 'foo', name: 'foo' };
var proxy = new Proxy(foo, handler);
console.log('_name' in proxy); // false
console.log('name' in proxy); // true
```

#### 3. ownKeys

拦截自身属性的读取操作。并返回目标对象所有自身属性的属性名数组。具体返回结果可结合 MDN [属性的可枚举性和所有权](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)

- `Object.getOwnPropertyName(proxy)`
- `Object.getOwnPropertySymbols(proxy)`
- `Object.keys(proxy)`
- `for ... in`循环

```js
let target = {
  _foo: 'foo',
  _bar: 'bar',
  name: 'An'
};

let handler = {
  ownKeys (target) {
    return Reflect.ownKeys(target).filter(key => key.startsWith('_'));
  }
};

let proxy = new Proxy(target, handler);
for (let key of Object.keys(proxy)) {
  console.log(target[key]);
}
// "An"
```



#### 4. apply

apply 拦截 Proxy 实例作为函数调用的操作，比如函数的调用（`proxy(...args)`）、call（`proxy.call(object, ...args)`）、apply（`proxy.apply(...)`）等。

```js
var target = function () { return 'I am the target'; };
var handler = {
  apply: function () {
    return 'I am the proxy';
  }
};

var proxy = new Proxy(target, handler);

proxy();
// "I am the proxy"
```



Proxy 方法太多，这里只是将常用的简要介绍，更多请看阮一峰老师的 [《ECMAScript 6 入门》](https://link.juejin.im/?target=http%3A%2F%2Fes6.ruanyifeng.com%2F%23docs%2Fproxy)