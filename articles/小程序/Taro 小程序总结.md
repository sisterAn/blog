最近，工作加班写小程序，虽然之前也写了不少小程序，但大多是使用小程序原生来写，也使用过wepy框架，风格和vue差不多，这次尝试使用taro框架。今天小程序告一断落，特此总结一波。

### 安装使用Taro
1. 安装
```js
npm install -g @tarojs/cli
```
2. 使用命令创建模板项目
```js
$ taro init myApp
```
3. 微信小程序编译预览及打包
```js
$ npm run dev:weapp
$ npm run build:weapp
```
主要详见官网Taro[安装及使用](https://nervjs.github.io/taro/docs/GETTING-STARTED.html)，重要的事情说三遍，一定要认真看官方文档、官方文档、官方文档，做好基础的入门。

### 开发前注意
若使用 **微信小程序预览**模式 ，则需下载并使用微信开发者工具添加项目进行预览，此时需要注意微信开发者工具的项目设置
- 需要设置关闭 ES6 转 ES5 功能，开启可能报错
- 需要设置关闭上传代码时样式自动补全，开启可能报错
- 需要设置关闭代码压缩上传，开启可能报错


### 框架
```js
.
├── dist                   编译结果目录
├── config                 配置目录
|   ├── dev.js             开发时配置
|   ├── index.js           默认配置
|   └── prod.js            打包时配置
├── src                    源码目录
|   ├── pages              页面文件目录
|   |   ├── index          index 页面目录
|   |   |   ├── index.js   index 页面逻辑
|   |   |   └── index.css  index 页面样式
|   ├── app.css            项目总通用样式
|   └── actions            redux actions
|   └── constants         
|   └── libs             
|   └── net           
|   └── asset          
|   └── reducers       
|   └── store            
|   └── utils            
└── package.json
```
### 给组件设置 defaultProps

在微信小程序端的自定义组件中，只有在 properties 中指定的属性，才能从父组件传入并接收
```js
Component({
  properties: {
    myProperty: { // 属性名
      type: String, // 类型（必填），目前接受的类型包括：String, Number, Boolean, Object, Array, null（表示任意类型）
      value: '', // 属性初始值（可选），如果未指定则会根据类型选择一个
      observer: function (newVal, oldVal, changedPath) {
         // 属性被改变时执行的函数（可选），也可以写成在 methods 段中定义的方法名字符串, 如：'_propertyChange'
         // 通常 newVal 就是新设置的数据， oldVal 是旧数据
      }
    },
    myProperty2: String // 简化的定义方式
  }
  ...
})
```
而在 Taro 中，对于在组件代码中使用到的来自 props 的属性，会在编译时被识别并加入到编译后的 properties 中，暂时支持到了以下写法
```js
this.props.property

const { property } = this.props

const property = this.props.property
```
但是一千个人心中有一千个哈姆雷特，不同人的代码写法肯定也不尽相同，所以 Taro 的编译肯定不能覆盖到所有的写法，而同时可能会有某一属性没有使用而是直接传递给子组件的情况，这种情况是编译时无论如何也处理不到的，这时候就需要大家在编码时给组件设置 defaultProps 来解决了。
```js
export default class LGLine extends Component {
  static propTypes = {
    value: string,
  };

  static defaultProps = {
    value: '',
  };
  render () {
    const { value } = this.props
    return (
      <View className='height-1 base-width-12'>
      </View>
    )
  }

}
```
组件设置的 defaultProps 会在运行时用来弥补编译时处理不到的情况，里面所有的属性都会被设置到 properties 中初始化组件，正确设置 defaultProps 可以避免很多异常的情况的出现。

### JS 编码必须用单引号

在 Taro 中，JS 代码里必须书写单引号，特别是 JSX 中，如果出现双引号，可能会导致编译错误。

### 环境变量 process.env 的使用

不要以解构的方式来获取通过 env 配置的 process.env 环境变量，请直接以完整书写的方式 process.env.NODE_ENV 来进行使用
```js
// 错误写法，不支持
const { NODE_ENV = 'development' } = process.env
if (NODE_ENV === 'development') {
  ...
}

// 正确写法
if (process.env.NODE_ENV === 'development') {

}
```
### cover-view
`cover-view`经常使用到换行，设置文本内容换行 `white-space: normal;`
### 动态获取view的滚动位置
由于我们在滚动页面中经常会使用到`textarea`，`video`等原生组件，这些原生组件在`scroll-view`中的支持并不友好，所有有时我们有时无法使用`scroll-view`，也就无法去获取`scrollTop`，例如[Taro 拖拽排序](https://blog.csdn.net/lunahaijiao/article/details/86576845)，这时，可以这样做：
```js
onPageScroll () {
    let that = this;
    var query = Taro.createSelectorQuery()
    query.select('.content-view').boundingClientRect()
    query.selectViewport().scrollOffset()
    query.exec(function(res) {
      that.setState({
        scrollPosition: {
          scrollTop: res[1].scrollTop,
          scrollY: that.state.scrollPosition.scrollY
        },
      })
    })
  }
```
### 性能提升
小程序项目中遇到的性能问题，大多是频繁地调用 setData 造成的，这是由于每调用一次 setData，小程序内部都会将该部分数据在逻辑层（运行环境 JSCore）进行类似序列化的操作，将数据转换成字符串形式传递给视图层（运行环境 WebView），视图层通过反序列化拿到数据后再进行页面渲染，这个过程下来有一定性能开销。

所以开发过程中，我们建议尽量对 setData 进行合并，减少调用次数，例如：
```js
this.setData({ foo: 'Strawberry' })
this.setData({ foo: 'Strawberry', bar: 'Fields' })
this.setData({ baz: 'Forever' })
```
以上代码调用了 3 次 setData，造成不必要的性能开销，应对其进行合并：
```js
this.setData({
    foo: 'Strawberry',
    bar: 'Fields',
    baz: 'Forever',
})
```
而使用 Taro 之后，更新数据时调用的 setState 为异步方法，它会自动地对同一事件循环里的多次 setState 调用进行合并处理，此外还会进行数据 diff 优化，自动剔除那些未变更的数据，从而有效避免了此类性能问题。例如：
```JS
// 初始时
this.state = {
    foo: '1967',
    bar: {
        foo: 'Strawberry',
        bar: 'Fields',
        baz: 'Forever',
    }
}
// 第一次更新
this.setState({
    bar: {
        foo: 'Norwegian',
        bar: 'Fields',
        baz: 'Forever',
    }
})
// 紧接着进行第二次更新
this.setState({
    foo: '1967',
    bar: {
        foo: 'Norwegian',
        bar: 'Wood',
        baz: 'Forever',
    }
})
```
以上代码虽然经过两次 setState，但只有 bar.foo 和 bar.bar 的数据更新了，此时 Taro 内部会自动对数据进行合并、并剔除重复数据，最终执行代码为：
```js
// this.$scope 在小程序环境中为 page 实例
this.$scope.setData({
    'bar.foo': 'Norwegian',
    'bar.bar': 'Wood',
})
```
### 更多
其他的问题，请看下面：
[Taro event handler 传递参数有问题](https://blog.csdn.net/lunahaijiao/article/details/86574479)
[Taro 阻止事件冒泡](https://blog.csdn.net/lunahaijiao/article/details/86574610)
[Taro 对接腾讯云对象存储服务COS](https://blog.csdn.net/lunahaijiao/article/details/86575746)

本文参考了[Taro 在京东购物小程序上的实践](https://aotu.io/notes/2018/09/11/taro-in-jd/index.html)及[最佳实践](https://nervjs.github.io/taro/docs/best-practice.html)