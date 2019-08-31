相对于ES5继承，ES6继承就简单的多，关于继承机制的设计思想，请参见 [Javascript继承机制的设计思想](http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html) 。



## ES5 继承

定义一个父类

```js
function SuperType () {
  // 属性
  this.name = 'SuperType';
}
// 原型方法
SuperType.prototype.sayName = function() {
  return this.name;
};
```



### 一、 原型链继承

**将父类的实例作为子类的原型**

```js
// 父类
function SuperType () {
  this.name = 'SuperType'; // 父类属性
}
SuperType.prototype.sayName = function () { // 父类原型方法
  return this.name;
};

// 子类
function SubType () {
  this.subName = "SubType"; // 子类属性
};

SubType.prototype = new SuperType(); // 重写原型对象，代之以一个新类型的实例
// 这里实例化一个 SuperType 时， 实际上执行了两步
// 1，新创建的对象复制了父类构造函数内的所有属性及方法
// 2，并将原型 __proto__ 指向了父类的原型对象

SubType.prototype.saySubName = function () { // 子类原型方法
  return this.subName;
}

// 子类实例
let instance = new SubType();

// instanceof 通过判断对象的 prototype 链来确定对象是否是某个类的实例
instance instanceof SubType; // true
instance instanceof SuperType; // true

// 注意
SubType instanceof SuperType; // false
SubType.prototype instanceof SuperType ; // true
```

