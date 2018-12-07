
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
另外有一点儿要注意的是, 当你想写一个公共方法, 需要传入setData的键名时, 直接将参数放在键的位置是不行的, 你需要加一个中括号. 记得把调用页面的`this`一起穿.
```js  
that.setData({
  [$name]: $value
})
```  

## 5. 关于if和hidden的条件渲染  

微信小程序里的`wx:if`和`hidden`是同**vue**里的`v-if`和`v-show`类似.  
`wx:if`与`v-if`完全相同, 是控制是否加载这个元素的.  
而`hidden`和`v-show`类似, 只是简单的控制元素`display`值, 所以当你对此元素有`display`上设置的话, `hidden`就无效了.  
另外需要注意的是, `hidden`值为真是**隐藏**, `v-show`值为真是**显示**.  
**react**里的条件渲染是在`render`时, 直接通过**js**控制的.  
微信有一个神奇的标签`<block></block>`, 他是一个空标签, 主要用在`wx:if, hidden, wx:for`上. 在渲染的时候就不需要多一个`<view></view>`标签了.

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

## 7. 元素上绑定方法, 以及绑定data  

在这一点上微信小程序和vue的差别就已经不小了.  
在代码中可以看见, 小程序绑定方法使用的是`bind:tap(bindtap)`,  
而vue里是`v-on:click="doSomething"(@click="doSomething")`.  
在微信小程序里, 你需要`data-$name`, 然后在js里通过`e`来获取. 这时有一点儿要注意, 获取的值要在`currentTarget`里找, 不要使用其他的键名, 虽然都可一看到有一样的键值对, 但是会出现各种问题.  
>**注意啦**`currentTarget.dataset.$name`里, 他的`$name`并不是你写在元素里的名字, 会全部变成小写字母.  

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
> [微信小程序开发用 JS ES8 async/await 解决异步调用](https://pluwen.com/archives/397)  
> 
[regenerator-runtime](https://github.com/facebook/regenerator/blob/master/packages/regenerator-runtime/runtime.js) 引入, 直接就可以使用`async`和`awiat`啦! 除了在哪里用就都要引入之外, 暂时没什么缺点.  
需要注意的是, 虽然本人使用`async/await`时没有出现过问题. 但是在使用es6 `forof`的时候出现过莫名其妙的解析失败, 将其换成`forin`之后就可以正常使用. 有此前车之鉴, 我的想法是, 尽量少用?

## 9. wx.request  

我们的小程序其中有一个获取11位秘钥的接口, 直接获取发现json格式自动截取了10位. 这种情况下, 给request设置一下接受格式为text, 就可以解决了.  
```js  
wx.request({
  url: "https://*",
  dataType: 'text',
  responseType: 'text',
  header: { 'content-type': 'text/html; charset=utf-8' },
  success: function (res) {
    console.log('factor', res);
  }
});
```  

## 10. 倒计时
看这里!
>[微信小程序---完整的验证码获取倒计时效果 ---根据手机号是否符合要求进行判断](https://blog.csdn.net/Candy_mi/article/details/80225359)  
倒计时的原理就是通过一个计时器改变data状态, 然后渲染到页面里. 根据自己的具体需求, 加些验证, 改变提示文字即可.  

## 11. 长按和单次点击功能不一样  
如果是一个长按复制和单机跳转怎么办呢? 你会发现, 单纯的使用`text`组件的长按复制和`bind:tap`, 长按的时候会触发tap事件.  
这时候就有两个属性`bindtouchstart`和`bindtouchend`出现了, 这是触碰开始和触碰结束, 判断开始到结束的时间, 来触发不同的方法即可.  
```html  
<view bindtouchstart="mytouchstart" bindtouchend="mytouchend" bind:tap="tapEvent">
</view>
```  
```js  
//按下事件开始  
mytouchstart: function (e) {
  let that = this;
  that.setData({
    touch_start: e.timeStamp
  })
},
//按下事件结束  
mytouchend: function (e) {
  let that = this;
  that.setData({
    touch_end: e.timeStamp
  })
},
//判断跳转还是复制文本
tapEvent: function (e) {
  let that = this;
  //触摸时间距离页面打开的毫秒数  
  var touchTime = that.data.touch_end - that.data.touch_start;
  let text = e.currentTarget.dataset.text
  //如果按下时间大于350为长按  
  if (touchTime > 350) {
    this.copy(text)
  } else {
    this.goTaskList()
  }
},
//长按复制
copy: function (text) {
  var that = this;
  wx.setClipboardData({
    data: text,
    success: function (res) {
      wx.showToast({
        title: '复制成功',
      });
    }
  });
},
```  

## 12. input和button样式的大坑

微信小程序里, input和button有一大堆的默认样式, 相当坑. 当你需要高度的自定义一个按钮, 就会让你头疼. 其中的伪类样式有一个border边框.
```css  
button {
  margin-left: 0;
  margin-right: 0;
  padding-left: 0;
  padding-right: 0;
  box-sizing: content-box;
  background-color: #FFFFFF;
}
button::after {
  border: none;
}
input {
  min-height: 0;
  outline: none;
  border: none;
  list-style: none;
}
```  

## 13. 文字超出省略  
微信小程序的文字超出省略就简单了, 他是`webkit`内核的, 直接使用常规的就OK.
```css  
// 一行文字省略
.ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
// 两行文字省略
.twoEllipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}
// 三行文字省略
.threeEllipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}
```  
还有一种使用纯css的兼容写法.在逛git的时候, 看见[happylindz](https://github.com/happylindz)大佬的[纯 CSS 实现多行文字截断](https://github.com/happylindz/blog/issues/12), 真是太佩服了. 以前从未想到过`float`可以这样用.

## 14. template模板

WXML提供模板(template), 可以在模板中定义代码片段, 然后在不同的地方调用.  
在一个wxml模板里, 给template起一个`name`, 这个`name`将会在调用模板时, 声明这个模板是谁(名字). 调用时使用一个`<import src='..'></import>`标签引入. 每个模板有一个单独的作用域, 在模板中使用的数据, 要通过`data="{{$data}}`传入, 否则无法调用.  
```html  
<template name="msgItem">
  <view>
    <text> {{index}}: {{msg}} </text>
    <text> Time: {{time}} </text>
  </view>
</template>
```  
```html  
<import src="../../templates.wxml"></import>
```  
```html  
<template is="msgItem" data="{{...item}}"/>
```  
```js  
Page({
  data: {
    item: {
      index: 0,
      msg: 'this is a template',
      time: '2016-09-15'
    }
  }
})
```  
WXSS可以使用@import语句可以导入外联样式表，@import后跟需要导入的外联样式表的相对路径，用;表示语句结束。
而js, 就和平常一样引入就行了. 调用时记得传入this.  

## 15. component组件  

component组件的写法基本和page页面一样, 简单用法参考官方文档即可.  
我就用到的地方说几句.  
- 子组件的wxss不会自动引入app.wxss的公共样式, 需要手动引入.  
- component组件里, 是有他自己的生命周期的, `ready(){}` 比较常用, 他是在**组件生命周期函数，在组件布局完成后执行**. 更多的生命周期参考文档.  
- 父子组件通信可就麻烦了, 像vue一样, 子组件不可以直接向父组件传值. 需要在父组件里向子组件绑定一个方法, 子组件在执行自己的方法之后, 调用这个方法, 通过`this.triggerEvent('add-collect-project', myEventDetail)`给父组件传值.  
- 绑定的方法和vue一样, 不支持驼峰写法, 要用 `-` 相连.
> 父组件wxml 
```html
<!-- 列表 -->
<my-projectList bind:add-collect="addCollectProject" list="{{hotList[hotProjectGroup]}}"></my-projectList>
```  
> 父组件js
```js  
/**收藏
* @param  {} e
*/
addCollectProject: function (e) {
  console.log(e.detail.index);
},
```  
> 字组件wxml 
```html
<!-- 列表 -->
<view data-index="{{index}}" bind:tap="collectProject">
  <image class="img" src="" />
</view>
```  
> 子组件js
```js  
/**收藏
  * @param  {} e
  */
collectProject: function (e) {
  console.log(e);
  let index = e.currentTarget.dataset.index;
  var myEventDetail = { index }; // detail对象，提供给事件监听函数
  this.triggerEvent('add-collect', myEventDetail)
},  
```