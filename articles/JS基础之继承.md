相对于ES5继承，ES6继承就简单的多，关于继承机制的设计思想，请参见http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html

1. 在ES5中。我们可以用以下方法解决继承的问题

   ```js
   function Super() {}
   Super.prototype.getNumer = function() {
       return 1
   }
   
   function Sub()
   let sub = new Sub()
   // Object.create方法接受传入一个作为新创建对象的原型的对象，创建一个拥有指定原型和若干个指定属性的对象
   Sub.prototype = Object.create(Super.prototype, {
       constructor: {
           value: Sub,
           enumerable: false,
           writable: true.
           configurable: true
       }
   })
   ```

   以上继承实现思路就是将子类的原型设置为父类的原型

2. 但在ES6中，我们可以简单的通过`class`语法（语法糖）解决这个问题：

   ```js
   class MyDate extends Date {
       test() {
           return this.getTime()
       }
   }
   let myDate = new MyDate()
   myDate.test()
   ```

   但是 ES6 不是所有浏览器都兼容，所以我们使用 Babel 来编译这段代码。结果：

   ![继承](/Users/a123/Desktop/Study/JS/img/%E7%BB%A7%E6%89%BF.png)

   这是因为 JS 底层有限制，如果不是由 `Date` 构建出来的实例的话，是不能调用 `Date` 里的函数的。所以这也侧面的说明来：**ES6 中的`class` 继承与ES5 中的一般继承写法是不同的。**

   既然，底层限制了实例必须由 `Date` 构建出来，那么我们可以改变下思路继承：

   ```js
   function MyDate() {}
   
   MyDate.prototype.test = function() {
       return this.getTime()
   }
   let d = new Date()
   Object.setPrototypeOf(d, MyDate.prototype)
   Object.setPrototypeOf(MyDate.prototype, Date.prototype)
   ```

   以上继承实现的思路：先创建父类实例 => 改变实例原有的 `__proto__` 转而连接到子类的 `prototype` => 子类的 `prototype` 的 `__proto__` 改为父类的 `prototype`。

   通过以上方法实现的继承就可以完美解决 JS 底层的这个限制。



### ES5 继承

定义一个父类

```js
function People(name) {
    // 属性
    this.name = name
    // 实例方法
    this.eat = function() {
        console.log('eat')
    }
}
// 原型方法
People.prototype.run = function() {}
```

1. 原型链继承：将父类的实例作为子类的原型

   ```js
   function Man(name) {}
   Man.prototype = new People('Man')
   ```

   特点：既继承了父类的模板，又继承了父类的原型对象。优点是**继承了父类的模板，又继承了父类的原型对象**

   缺点：

   - 可以在子类构造函数中，为子类实例增加实例属性。如果要新增原型属性和方法，则必须放在new People()这样的语句之后执行。
   - 无法实现多继承
   - 来自原型对象的所有属性被所有实例共享
   - 创建子类实例时，无法向父类构造函数传参

2. 构造继承：使用父类的构造函数来增强子类实例，等于是复制父类的实例属性给子类（没用到原型）

   ```js
   function Man(name) {
       // 调用People构造函数
       // 实现 this.name = name
       People.call(this, name)
       this.gender = '男'
   }
   ```

   优点是**解决了1中子类实例共享父类引用对象的问题，实现多继承，创建子类实例时，可以向父类传递参数**

   缺点：

   - 实例并不是父类的实例，只是子类的实例
   - 只能继承父类的实例属性和方法，不能继承原型属性/方法
   - 无法实现函数复用，每个子类都有父类实例函数的副本，影响性能

3. 组合继承：通过调用父类构造，继承父类的属性并保留传参的优点，然后通过将父类实例作为子类原型，实现函数复用

   ```js
   function Man(name){
     People.call(this, name);
     this.name = name || 'Tom';
   }
   Man.prototype = new People();
   
   // 组合继承也需要修复构造函数指向的。
   Man.prototype.constructor = Man;
   ```

   优点是**弥补了方式2的缺陷，可以继承实例属性/方法，也可以继承原型属性/方法，不存在引用属性共享问题，可传参，可复用**

   缺点：

   - 调用了两次父类构造函数，生成了两份实例（子类实例将子类原型上的那份屏蔽了）

4. 寄生组合继承：通过寄生方式，砍掉父类的实例属性，这样，在调用两次父类的构造的时候，就不会初始化两次实例方法/属性，避免的组合继承的缺点

   ```js
   function Man(name){
     People.call(this, name);
     this.name = name || 'Tom';
   }
   (function(){
     // 创建一个没有实例方法的类
     var Super = function(){};
     Super.prototype = People.prototype;
     //将实例作为子类的原型
     Man.prototype = new Super();
   })();
   
   Man.prototype.constructor = Man; // 需要修复下构造函数
   ```

### ES6 继承

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