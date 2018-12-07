
> 小程序项目随笔, 记录一下一些想法, 遇到的问题和遗忘的知识
---  
- [1. 开发工具](##1-开发工具)  
- [2. 小程序的写法](##2-小程序的写法)  
- [3. 电池栏的适配](##3-电池栏的适配)  
- [4. 小程序里的data](##4-小程序里的data)  
- [5. 关于if和hidden的条件渲染](##5-关于if和hidden的条件渲染)  
- [6. 关于for循环渲染](##6-关于for循环渲染)  
- [7. 微信小程序里往元素上绑定方法, 以及绑定data](##7-微信小程序里往元素上绑定方法-以及绑定data)  
- [8. 关于微信里的js语法](##8-关于微信里的js语法)  
  
## 1. 开发工具  

本人使用的是vsCode编写小程序的, 安装Vetur和minapp插件以支持微信小程序. (vsCode的less编译真好用).
## 2. 小程序的写法  

首先嘛, 小程序和vue.js非常相似, 但也不尽相同, 本人有使用vue的经验, 并没有十分仔细的完整阅读微信小程序的文档, 以至于有很对问题明明文档里有直接的解决办法, 我还在抓破脑袋的瞎想. 幸亏我老哥也是做小程序的, 每次问他都是拿文档贴我脸上, 嘿嘿嘿.  
## 3. 电池栏的适配  

当ui不想使用微信自带的导航栏时, 自己写就得做电池栏的适配了.   
这个`statusBarHeight`就是电池栏的高度了, 注意单位是px, 往里写rpx的时候要*2. 微信小程序的rpx单位是以苹果6 dpr2为基准算的, 会根据机型自动调整, 暂时未发现有什么大问题.  
``` javascript  
wx.getSystemInfo({
  success: function (res) {  
    if (res.statusBarHeight == undefined) {  
      wx.setStorageSync('BGGetSystem',res.statusBarHeight);
    } else {  
      wx.setStorageSync('BGGetSystem',res.statusBarHeight);
    }
  }
})  
```
## 4. 小程序里的data  

微信小程序里读取是`this.data.$name`, 存储是`this.setData({$name: '$data'})`.  
vue里读取和存储都是`this.$name`,  
react里读取是`this.state.$name`, 存储是`this.setState({$name: '$data'})`.
## 5. 关于if和hidden的条件渲染  

微信小程序里的`wx:if`和`hidden`是同**vue**里的`v-if`和`v-show`类似.  
`wx:if`与`v-if`完全相同, 是控制是否加载这个元素的.  
而`hidden`和`v-show`类似, 只是简单的控制元素`display`值, 所以当你对此元素有`display`上设置的话, `hidden`就无效了.  
另外需要注意的是, `hidden`值为真是**隐藏**, `v-show`值为真是**显示**.  
**react**里的条件渲染是在`render`时, 直接通过**js**控制的.
## 6. 关于for循环渲染  

微信小程序的`wx:for`和`v-for`类似.  
值得注意的是, 微信小程序默认的循环value是item, 下标是index, 可以直接使用;  
而vue里需要你自己指定value和下标.  
> 微信小程序
```  html  
<view wx:for="{{array}}" wx:for-index="idx" wx:for-item="itemName">
  {{idx}}: {{itemName.message}}
</view>
```
> vue  
```  html  
<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
```
> react
```js
<ul>
  {
    this.state.list.map(function (item, i) {
      return (
        <li key={i}>
          <div>{item.value}</div>
        </li>
    )})
  }
</ul>
```
## 7. 微信小程序里往元素上绑定方法, 以及绑定data  

在这一点上微信小程序和vue的差别就已经不小了.  
在代码中可以看见, 小程序绑定方法使用的是`bind:tap(bindtap)`,  
而vue里是`v-on:click="doSomething"(@click="doSomething")`.  
在微信小程序里, 你需要`data-$name`, 然后在js里通过`e`来获取. 这时有一点儿要注意, 获取的值要在`currentTarget`里找, 不要使用其他的键名, 虽然都可一看到有一样的键值对, 但是会出现各种问题.  
vue就是普通的函数参数啦, 没什么好说的.  
至于react, `html+js`, 妥妥的.

> 微信小程序
```html
<view class="thinkCard" wx:for="{{investmentList}}" wx:key="{{item.investmentId}}" data-investmentId="{{item.investmentId}}" bind:tap="goThinkDetail">
  ***
</view>
```
```js  
goThinkDetail: function (e) {
  let investmentId = e.currentTarget.dataset.investmentid;
},
```
> vue
```js
<ul>  
  <li v-for="i, index in data.content" @click="captionClick(index)"> 
    <img class="yuan" src="../static/yuan.png"/>  
    {{ i.caption }}  
  </li>  
</ul>
 ```
> react
```html
<button onClick={this.handleClick}>按钮</button>
 ```  
## 8. 关于微信里的js语法  

微信小程序已经可以很好的支持es6(es2015), 多多使用es6的语法吧.  
`promise`可以帮您解决好多麻烦的问题.  
但是微信暂时没有支持es7及以上. 有些麻烦的异步问题非常想用`async`和`awiat`解决怎么办?  
[微信小程序开发用 JS ES8 async/await 解决异步调用](https://pluwen.com/archives/397)  
[regenerator-runtime](https://github.com/facebook/regenerator/blob/master/packages/regenerator-runtime/runtime.js) 引入, 直接就可以使用`async`和`awiat`啦! 除了在哪里用就都要引入之外, 暂时没什么缺点.  
需要注意的是, 虽然本人使用`async/await`时没有出现过问题. 但是在使用es6 `forof`的时候出现过莫名其妙的解析失败, 将其换成`forin`之后就可以正常使用. 有此前车之鉴, 我的想法是, 尽量少用?
