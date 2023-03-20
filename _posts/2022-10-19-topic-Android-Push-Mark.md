---
title: Android 角标控制
excerpt: 发送通知，桌面数字
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/u/320f9e8f7fc9
感谢您的关注。

**Android实现应用数字角标**
https://juejin.cn/post/7003536061695524877

***
### 热门机型角标文档

#### 华为角标
https://developer.huawei.com/consumer/cn/doc/development/Corner-Guides/30802


#### 小米角标
开发文档：https://dev.mi.com/console/doc/detail?pId=939
角标问题：https://dev.mi.com/console/doc/detail?pId=2321

> **角标与通知栏强关联！**
在应用开启”显示桌面图标角标“权限的前提下，当应用发通知在通知显示后，通知栏会通知桌面更新应用图标角标。角标数值的计算逻辑是，**统计应用在通知栏里显示的除媒体通知、进度条通知和常驻通知外的所有通知，累加其messageCount值**。用户在桌面点击应用图标启动应用时，**会同时隐藏掉应用角标。直到应用发送了新通知**，或者更新了通知的messageCount值后，才会重新显示出角标。


#### OPPO角标
https://open.oppomobile.com/bbs/forum.php?mod=viewthread&tid=2448&extra=page%3D1&aid=11392

> **需要申请！！！**
您好，桌面角标的功能，是需要申请的。您可通过以下链接的申请方式进行申请。
(社区搜索：教你如何给应用软件申请Push角标功能)
或者点击下方链接：
[https://open.oppomobile.com/bbs/ ... e%3D1&amp;aid=11392](https://open.oppomobile.com/bbs/forum.php?mod=viewthread&amp;tid=2448&amp;extra=page%3D1&amp;aid=11392)


#### VIVO 角标
https://dev.vivo.com.cn/documentCenter/doc/459
> **需要用户手动开启！！！**
接入成功后，“桌面图标角标”默认关闭，需要用户手动开启。
开启路径：“设置”-“通知与状态栏”-“应用通知管理”-应用名称-“桌面图标角标”。
未成功接入“桌面图标角标”的应用，无“桌面图标角标”选项。
备注：视OS版本差异，“桌面图标角标”名称可能为“应用图标标记”或“桌面角标”。

***

推送收不到的原因
https://docs.jiguang.cn/jpush/client/Android/android_faq#app