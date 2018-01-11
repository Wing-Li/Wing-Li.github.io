---
title: 在近期任务列表显示单个APP的多个Activity
excerpt: 类似微信小程序
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/u/320f9e8f7fc9
感谢您的关注。


用过微信小程序的人应该都可以发现，当我们打开一个小程序之后，在近期任务列表里面就会新创建一个页面(MIUI8 好像没有)，感觉这个功能还挺有意思的。<br />
研究一下，发现还挺简单的。<br />
先上一个效果图：

![](http://upload-images.jianshu.io/upload_images/1689895-3e9043802bfe2366.gif?imageMogr2/auto-orient/strip)

<hr />
有两种方式可以实现效果

### 第一种：代码实现
即页面跳转的时候增加 Flag，

    Intent intent = new Intent(this, Main2Activity.class);
    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_DOCUMENT);
    intent.addFlags(Intent.FLAG_ACTIVITY_MULTIPLE_TASK);
    startActivity(intent);

添加两个 Flag 即可，注意关闭的时候要使用:

    finishAndRemoveTask();

**FLAG_ACTIVITY_NEW_DOCUMENT：**此标志用于将文档打开到一个 基于此意图的新任务中；<br />
**FLAG_ACTIVITY_MULTIPLE_TASK：**此标志用于创建新任务并将活动导入其中。

> **注意：使用这种方式，必须具有在清单文件中设置的 android:launchMode="standard" 属性值（默认就是这个属性）**

<hr />
### 第二种：配置 AndroidManifest.xml
在要跳转的 Activit 配置

    <activity
        android:name=".Main3Activity"
        android:documentLaunchMode="intoExisting"
        android:excludeFromRecents="true"
        android:maxRecents="3"/>

AndroidManifest.xml 中的属性：
#### 1. documentLaunchMode(启动模式)：
**intoExisting：**如果之前已经打开过，则会打开之前的(类似于 Activity 的 singleTask)；<br />
**always：**不管之前有没有打开，都新创建一个(类似于 Activity 的 standard)；<br />
**none：**不会在任务列表创建新的窗口，依旧显示单个任务；<br />
**never：**不会在任务列表创建新的窗口，依旧显示单个任务，设置此值会替代 FLAG_ACTIVITY_NEW_DOCUMENT 和 FLAG_ACTIVITY_MULTIPLE_TASK 标志的行为（如果在 Intent 中设置了其中一个标志）。<br />

> **注：对于除 none 和 never 以外的值，必须使用 launchMode="standard" 定义 Activity。如果未指定此属性，则使用 documentLaunchMode="none"。**

#### 2. excludeFromRecents：
默认为 false 。<br />
设置为 true 时，只要你离开了这个页面，它就会从最近任务列表里移除掉。

#### 3. maxRecents：
设置为整型值，设置应用能够包括在概览屏幕中的最大任务数。默认值为 16。达到最大任务数后，最近最少使用的任务将从概览屏幕中移除。 android:maxRecents 的最大值为 50（内存不足的设备上为 25）；小于 1 的值无效。

***

项目在这里：<br />
https://github.com/Wing-Li/AppTask