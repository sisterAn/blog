### 实现步骤
#### 1. 拖拽排序实现
这一块不是难点，主要是[movable-view，movable-area](https://developers.weixin.qq.com/miniprogram/dev/component/movable-view.html#movable-area)及一些页面逻辑的计算（当前拖拽的控件，拖拽到什么地步进行排序）。

**需要注意：排序时setState/setData的延时，可能会引起多次排序错乱**

#### 2.  性能优化
在简单的拖拽排序基本实现后，我们需要关注它的性能及流畅度。
由于我们的`contents`(拖拽排序数组)中设计`textarea`、`video`等原生控件。

> 1. 页面滑动的时候，可能出现`textarea`、`video`凌驾于`navigator`或其他高层级组件上

这是因为`textarea`、`video`为原生组件，脱离在 WebView 渲染流程外，原生组件的层级是最高的，所以页面中的其他组件无论设置 z-index 为多少，都无法盖在原生组件上，
为了解决原生组件层级最高的限制，小程序专门提供了  [cover-view 、 cover-image](https://developers.weixin.qq.com/miniprogram/dev/component/cover-view.html)组件，可以覆盖在部分原生组件上面。我们需要把在`textarea`、`video`层次上的组件写到`cover-view`中。

> 2. 滑动页面的时候，当滑动落在`textarea`上时，可能出现页面卡顿的现象

这是因为我们把`contents`(拖拽排序数组)写在了`scroll-view` 中，而滑动手势落在`textarea`上时，`textarea`的层级在`scroll-view`之上，操作会被`textarea`拦截,就不会作用到`scroll-view`滚动上，这时，两者只能选其一，`textarea`是我们必须要有的功能，如果用伪多行输入又会造成更多的性能问题，如果你有更好的方案，可以试一下，这里我们去除`scroll-view`
##### 出现的问题： 获取页面滚动问题
由于我们通过页面的`clientY`以及`scrollY`来得到当前拖拽控件及拖拽路径，而不使用`scroll-view`后无法`scroll-view`滚动的距离，这时，需要使用页面的`onPageScroll`方法
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
> 3. 文本控件拖拽时，textarea没有跟随移动

由于`textarea`不是`position: fixed`的，所以`fixed={true}`无效，我这里的解决方法是把拖拽的时`movable-view`的文本框设为`text`组件。

#### 3. 后续
后续如果继续优化，我会不断更新，感谢你的阅读

