如果，你在开发过程中，发现event handler 传递参数有问题，例如：代码里使用onClick={this.handleClick.bind(this, 1)}的方式绑定事件处理程序，并且传递值，其定义是

```js
// 方法
handleClick (params , e) {
    console.log('params:', params)
    console.log('e:', e)
}
```
```js
// render
<View className='flexDirection-c'>
     <View className='base-width-12 lh-22 tx-c fs-14' onClick={this.handleClick.bind(this, -1)}>全部商品</View>
     <View className='base-width-12 lh-22 tx-c fs-14' onClick={this.handleClick.bind(this, 0)}>在售商品</View>
     <View className='lh-22 tx-c fs-14' onClick={this.handleClick.bind(this, 1)}>下架商品</View>
</View>
```
打印结果你如果看到：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190121144820110.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1bmFoYWlqaWFv,size_16,color_FFFFFF,t_70)
而我们所期望的结果是：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190121145226756.png)
这是因为你项目中的`taro-cli`与依赖的版本不同，升级依赖即可，
项目中必须确保 CLI 和项目依赖是相同版本。