![原型链继承](https://user-images.githubusercontent.com/19721451/56806156-b3d77580-685d-11e9-9931-ded6cc8c0229.png)

- **特点**：利用原型，让一个引用类型继承另一个引用类型的属性及方法

- **优点**：**继承了父类的模板，又继承了父类的原型对象**

- **缺点**：

  - 可以在子类构造函数中，为子类实例增加实例属性。如果要**新增原型属性和方法**，则必须放在 `SubType.prototype = new SuperType('SubType');` 这样的语句**之后**执行。

  - 无法实现多继承

  - 来自原型对象的所有属性被**所有实例共享**

    ```js
    // 父类
    function SuperType () {
      this.colors = ["red", "blue", "green"];
      this.name = "SuperType";
    }
    // 子类
    function SubType () {}
    
    // 原型链继承
    SubType.prototype = new SuperType();
    
    // 实例1
    var instance1 = new SubType();
    instance1.colors.push("blcak");
    instance1.name = "change-super-type-name";
    console.log(instance1.colors); // ["red", "blue", "green", "blcak"]
    console.log(instance1.name); // change-super-type-name
    // 实例2
    var instance2 = new SubType();
    console.log(instance2.colors); // ["red", "blue", "green", "blcak"]
    console.log(instance2.name); // SuperType
    ```

    ![prototype-shared](https://user-images.githubusercontent.com/19721451/56806180-bfc33780-685d-11e9-8108-71cd78684d64.png)

    **注意**：更改 `SuperType` **引用类型属性**时，会使 `SubType` 所有实例共享这一更新。基础类型属性更新则不会。

  - **创建子类实例时，无法向父类构造函数传参**，或者说是，没办法在不影响所有对象实例的情况下，向超类的构造函数传递参数



### 二、 构造继承

**基本思想**：在子类型的构造函数内部调用父类型构造函数。

**注意**：函数只不过是在特定环境中执行代码的对象，所以这里使用 apply/call 来实现。

**使用父类的构造函数来增强子类实例，等于是复制父类的实例属性给子类（没用到原型）**

```js
// 父类
function SuperType (name) {
  this.name = name; // 父类属性
}
SuperType.prototype.sayName = function () { // 父类原型方法
  return this.name;
};

// 子类
function SubType () {
  // 调用 SuperType 构造函数
  SuperType.call(this, 'SuperType'); // 在子类构造函数中，向父类构造函数传参
  // 为了保证子父类的构造函数不会重写子类的属性，需要在调用父类构造函数后，定义子类的属性
  this.subName = "SubType"; // 子类属性
};
// 子类实例
let instance = new SubType(); // 运行子类构造函数，并在子类构造函数中运行父类构造函数，this绑定到子类
```

![构造函数继承](https://user-images.githubusercontent.com/19721451/56806216-de293300-685d-11e9-9b93-18db6e673d2a.png)

- **优点**：**解决了1中子类实例共享父类引用对象的问题，实现多继承，创建子类实例时，可以向父类传递参数**
- **缺点**：
  - 实例并不是父类的实例，只是子类的实例
  - **只能**继承父类的实例属性和方法，**不能**继承原型属性/方法
  - **无法实现函数复用**，每个子类都有父类实例函数的副本，影响性能



#### 扩展一：new

**new 关键字**创建的对象**实际上是对新对象 this 的不断赋值，并将 prototype 指向类的 prototype 所指向的对象**。

```js
var SuperType = function (name) {
    var nose = 'nose' // 私有属性
    function say () {} // 私有方法
    
    // 特权方法
    this.getName = function () {} 
    this.setName = function () {}
    
    this.mouse = 'mouse' // 对象公有属性
    this.listen = function () {} // 对象公有方法
    
    // 构造器
    this.setName(name)
}

SuperType.age = 10 // 类静态公有属性（对象不能访问）
SuperType.read = function () {} // 类静态公有方法（对象无法访问）

SuperType.prototype = { // 对象赋值（也可以一一赋值）
    isMan: 'true', // 公有属性
    write: function () {} // 公有方法
}

var instance = new SuperType()
```

![new](https://user-images.githubusercontent.com/19721451/58102668-3f051a80-7c14-11e9-8314-e3e8fd0ac280.png)

所以类的构造函数内定义的 **私有变量或方法** ，以及类定义的 **静态公有属性及方法** ，在 **new** 的实例对象中都将 **无法访问** 。



### 三. 组合继承

顾名思义，组合继承就是将原型链继承与构造函数继承组合在一起，从而发挥两者之长的一种继承模式。

**基本思想**：使用**原型链**继承使用对原型属性和方法的继承，通过**构造函数**继承来实现对实例属性的继承。这样既能通过在原型上定义方法实现函数复用，又能保证每个实例都有自己的属性。

**通过调用父类构造，继承父类的属性并保留传参的优点，然后通过将父类实例作为子类原型，实现函数复用**

```js
// 父类
function SuperType (name) {
  this.colors = ["red", "blue", "green"];
  this.name = name; // 父类属性
}
SuperType.prototype.sayName = function () { // 父类原型方法
  return this.name;
};

// 子类
function SubType (name, subName) {
  // 调用 SuperType 构造函数
  SuperType.call(this, name); // ----第二次调用 SuperType----
  this.subName = subName;
};

// ----第一次调用 SuperType----
SubType.prototype = new SuperType(); // 重写原型对象，代之以一个新类型的实例

SubType.prototype.constructor = SubType; // 组合继承需要修复构造函数指向
SubType.prototype.saySubName = function () { // 子类原型方法
  return this.subName;
}

// 子类实例
let instance = new SubType('An', 'sisterAn')
instance.colors.push('black')
console.log(instance.colors) // ["red", "blue", "green", "black"]
instance.sayName() // An
instance.saySubName() // sisterAn

let instance1 = new SubType('An1', 'sisterAn1')
console.log(instance1.colors) //  ["red", "blue", "green"]
instance1.sayName() // An1
instance1.saySubName() // sisterAn1
```

![组合继承1](https://user-images.githubusercontent.com/19721451/58102646-33195880-7c14-11e9-9b36-809b8996d90f.png)

第一次调用 `SuperType` 构造函数时，`SubType.prototype` 会得到两个属性`name`和`colors`；当调用 `SubType` 构造函数时，第二次调用 `SuperType` 构造函数，这一次又在新对象属性上创建了 `name`和`colors`，这两个属性就会屏蔽原型对象上的同名属性。

```js
// instanceof：instance 的原型链是针对 SuperType.prototype 进行检查的
instance instanceof SuperType // true
instance instanceof SubType // true

// isPrototypeOf：instance 的原型链是针对 SuperType 本身进行检查的
SuperType.prototype.isPrototypeOf(instance) // true
SubType.prototype.isPrototypeOf(instance) // true
```

![组合继承2](https://user-images.githubusercontent.com/19721451/58102635-2d237780-7c14-11e9-981c-6f638830db10.png)

- **优点**：**弥补了方式2的缺陷，可以继承实例属性/方法，也可以继承原型属性/方法，不存在引用属性共享问题，可传参，可复用**
- **缺点**：
  - 调用了两次父类构造函数，生成了**两份实例**（子类实例将子类原型上的那份屏蔽了）



### 四. 寄生组合继承

在组合继承中，调用了两次父类构造函数，这里  **通过通过寄生方式，砍掉父类的实例属性，这样，在调用两次父类的构造的时候，就不会初始化两次实例方法/属性，避免的组合继承的缺点**

**主要思想**：借用 **构造函数** 继承 **属性** ，通过 **原型链的混成形式** 来继承 **方法**

```js
// 父类
function SuperType (name) {
  this.colors = ["red", "blue", "green"];
  this.name = name; // 父类属性
}
SuperType.prototype.sayName = function () { // 父类原型方法
  return this.name;
};

// 子类
function SubType (name, subName) {
  // 调用 SuperType 构造函数
  SuperType.call(this, name); // ----第二次调用 SuperType，继承实例属性----
  this.subName = subName;
};

// ----第一次调用 SuperType，继承原型属性----
SubType.prototype = Object.create(SuperType.prototype)

SubType.prototype.constructor = SubType; // 注意：增强对象

let instance = new SubType('An', 'sisterAn')
```
![寄生组合](https://user-images.githubusercontent.com/19721451/58176578-26a90480-7cd5-11e9-83d3-ff7e1dfe58ba.png)

**优点**：

- 只调用一次 `SuperType` 构造函数，只创建一份父类属性
- 原型链保持不变
- 能够正常使用 `instanceof` 与 `isPrototypeOf`



### 五. 原型式继承

**实现思路就是将子类的原型设置为父类的原型**

```js
// 父类
function SuperType (name) {
  this.colors = ["red", "blue", "green"];
  this.name = name; // 父类属性
}
SuperType.prototype.sayName = function () { // 父类原型方法
  return this.name;
};

/** 第一步 */
// 子类，通过 call 继承父类的实例属性和方法，不能继承原型属性/方法
function SubType (name, subName) {
  SuperType.call(this, name); // 调用 SuperType 的构造函数，并向其传参 
  this.subName = subName;
}

/** 第二步 */
// 解决 call 无法继承父类原型属性/方法的问题
// Object.create 方法接受传入一个作为新创建对象的原型的对象，创建一个拥有指定原型和若干个指定属性的对象
// 通过这种方法指定的任何属性都会覆盖原型对象上的同名属性
SubType.prototype = Object.create(SuperType.prototype, { 
  constructor: { // 注意指定 SubType.prototype.constructor = SubType
    value: SubType,
    enumerable: false,
    writable: true,
    configurable: true
  },
  run : { 
    value: function(){ // override
      SuperType.prototype.run.apply(this, arguments); 
      	// call super
      	// ...
    },
    enumerable: true,
    configurable: true, 
    writable: true
  }
}) 

/** 第三步 */
// 最后：解决 SubType.prototype.constructor === SuperType 的问题
// 这里，在上一步已经指定，这里不需要再操作
// SubType.prototype.constructor = SubType;

var instance = new SubType('An', 'sistenAn')
```

![原型继承1](https://user-images.githubusercontent.com/19721451/58176560-1e50c980-7cd5-11e9-9fc9-33f84e05f0a6.png)

如果希望能 **多继承** ，可使用 **混入** 的方式

```js
// 父类 SuperType
function SuperType () {}
// 父类 OtherSuperType
function OtherSuperType () {}

// 多继承子类
function AnotherType () {
    SuperType.call(this) // 继承 SuperType 的实例属性和方法
    OtherSuperType.call(this) // 继承 OtherSuperType 的实例属性和方法
}

// 继承一个类
AnotherType.prototype = Object.create(SuperType.prototype);

// 使用 Object.assign 混合其它
Object.assign(AnotherType.prototype, OtherSuperType.prototype);
// Object.assign 会把  OtherSuperType 原型上的函数拷贝到 AnotherType 原型上，使 AnotherType 的所有实例都可用 OtherSuperType 的方法

// 重新指定 constructor
AnotherType.prototype.constructor = AnotherType;

AnotherType.prototype.myMethod = function() {
     // do a thing
};

let instance = new AnotherType()
```

**最重要的部分是**：

- `SuperType.call` 继承实例属性方法
- 用 `Object.create()` 来继承原型属性与方法
- 修改 `SubType.prototype.constructor `的指向



## ES6 继承

首先，实现一个简单的 ES6 继承：

```js
class People {
    constructor(name) {
        this.name = name
    }
    run() { }
}

// extends 相当于方法的继承
// 替换了上面的3行代码
class Man extends People {
    constructor(name) {
        // super 相当于属性的继承
        // 替换了 People.call(this, name)
        super(name)
        this.gender = '男'
    }
    fight() { }
}
```


### 一、class

> **class 声明**创建一个基于原型继承的、具有给定名称的新类，其中构造函数（ `constructor` ）是可选的。



#### 1. class 声明

使用 ES6 class 定义一个计数器：

```js
class Counter extends HTMLElement {
  constructor() {
    super();
    this.onclick = this.clicked.bind(this);
    this.x = 0;
  }
  
  clicked() {
    this.x++;
    window.requestAnimationFrame(this.render.bind(this));
  }
  
  connectedCallback() { this.render(); }

  render() {
    this.textContent = this.x.toString();
  }
}
window.customElements.define('num-counter', Counter);
```



#### 2. class 字段声明

使用 ESnext 字段声明建议，上述示例可以写成：

```js
class Counter extends HTMLElement {
  x = 0;

  constructor() {
    super();
    this.onclick = this.clicked.bind(this);
  }

  clicked() {
    this.x++;
    window.requestAnimationFrame(this.render.bind(this));
  }

  connectedCallback() { this.render(); }

  render() {
    this.textContent = this.x.toString();
  }
}
window.customElements.define('num-counter', Counter);
```

在上面的示例中，你可以看到使用语法 x = 0 声明的字段。还可以将没有初始化程序的字段声明为 x。通过预先声明字段，类定义变得更加自我记录;实例经历较少的状态转换，因为声明的字段始终存在。

### 3. super(props)

在 JavaScript 中，`super` 指的是父类（即超类）的构造函数。

值得注意的是，在调用父类的构造函数之前，你是不能在 constructor 中使用 `this` 关键字的。JavaScript 不允许这个行为。

```js
class Checkbox extends React.Component {
  constructor(props) {
    // 🔴  还不能使用 `this`
    super(props);
    // ✅  现在可以了
    this.state = { isOn: true };
  }
  // ...
}
```

JavaScript 有足够合理的动机来强制你在接触 `this` 之前执行父类构造函数。考虑考虑一些类层次结构的东西：

```js
class Person {
  constructor(name) {
    this.name = name;
  }
}

class PolitePerson extends Person {
  constructor(name) {
    this.greetColleagues(); // 🔴  这是禁止的，往后见原因
    super(name);
  }
  greetColleagues() {
    alert('Good morning folks!');
  }
}
```

 `this.greetColleagues` 在 `super()` 给 `this.name` 赋值前就已经执行。`this.name` 此时甚至尚未定义。可以看到，这样的代码难以往下推敲。

为了避免落入这个陷阱，**JavaScript 强制你在使用 this 之前先行调用 super。**让父类来完成这件事情！：

```js
constructor(props) {
    super(props);
    // ✅ 能使用 `this` 了
    this.state = { isOn: true };
  }
```



### 4. 核心代码

`extends` 继承的核心代码如下，其实现和上述的寄生组合式继承方式一样

```Js
function _inherits(subType, superType) {
    // 创建对象，Object.create 创建父类原型的一个副本
    // 增强对象，弥补因重写原型而失去的默认的 constructor 属性
    // 指定对象，将新创建的对象赋值给子类的原型 subType.prototype
    subType.prototype = Object.create(superType && superType.prototype, {
        constructor: { // 重写 constructor
            value: subType,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    if (superType) {
        Object.setPrototypeOf 
            ? Object.setPrototypeOf(subType, superType) 
            : subType.__proto__ = superType;
    }
}
```



## 继承的使用场景

- 不要仅仅为了使用而使用它们，这只是在浪费时间而已。
- 当需要创建 **一系列拥有相似特性的对象** 时，那么创建一个包含所有共有功能的通用对象，然后在更特殊的对象类型中继承这些特性。
- 应避免多继承，造成混乱。

**注:** 考虑到JavaScript的工作方式，由于原型链等特性的存在，在不同对象之间功能的共享通常被叫做 **委托** - 特殊的对象将功能委托给通用的对象类型完成。这也许比将其称之为继承更为贴切，因为“被继承”了的功能并没有被拷贝到正在“进行继承”的对象中，相反它仍存在于通用的对象中。



#### 扩展二：IIFE：避免污染全局命名空间

```js
function foo() {...}     // 这是定义，Declaration；定义只是让解释器知道其存在，但是不会运行。

foo();                   // 这是语句，Statement；解释器遇到语句是会运行它的

// 错误写法
function foo(...){}();   // 因为 function foo(...){} 这个部分只是一个声明，对于解释器来说，就好像你写了一个字符串 "function foo(...){}"，它需要使用解析函数，比如 eval() 来执行它才可以。所以把 () 直接放在声明后面是不会执行，这是错误的语法。我们需要把 声明 变成 表达式（Expression）

// 正确写法
// 写法 1
var foo = function () {...};    // 这就不是定义，而是表达式了。
foo();
// 写法 2
(function foo() {...})    // 这里是故意换行，以便于理解，和 写法 3 是一致的
();
// 写法 3
(function foo(){...}());
// 写法 4
!function foo() {...}();
// 写法 5
+function foo() {...}();
// 写法 6
void function () {
    // 这里是真正需要的代码
}();                 
```