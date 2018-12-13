
>**小程序项目随笔, 记录一下一些想法, 遇到的问题和遗忘的知识**
---  
> [Markdown基本语法](https://www.jianshu.com/p/191d1e21f7ed)  
> [实用帖 | 如何为 Markdown 文件自动生成目录？](https://www.jianshu.com/p/4721ddd27027)
---

- [1. 开发工具](#1-开发工具)
- [2. 小程序的写法](#2-小程序的写法)
- [3. 电池栏的适配](#3-电池栏的适配)
- [4. 小程序里的data](#4-小程序里的data)
- [5. 关于if和hidden的条件渲染](#5-关于if和hidden的条件渲染)
- [6. 关于for循环渲染](#6-关于for循环渲染)
- [7. 元素上绑定方法, 以及绑定data](#7-元素上绑定方法-以及绑定data)
- [8. 关于微信里的js语法](#8-关于微信里的js语法)
- [9. wx.request](#9-wxrequest)
- [10. 倒计时](#10-倒计时)
- [11. 事件](#11-事件)
  - [11.1. 长按和单次点击功能不一样](#111-长按和单次点击功能不一样)
- [12. input和button样式的大坑](#12-input和button样式的大坑)
- [13. template模板与component组件](#13-template模板与component组件)
  - [13.1. template模板](#131-template模板)
  - [13.2. component组件](#132-component组件)
- [14. input与textarea](#14-input与textarea)
  - [14.1. input](#141-input)
  - [14.2. textarea](#142-textarea)
  - [14.3. css文字换行](#143-css文字换行)
  - [14.4. 文字超出省略](#144-文字超出省略)
  - [14.5. 文字两端对齐](#145-文字两端对齐)
  - [14.6. inline-block元素设置overflow:hidden属性导致相邻行内元素向下偏移](#146-inline-block元素设置overflowhidden属性导致相邻行内元素向下偏移)
- [15. 又爱又恨的scroll-view](#15-又爱又恨的scroll-view)
  - [15.1. 回到顶部和锚点跳转](#151-回到顶部和锚点跳转)
  - [15.2. 竖向滚动问题](#152-竖向滚动问题)
  - [15.3. 获取滚动高度问题](#153-获取滚动高度问题)
  - [15.4. 上拉加载](#154-上拉加载)
  - [15.5. 在有遮罩层时, 可以滚动](#155-在有遮罩层时-可以滚动)
- [16. 路由](#16-路由)
- [17. z-index(层叠上下文)](#17-z-index层叠上下文)
  
# 1. 开发工具  

本人使用的是vsCode编写小程序的, 安装Vetur和minapp插件以支持微信小程序. (vsCode的less编译真好用).  

# 2. 小程序的写法  

首先嘛, 小程序和vue.js非常相似, 但也不尽相同, 本人有使用vue的经验, 并没有十分仔细的完整阅读微信小程序的文档, 以至于有很对问题明明文档里有直接的解决办法, 我还在抓破脑袋的瞎想. 幸亏我老哥也是做小程序的, 每次问他都是拿文档贴我脸上, 嘿嘿嘿.  

# 3. 电池栏的适配  

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

# 4. 小程序里的data  

微信小程序里读取是`this.data.$name`, 存储是`this.setData({$name: '$data'})`.  
vue里读取和存储都是`this.$name`,  
react里读取是`this.state.$name`, 存储是`this.setState({$name: '$data'})`.  
另外有一点儿要注意的是, 当你想写一个公共方法, 需要传入setData的键名时, 直接将参数放在键的位置是不行的, 你需要加一个中括号. 记得把调用页面的`this`一起穿.
```js  
this.setData({
  [$name]: $value
})
```    
如果你有一个
```js
data: {
  list: {
    list1: [1],
    list2: [2],
    list3: [3],
  }
},
```  
想只给list1赋值怎么办？加个引号即可  
```js
this.setData({
  'list.list1': [1, 2, 3]
})
```  
```js
data: {
  _list: [1, 2, 3]
}
this.setData({
  '_list[0]': [1, 2, 3]
})
```  

>[小程序赋值代码片段](https://developers.weixin.qq.com/s/MBFvIBmD7W4H)

# 5. 关于if和hidden的条件渲染  

微信小程序里的`wx:if`和`hidden`是同**vue**里的`v-if`和`v-show`类似.  
`wx:if`与`v-if`完全相同, 是控制是否加载这个元素的.  
而`hidden`和`v-show`类似, 只是简单的控制元素`display`值, 所以当你对此元素有`display`上设置的话, `hidden`就无效了.  
另外需要注意的是, `hidden`值为真是**隐藏**, `v-show`值为真是**显示**.  
**react**里的条件渲染是在`render`时, 直接通过**js**控制的.  
微信有一个神奇的标签`<block></block>`, 他是一个空标签, 主要用在`wx:if, hidden, wx:for`上. 在渲染的时候就不需要多一个`<view></view>`标签了.

# 6. 关于for循环渲染  

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

# 7. 元素上绑定方法, 以及绑定data  

在这一点上微信小程序和vue的差别就已经不小了.  
在代码中可以看见, 小程序绑定方法使用的是`bind:tap(bindtap)`,  
而vue里是`v-on:click="doSomething"(@click="doSomething")`.  
在微信小程序里, 你需要`data-$name`, 然后在js里通过`e`来获取. 这时有一点儿要注意, 获取的值要在`currentTarget`里找, 不要使用其他的键名, 虽然都可一看到有一样的键值对, 但是会出现各种问题.  
- **注意啦**`currentTarget.dataset.$name`里, 他的`$name`并不是你写在元素里的名字, 会全部变成小写字母.  

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

# 8. 关于微信里的js语法  

微信小程序已经可以很好的支持es6(es2015), 多多使用es6的语法吧.  
`promise`可以帮您解决好多麻烦的问题.  
但是微信暂时没有支持es7及以上. 有些麻烦的异步问题非常想用`async`和`awiat`解决怎么办?  
> [微信小程序开发用 JS ES8 async/await 解决异步调用](https://pluwen.com/archives/397)  
> 
[regenerator-runtime](https://github.com/facebook/regenerator/blob/master/packages/regenerator-runtime/runtime.js) 引入, 直接就可以使用`async`和`awiat`啦! 除了在哪里用就都要引入之外, 暂时没什么缺点.  
需要注意的是, 虽然本人使用`async/await`时没有出现过问题. 但是在使用es6 `forof`的时候出现过莫名其妙的解析失败, 将其换成`forin`之后就可以正常使用. 有此前车之鉴, 我的想法是, 尽量少用?

# 9. wx.request  

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

# 10. 倒计时  

看这里!
>[微信小程序---完整的验证码获取倒计时效果 ---根据手机号是否符合要求进行判断](https://blog.csdn.net/Candy_mi/article/details/80225359)  

倒计时的原理就是通过一个计时器改变data状态, 然后渲染到页面里. 根据自己的具体需求, 加些验证, 改变提示文字即可.  

# 11. 事件  

> 事件分类  

| 类型               | 触发条件                                                                               | 最低版本 |
| ------------------ | -------------------------------------------------------------------------------------- | -------- |
| touchstart         | 手指触摸动作开始                                                                       |
| touchmove          | 手指触摸后移动                                                                         |
| touchcancel        | 手指触摸动作被打断，如来电提醒，弹窗                                                   |
| touchend           | 手指触摸动作结束                                                                       |
| tap                | 手指触摸后马上离开                                                                     |
| longpress          | 手指触摸后，超过350ms再离开，如果指定了事件回调函数并触发了这个事件，tap事件将不被触发 | 1.5.0    |
| longtap            | 手指触摸后，超过350ms再离开（推荐使用longpress事件代替）                               |
| transitionend      | 会在 WXSS transition 或 wx.createAnimation 动画结束后触发                              |
| animationstart     | 会在一个 WXSS animation 动画开始时触发                                                 |
| animationiteration | 会在一个 WXSS animation 一次迭代结束时触发                                             |
| animationend       | 会在一个 WXSS animation 动画完成时触发                                                 |
| touchforcechange   | 在支持 3D Touch 的 iPhone 设备，重按时会触发                                           | 1.9.90   |

## 11.1. 长按和单次点击功能不一样  

如果是一个长按复制和单机跳转怎么办呢? 你会发现, 单纯的使用`text`组件的长按复制和`bind:tap`, 长按的时候会触发tap事件.  
这时候就有两个属性`bindtouchstart`和`bindtouchend`出现了, 这是触碰开始和触碰结束, 判断开始到结束的时间, 来触发不同的方法即可.  

```html  
<view data-text="{text}" bindtouchstart="mytouchstart" bindtouchend="mytouchend" bind:tap="tapEvent">
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

# 12. input和button样式的大坑

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

# 13. template模板与component组件  

>[微信小程序template模板与component组件的区别和使用](https://www.cnblogs.com/xyyt/p/9559326.html)

## 13.1. template模板  

WXML提供模板(template), 可以在模板中定义代码片段, 然后在不同的地方调用.  
在一个wxml模板里, 给template起一个`name`, 这个`name`将会在调用模板时, 声明这个模板是谁(名字). 调用时使用一个`<import src='..'></import>`标签引入. 每个模板有一个单独的作用域, 在模板中使用的数据, 要通过`data="{{$data}}`传入, 否则无法调用.  
> 子模板
```html  
<template name="msgItem">
  <view>
    <text> {{index}}: {{msg}} </text>
    <text> Time: {{time}} </text>
  </view>
</template>
```  
> 父page引入
```html  
<import src="../../templates.wxml"></import>
```  
> 父page使用
```html  
<template is="msgItem" data="{{...item}}"/>
```  
> 向子模板传入item
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

## 13.2. component组件  

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

# 14. input与textarea

微信小程序里的input和textarea是原生组件, 小程序里的原生组件层级是最高的, 不可被覆盖. 并且无法在 `scroll-view、swiper、picker-view、movable-view` 中使用. 会产生各种莫名其妙的问题, 例如在滚动时诡异的表现.  

## 14.1. input  

普通的 `input` 输入框, 看似没有这些问题, 那是小程序做了一定的处理. 大概的原理就是, 在非输入状态下,  `input` 只是一个普通的元素, 点击的时候就会变成正常的 `input` .  
这样解决了原生组件的样式问题, 但是却带来了一个新的**bug**, 聚焦和非聚焦状态下的 `input` 表现不一致, 切换时会出现文字跳动的问题, 这个**bug**暂时无解.  
获取 `input` 文字内容有两种方法 `bindinput` 和 `bindblur` . 在具体使用 `input` 时,  `bindblur`有这样一个表现, 在 `input` 输入文字后, 直接点击 `button` 进行操作,  `bindblur`  会不触发. 所以在登陆或者搜索等操作时, 还是使用 `bindinput` 获取吧.  
 `input` 有5种类型, 其中4种type和1个password.  
> type值  

| 值     | 说明               |
| ------ | ------------------ |
| text   | 文本输入键盘       |
| number | 数字输入键盘       |
| idcard | 身份证输入键盘     |
| digit  | 带小数点的数字键盘 |

`type`可以直接改变键盘布局(大部分情况下).  
`password`为`true`时, 会将输入内容会表现为实心圆, 输入法会切换为**英文状态(安卓)|英文输入(ios)**. 这时候要注意, 如果你更改`password`属性为`false`, 会变为普通的`text`类型, 这时候是可以输入中文的, 所以最好添加一个中文验证.  
```js
if (/[\u4e00-\u9fa5]/.test(PWNum)) {
    that.setData({
      PWFlag: false,
    });
    Tip('密码不能有中文');
  } 
```  
`confirm-type`可以设置键盘右下角按钮的文字，仅在type='text'时生效.  
> confirm-type 有效值： 
> 
| 值     | 说明                   |
| ------ | ---------------------- |
| send   | 右下角按钮为“发送”   |
| search | 右下角按钮为“搜索”   |
| next   | 右下角按钮为“下一个” |
| go     | 右下角按钮为“前往”   |
| done   | 右下角按钮为“完成”   |
- 注：confirm-type的最终表现与手机输入法本身的实现有关，部分安卓系统输入法和第三方输入法可能不支持或不完全支持.  
  
`bindconfirm`点击完成按钮时触发, 写法和bindtap相同, 可以接收一个e: `event.detail = {value: value}`.  
微信小程序的`placeholder`样式通过`placeholder-class`属性, 接收一个**class类名**或`placeholder-style`属性, 使用**行内样式**.  
`input`还有`value`属性, 输入框的内容; `disabled`属性, 是否禁用等.  

## 14.2. textarea  

textarea多行文本输入的使用方法基本和input一致, 但是有一点, 小程序并没有对齐进行额外的处理, 所以在非聚焦情况下, textarea的层级永远最高, 还会出现诡异的滚动情况.  
- tip: 不建议在多行文本上对用户的输入进行修改，所以 textarea 的 bindinput 处理函数并不会将返回值反映到 textarea 上。  

为了能有更好的表现效果, 在有滚动和遮罩的场景, 建议非聚焦情况下使用`view`做一个展示效果, 通过`wx:if`切换`view`与`textarea`, 更改`focus`属性为`true`获取焦点.  

## 14.3. css文字换行  

提到了文字输入, 补充一下文字换行的css小知识.  
> [你真的了解word-wrap和word-break的区别吗？](https://www.cnblogs.com/2050/archive/2012/08/10/2632256.html)  

文字换行css有两个属性`word-wrap: break-word`和`word-break: break-all`.  
简单来说, 当一个单词(特指英文单词)过长, 一行放不下后, 浏览器(包括微信小程序)会另起一行来放置这个单词. 然而, 当第二行还是放不下怎么办? 对不起, 单词就超出去啦!  
`word-wrap: break-word`会使所有的'**行**', 都带上超出换行的属性, 直到他不再超出. 并不会改变默认的**另起一行效果**.  
而`word-break: break-all`就是简单粗暴的, 以每一个字母来当做换行的**标识**, 这时候就不存在超出另起一行的表现了. 换句话来说就是, 已经没有单词这个概念了, 每一个字母就是一个单词.  
`word-break: break-all`除了opera外，其他都支持.  

## 14.4. 文字超出省略  

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

## 14.5. 文字两端对齐  

>[小技巧|CSS如何实现文字两端对齐](https://segmentfault.com/a/1190000011336392)  

css有一个文字两端对齐的属性`text-align: justify`, 它可以在多行文字(中英数字符号交错)导致一行不能很好的占满时, 实现两端对齐的效果. 但是有个问题是, 在一行没有占满时, 这个属性就不起作用了.  
想要让它起效, 需要添加一个行内空标签.  
```html
<div>添加一个行内空标签就有效了！<i></i></div>
```
```css
div i{
  display:inline-block;
  /*padding-left: 100%;*/
  width:100%;
}
```
- padding-left: 100%和width:100%都可以达到效果，选用其一即可  

也可改用after、before伪元素  
```css  
div::after {
  content: " ";
  display: inline-block;
  width: 100%;
}
```

## 14.6. inline-block元素设置overflow:hidden属性导致相邻行内元素向下偏移  

>[inline-block元素设置overflow:hidden属性导致相邻行内元素向下偏移](https://blog.csdn.net/iefreer/article/details/50421025)  

常用的解决方法是为上述inline-block元素添加vertical-align: bottom。  

# 15. 又爱又恨的scroll-view  
说起`scroll-view `, 那可真是让人又爱又恨, 爱它封装了好多方法, 使用就能解决大部分问题, 甚至`scroll-view `组件还能触发`ios`的Q弹特效. 恨它bug无数, 不知什么时候就一头栽坑里爬不出来.  
> 贴张属性表  

| 属性名                | 类型            | 默认值 | 说明                                                                                          |
| --------------------- | --------------- | ------ | --------------------------------------------------------------------------------------------- |
| scroll-x              | Boolean         | false  | 允许横向滚动                                                                                  |
| scroll-y              | Boolean         | false  | 允许纵向滚动                                                                                  |
| upper-threshold       | Number / String | 50     | 距顶部/左边多远时（单位px，2.4.0起支持rpx），触发 scrolltoupper 事件                          |
| lower-threshold       | Number / String | 50     | 距底部/右边多远时（单位px，2.4.0起支持rpx），触发 scrolltolower 事件                          |
| scroll-top            | Number / String |        | 设置竖向滚动条位置（单位px，2.4.0起支持rpx）                                                  |
| scroll-left           | Number / String |        | 设置横向滚动条位置（单位px，2.4.0起支持rpx）                                                  |
| scroll-into-view      | String          |        | 值应为某子元素id（id不能以数字开头）。设置哪个方向可滚动，则在哪个方向滚动到该元素            |
| scroll-with-animation | Boolean         | false  | 在设置滚动条位置时使用动画过渡                                                                |
| enable-back-to-top    | Boolean         | false  | iOS点击顶部状态栏、安卓双击标题栏时，滚动条返回顶部，只支持竖向                               |
| bindscrolltoupper     | EventHandle     |        | 滚动到顶部/左边，会触发 scrolltoupper 事件                                                    |
| bindscrolltolower     | EventHandle     |        | 滚动到底部/右边，会触发 scrolltolower 事件                                                    |
| bindscroll            | EventHandle     |        | 滚动时触发，event.detail = {scrollLeft, scrollTop, scrollHeight, scrollWidth, deltaX, deltaY} |

- 使用竖向滚动时，需要给`<scroll-view>`一个固定高度，通过 WXSS 设置 height。  

## 15.1. 回到顶部和锚点跳转  

使用`scroll-top="0"`即可实现回到顶部的功能, 想要动画过渡添加`scroll-with-animation="{{true}}"`.  
锚点跳转, `scroll-into-view="{{intoView}}"`直接填入需要跳转的元素id即可实现, 注意此时的元素id不加`'#'`. 

## 15.2. 竖向滚动问题  

文档中有提, `scroll-view`需要有一个高度, 才可以属性滚动, 这个高度是指`scroll-view`这个组件的显示高度. 但是我遇到过一个问题, 在iphone8(具体ios版本号记不得了)里, 设置`height: 100%`不能触发竖向滚动, 目前我是给`scroll-view`添加一个`min-height:*rpx`来解决问题.  
我遇到了了一个在`scroll-view`的父元素是`flex1`自适应元素的时候, 要给该元素添加一个`display: flex`, 否则无法触发滚动问题. 不知是我的写法有误还是什么.  

## 15.3. 获取滚动高度问题  

使用`bindscroll`既可获取'scrollTop'等属性, 但是它有一个问题, 就是反应特别慢, 特别是开发工具里(真机还好), 有时你滚了半天了, 才堪堪触发效果. 使用js判断在只在条件达到时`setData`可以好上那么一点儿. 
获取 SelectorQuery 对象实例可以在真机上效果稳定.  
> [SelectorQuery wx.createSelectorQuery()](https://developers.weixin.qq.com/miniprogram/dev/api/wx.createSelectorQuery.html)  
```js  
const query = wx.createSelectorQuery()
query.select('#the-id').boundingClientRect()
query.selectViewport().scrollOffset()
query.exec(function(res){
  res[0].top       // #the-id节点的上边界坐标
  res[1].scrollTop // 显示区域的竖直滚动位置
})
```  

## 15.4. 上拉加载

`bindscrolltolower`可以轻松的实现上拉加载数据的功能, 只需在触发`bindscrolltolower`进行'pageNO++'重新请求拼接结果即可.  
>[微信小程序之下拉加载和上拉刷新](https://www.cnblogs.com/simba-lkj/p/6274232.html)  
>[微信小程序实战篇-下拉刷新与加载更多](https://blog.csdn.net/u012927188/article/details/73369201/)  

## 15.5. 在有遮罩层时, 可以滚动  

如果出现在遮罩弹出的时候, `scroll-view`滚动穿透的问题, 在弹出时设置`scroll-x`或`scroll-y`为`false`即可.  

# 16. 路由  

> [路由](https://developers.weixin.qq.com/miniprogram/dev/api/wx.navigateBack.html)  

微信小程序的路由具有10层的限制, 超出会出问题.  
在需要的时候, 可以`wx.reLaunch(Object object)`清除. 但是不要滥用.  
我在使用路由的时候, 遇到过一个诡异的情况, 在从tabbar页切换到其它页面时, 莫名的加载了另一个tabbar页, `wx.navigateBack(Object object)`也可正常返回. 但是被额外加载的tabbar页`scroll-view`的`scroll-top`失效了, 不得已使用`reLaunch`切换, 算是解决了问题.  

# 17. z-index(层叠上下文)  

![著名的七阶层叠水平](https://github.com/WoolBallLi/wxmini-Ithink/img/7.png)  

> [深入理解 CSS 属性 z-index](https://github.com/happylindz/blog/issues/11)  

> 摘自[MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context)  

文档中的层叠上下文由满足以下任意一个条件的元素形成： 
- 根元素 (HTML),
- z-index 值不为 "auto"的 绝对/相对定位，
- 一个 z-index 值不为 "auto"的 flex 项目 (flex item)，即：父元素 - display: flex|inline-flex，
- [opacity](https://developer.mozilla.org/zh-CN/docs/Web/CSS/opacity) 属性值小于 1 的元素（参考 [the specification for opacity](https://www.w3.org/TR/css-color-3/#transparency)），
- [transform](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform) 属性值不为 "none"的元素，
- [mix-blend-mode](https://developer.mozilla.org/zh-CN/docs/Web/CSS/mix-blend-mode) 属性值不为 "normal"的元素，
- [filter](https://developer.mozilla.org/zh-CN/docs/Web/CSS/filter)值不为“none”的元素，
- [perspective](https://developer.mozilla.org/zh-CN/docs/Web/CSS/perspective)值不为“none”的元素，
- [isolation](https://developer.mozilla.org/zh-CN/docs/Web/CSS/isolation) 属性被设置为 "isolate"的元素，
- position: fixed
- 在 [will-change](https://developer.mozilla.org/zh-CN/docs/Web/CSS/will-change) 中指定了任意 CSS 属性，即便你没有直接指定这些属性的值（参考 这篇文章）
- [-webkit-overflow-scrolling](https://developer.mozilla.org/zh-CN/docs/Web/CSS/-webkit-overflow-scrolling) 属性被设置 "touch"的元素