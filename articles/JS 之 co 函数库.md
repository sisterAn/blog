co 函数库(https://github.com/tj/co)是 TJ 大神基于ES6 generator 的异步解决方案。要理解 co ，你必须首先理解 ES6 generator，可以看我之前的文章，这里不在赘述。

co 最大的好处在于通过它可以把异步的流程以同步的方式书写出来，并且可以使用 try/catch。

废话少说，上实例：

```js
var co = require('co')
var fs = require('fs')
// wrap the function to thunk
function readFile(filename) {
    return new Promise(function(resolve, reject) {
        fs.readFile(filename, function(err, date) {
            if (err) reject(err)
            resolve(data)
        })
    })
}
// generator 函数
function *gen() {
    var file1 = yield readFile('./file/1.txt') // 1.txt内容为：content in 1.txt
    var file2 = yield readFile('./file/2.txt') // 2.txt内容为：content in 2.txt
    console.log(file1)
    console.log(file2)
    return 'done'
}
// co
co(gen).then(function(err, result) {
    console.log(result)
})
// content in 1.txt
// content in 2.txt
// done
```

在上例中，co 函数库可以让你不用编写 generator 函数的执行器，generator 函数只要放在 co 函数里，就会自动执行。

再看一个例子：

```js
co(function *(){
  try {
    var res = yield get('http://baidu.com');
    console.log(res);
  } catch(e) {
    console.log(e.code) 
 }
})
```

**co 函数库其实就是将两种自动执行器（Thunk 函数和 Promise 对象），包装成一个库。但用 co 的一个代价是 Generator 函数的 yield 命令后面必须返回一个 Thunk 或者一个 Promise。**

源码实现：

```js
function co(gen) { // co 接受一个 generator 函数
    var ctx = this
    var args = slice.call(arguments, 1)
    
    return new Promise(function(resolve, reject) { // co 返回一个 Promise 对象
        if(typeof gen === 'function') gen = gen.apply(ctx, args) // gen 为 generator 函数，执行该函数
        if(!gen || typeof gen.next !== 'function') return resolve(gen) // 不是则返回并更新 Promise状态为 resolve
        
        onFulfilled() // 将generator 函数的 next 方法包装成 onFulfilled，主要是为了能够捕获抛出的异常
        
        /**
     	 * @param {Mixed} res
     	 * @return {Promise}
     	 * @api private
     	*/
        function onFulfilled(res) {
            var ret;
            try {
                ret = gen.next(res)
            } catch (err) {
                return reject(err)
            }
            next(ret)
        }
        
        /**
     	 * @param {Error} err
     	 * @return {Promise}
     	 * @api private
     	*/
        function onRejected(err) {
            var ret
            try {
                ret = gen.throw(err)
            } catch (err) {
                return reject(err)
            }
            next(ret)
        }
        
        /**
     	 * Get the next value in the generator,
    	 * return a promise.
    	 *
    	 * @param {Object} ret
    	 * @return {Promise}
    	 * @api private
     	*/
        function next(ret) {
            if(ret.done) return resolve(ret.value)
            var value = toPromise.call(ctx, ret.value) // if (isGeneratorFunction(obj) || isGenerator(obj)) return co.call(this, obj);
            if(value && isPromise(value)) return value.then(onFulfilled, onRejected)
            return onRejected(new TypeError('You may only yield a function, promise, generator, but the following object was passed: ' + String(ret.value) + '"'))
        }
    })
}
```

注意：**`onFulfilled`这个函数只在两种情况下被调用，一种是调用co的时候执行，还有一种是当前promise中的所有逻辑都执行完毕后执行**