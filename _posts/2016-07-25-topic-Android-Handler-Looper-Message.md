---
title: Android消息处理机制简单分析
excerpt: Handler,Looper,MessageQueue,Message
categories:
  - topics
tag: 源码探索  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

首先说明：本文不会讲的太多，只是简单分析了一下，网上关于 Android消息处理 的介绍已经非常多了，而且都讲的挺好的，就不重复的制造轮子了。
我呢，也是看了看源码，然后在此记录，便于以后回顾。亲自去看一看，会比看别人的文章好很多倍，记忆也会更加深刻。即使忘了，回忆起来也会非常快。

分享一篇文章：
http://www.cnblogs.com/codingmyworld/archive/2011/09/14/2174255.html
只不过这篇文章是2011年9月写的，有点早。
去年我也是把这篇文章看了好多遍的。

<hr />

对于 Message，MessageQueue，Looper ，Handler 的简单分析，
如下图：


![](http://upload-images.jianshu.io/upload_images/1689895-e310e398db030ba7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 昨天有人问了个问题：在子线程中new Handler()会怎么样？

下图就是答案，因为在子线程中，没有 Looper ，必须手动调用 Looper.prepare(); 之后，才能把本线程升级为 Looper线程。

**也正是这个问题，验证了第一张图里面的结论，真正做事的人，是 Message，MessageQueue，Looper 这帮人，Handler 只是更方便的管理他们。
而 Looper 就是核心类。**

![](http://upload-images.jianshu.io/upload_images/1689895-76129ba89d129e74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 那为什么在主线程中我们都没有手动调用 Looper.prepare(); 也能使用 Handler 呢？
看下面这张图，知道答案了吧？


![](http://upload-images.jianshu.io/upload_images/1689895-789d9bfb5992575e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br />
<hr />
收藏一些好文章：
[Android消息机制的原理及源码解析](http://www.jianshu.com/p/f10cff5b4c25)