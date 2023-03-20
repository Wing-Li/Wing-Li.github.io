---
title: 页面返回时，
excerpt: 携带参数
categories:
  - topics
tag: 微信小程序 
---


**需求：** 跳转到一个新的页面输入内容，然后点击确定，返回到上一个页面，获取到刚才输入的内容，开始进行自己的操作。

#### 正常跳转
在小程序页面跳转的时候是通过 wx.navigateTo() 或者 wx.redirect() 并且携带 url ，然后再目标的 Page 页面中获取参数。

正常跳转的逻辑：

    wx.navigateTo({
      url: "/pages/mypage/mypage?a=1&b=2"
    })

跳转之后获取参数：

    Page({
      onLoad: function (options) {
        var a = options.a; // 值：1
        var b = options.b; // 值：2
      }
    })

***
#### 前一个页面获取 第二个页面 输入的内容

##### 1. 使用全局数据存储
这种方法就是将自己想要储存的数据存到 App 对象上，使其成为全局的对象。
这样之后就可以在全局获取此数据了。

我觉得这种办法作用域太大了，如果此参数在很多地方使用，还可以存到这里。但是如果只在这一个页面使用，存到这里有点浪费。

##### 2. 获取上个页面的目标参数，进行赋值
通过小程序提供的 API：getCurrentPages() 可以获取当前页面跳转的栈信息，这个栈存放着我们跳转过的路径，我们可以获取到上一个页面的 page ，然后修改上一个页面的 data 里的数据。
具体代码如下：

    var pages = getCurrentPages();
    var currPage = pages[pages.length - 1];  // 当前页面
    var prevPage = pages[pages.length - 2];  // 上一个页面

    //直接调用上一个页面的setData()方法，把数据存到上一个页面中去
    prevPage.setData({
      mydata: {a:1, b:2}
    })

这样我们就可以让上一个页面获取我们这个页面的数据。

***

这里提供了两种方法，推荐使用第二种。

第一种的作用域太大了；而且相对而言第二种方法逻辑也更清晰一点。