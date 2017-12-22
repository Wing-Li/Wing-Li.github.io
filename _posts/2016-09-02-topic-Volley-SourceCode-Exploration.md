---
title: Volley 源码探索
excerpt: 思想分析
categories:
  - topics
tag: 源码探索  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

一直听说 Volley 源码写的非常精妙，对于高并发处理非常不错。
研究一番之后，发现确实不错。

<hr />
### 先说一说我是怎么看 Volley 的源码的。

> 1. 首先，自己先研究一遍源码，从 Volley 入手，看了个遍，基本上占了一半的时间；
2. 然后根据[Volley 源码解析](http://a.codekk.com/detail/Android/grumoon/Volley%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90)，在重新看一遍代码(占剩余时间的三分之二)；
3. 看完之后，自己再抛开文章，自己重新看一遍代码；
4. 根据思路，写下看完之后的理解。

[Volley 源码解析](http://a.codekk.com/detail/Android/grumoon/Volley%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90) 这篇文章对 Volley 整个项目介绍的非常详细，以及流程图也做的非常清晰，非常有利于结合起来一起看。

针对于 Volley 的源码解析文章特别的多，而且基本都写的不错，如果再写一遍跟抄袭差不多，所以我还是只写出自己看完之后的理解总结。
虽然，写出来好像没什么，但是其中滋味，看完之后 真的会有另一番味道。

<hr />

## Volley 整体流程图

![](http://upload-images.jianshu.io/upload_images/1689895-37ac8c0b5cb4688f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### RequestQueue 工作流程：

##### CacheDispatcher（无限循环）：缓存调度

1. 先看缓存有没有；
        若没有，则将其加入Network 工作队列。

2. 若有，看缓存是否过期；
        若过期，则将其加入Network 工作队列。

3. 若没过期，则返回 Response，不进行网络请求。

##### NetworkDispatcher（无限循环）：网络调度

1. 查看请求是否被取消；
        若取消，则结束这次请求，开始下一次循环，执行下一个请求。

2. 没有取消，查看请求的地址是否有相应；
        无响应，则结束这次请求，开始下一次循环，执行下一个请求。

3. 有响应，则用 Network 进行 request 的请求，返回 Response；

4. 是否需要缓存；
        需要，则存入Cache。
        不需要，则继续往下。

5. 返回给 ResponseDelivery 。

<br />
> **个人理解：**
>
两个循环均是无限循环；<br />
互相存入或获取；<br />
两者互相配合，使网络频繁请求更加高效。

大家去看源码时，以此 为主线去看源码里面的一些东西，相信可以理解的更快一些。
<br />

<hr />

### 关于 Volley 的框架结构
关于这张图，估计只有我能看懂吧， 哈哈哈....

这个是看第一遍的时候，边看边画的，所以不太清晰。

这张图是以 Network 为主线。
而 Volley 的整体设计应该是以 Request 为主线。

![](http://upload-images.jianshu.io/upload_images/1689895-7fe508e7b11e7bf7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<hr />

因为断断续续看了两三天，所以记忆比较深，在此写下思路，以便后来复习时，能够快速建立当时脑中的逻辑模型。

推荐文章：[Volley 源码解析](http://a.codekk.com/detail/Android/grumoon/Volley%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90)

OK。
如果本文有什么问题，请一定指出。
O(∩_∩)O