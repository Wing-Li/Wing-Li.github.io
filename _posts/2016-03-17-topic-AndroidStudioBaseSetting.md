---
title: AndroidStudio基本设置
excerpt: AndroidStudio基本设置，以及一些使用小技巧
categories:
  - topics
tag: 开发习惯  
date: 2016-03-17 18:05
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

本文是另一篇文章的衍生。
点击进入另一篇： [记录自己的开发习惯，Chrome插件、studio插件及设置格式迁移](http://www.jianshu.com/p/c4988c8be615)
因为本人最近换了一份工作，正好趁这个机会记录一下自己的一些开发习惯。

##### 拷贝studio设置的格式
**发现了一个好功能：当更换电脑的时候，只需要从家里的studio中导出一个seting.jar包，然后到新的电脑上导入studio就可以 拷贝设置的种种格式了。**

具体使用的办法在 上面的那篇文章里，**第三部分：Android studio设置的格式**

还是上篇文章的那些话：
> 每个人都有一些自己的开发习惯，比如使用的开发工具或者插件，以及收集资料的方式。
写此文是为了保存一下自己的开发习惯，以便于快速的更换开发环境，虽然开发环境肯定不会经常换。但是，如果丢失了一些东西，对于开发过程多多少少还是有一些影响的。

虽然AS的各种，各类大牛都已经写过很多文章了，但是毕竟每个人都有自己的习惯。这里是记录一些我自己的习惯，会一直补充。


##### 界面设置
默认的 Android Studio 为灰色界面，可以选择使用炫酷的黑色界面。
**Settings** --> **Appearance** --> **Theme** ，选择 **Darcula** 主题即可。[![](http://upload-images.jianshu.io/upload_images/1689895-78db927170da9795.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://ask.android-studio.org/uploads/article/20141212/2e9c824339bacc270e07a8b47efc9400.png)

##### 默认文件编码
建议使用 **utf-8** ，中国的 **Windows** 电脑，默认的字符编码为 **GBK** 。**Settings** --> **File Encodings** 。建议将 **IDE Encoding** 、 **Project Encoding** 、 **Properties Fiels** 都设置成统一的编码。[![](http://upload-images.jianshu.io/upload_images/1689895-c855899b6503b2c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://ask.android-studio.org/uploads/article/20141212/2d135e28f1b04d00c2775503a1f09ad4.png)

##### 显示行号
**Settings** --> **Editor** --> **Appearance** ，勾选 **Show line numbers** 。[![](http://upload-images.jianshu.io/upload_images/1689895-60c3118522fe84bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://ask.android-studio.org/uploads/article/20141212/f33d02a14bd68a96650c236139041bd2.png)

##### 自动导入
当你从其他地方复制了一段代码到Android Studio中，默认的Android Studio不会自动导入这段代码中使用到的类的引用。你可以这么设置。
**Settings** --> **Editor** --> **Auto Import** ，勾选 **Add unambiguous improts on the fly** 。[![](http://upload-images.jianshu.io/upload_images/1689895-602269912150833c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)](http://ask.android-studio.org/uploads/article/20141212/cad3a43633d78098097144cedfca91ab.png)

##### 鼠标悬停显示方法说明
eclipse中只要鼠标放在方法上，就会显示出方法的说明。但是在studio中，默认是不显示的。我们可以设置它来显示。
**Editor** --> **General** ；后面的数字 是悬停多久，才显示(单位：毫秒)。

![](http://upload-images.jianshu.io/upload_images/1689895-32284183ff838c33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### studio打开多个项目
在eclipse中多个项目直接导入，然后利用下面的分屏查看，就可以很方便的查看多个项目。
在AndroidStudio打开多的项目，其实也非常简单。如下图

![](http://upload-images.jianshu.io/upload_images/1689895-c1d7f00189d19f4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 分屏查看代码
之前在eclipse用的也挺多的，eclipse直接拖动就可以，studio还以为不行呢。后来还是找到了，而且studio比eclipse还更智能了呢。操作方式在下面
![](http://upload-images.jianshu.io/upload_images/1689895-33cd5c61204140f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
操作方式，在标签点右键：
![](http://upload-images.jianshu.io/upload_images/1689895-f23afa7da560e122.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 删除的类的历史记录（也可以恢复）
在整理项目的时候，有很多多余的文件，我选择性的删除了好多，然后发现在删除当中，错删了一个，如果全部 Ctrl + Z 的话，刚刚好不容易慢慢选出来的就要全部被恢复了。studio也提供了查看删除的历史记录 的功能：
**操作方式：**鼠标点在包名上，进行 如图 操作：
**PS：忽略我的代码，仅为了演示**

![中间有些类误删了](http://upload-images.jianshu.io/upload_images/1689895-5f641772a0753e17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所有的操作，都会被记录下来，如下图。
前面是，我恢复了其中的一些类。
最后一个，是我删除这个包的操作，删除了所有的类，来找我要恢复的类，点击恢复。

![点右键，选择恢复](http://upload-images.jianshu.io/upload_images/1689895-77ce0cda127c88d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 与未修改前的类对比
我们在一个类中，巴拉巴拉修改了一些东西，然后发现写错了。然后就会很惆怅，刚刚到底改了什么东西...... 查看方法跟上面的是一样的（它也属于历史记录）。
**操作方式：**把鼠标点在当前的类里面（焦点在当前类中，如果鼠标焦点在目录的文件名上，就跟上一个功能一样了），然后进行如图操作：
**PS：忽略我的代码，仅为了演示**
![下面是效果图](http://upload-images.jianshu.io/upload_images/1689895-b3d25146b40bc709.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![修改前与未修改的对比，行数都显示的非常清楚。](http://upload-images.jianshu.io/upload_images/1689895-bfd8bbf03469793f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还有，注意看最左边有个目录，这个是我对这个类进行的一些各种修改。
历史记录都会被保存起来。


##### 静态检查代码可能存在的隐患
Android Studio提供了一种静态检查代码的方式,如图下方显示的就是所有提示可能会出错的位置，可以作为参考，根据情况决定是否修改。

**Analyze** --> **Inspect Code...**

![](http://upload-images.jianshu.io/upload_images/1689895-fd527f56fac71266.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 删除主项目中无用的资源文件
来自：[Android打包提速实践](http://www.jianshu.com/p/e456a5ac8613)
项目开发中多少都会存留一些无用的代码和资源，资源越多打包合并资源的时间就越长。然而删除无用的代码对于提升打包速度的作用微乎其微，我们可以利用混淆这一利器在打release包的时候将无用代码一次性剔除掉。对于资源文件，as提供了自动检测失效文件和删除的功能，这个绝对值得一试。
![remove res](http://upload-images.jianshu.io/upload_images/1689895-39e66c803dcff9da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在弹出的对话框中，我强烈建议不要勾选删除无用的id，因为databinding会用到一些id，但这在代码中没有体现，所以as会认为这些id是无用的。如果你删除了这些id，那么就等着编译失败吧。别问我是怎么知道的T_T。顺便说一下，每次做这种操作前记得commit一下，方便做diff。
![](http://upload-images.jianshu.io/upload_images/1689895-cd6e7b8c8a444ed2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### studio检测更新
设置自动升级，默认是只检测更新稳定版。如果想用稳定版的话，就不用修改这个了。改了这个之后就会检测到预览版的更新。
> **一个版本用习惯了，非必要情况下，还是不要更新的好。 一大堆乱七八糟的事情都来了。**
我更新了之后，之前下载的gradle没了。

**Appearance & Behavior** --> **System Settings** --> **Updates** 或者直接
点Help --> Check for update... --> Updates 界面是一样的

![](http://upload-images.jianshu.io/upload_images/1689895-42b2944dbe4b0c29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### Android Studio 查找含有中文字符串的位置
一般敲代码，为了“省时便利”，譬如View、Toast、Log等带有中文字符串的，大多数人都直接在代码（类文件）完成，这也为未来埋下了隐患。
打开全局搜索，利用其查找功能和正则表达式即可找到，下面先献上正则表达式：

    ^((?!(\*|//)).)+[\u4e00-\u9fa5]

![](http://upload-images.jianshu.io/upload_images/1689895-081a19dc564c10aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

***
<br />
##### 快捷键，这个放在最后：根据个人的习惯做一些修改
搜索**Keymap**。以Eclipse为原型，Copy一个自己的。

![](http://upload-images.jianshu.io/upload_images/1689895-9e7b89c47da4e12a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
个人习惯修改了一些：（在搜索框输入前面的，然后修改快捷键。应该是有快捷键重复的，先去把已经有的删除掉，或者修改成别的。点击最右边的搜索图标，可以搜索快捷键。）

修改的：
DeleteLine -> Ctrl+D ：删除行
DuplicateLines：->Ctrl+Alt+M：复制一行，会换行（个人习惯）
ReformatCode ->Ctrl+Shift+F：格式化代码

默认的：
Alt + enter：（alt+/）代码提示
Ctrl + H：全局搜索（手动输入，选中也可以自动带入）
Ctrl + G：选中之后，直接 全局搜索
Ctrl + O：本类成员预览
F2 ： 提示当前类 或 方法的信息。（eclipse中鼠标悬停即可显示）
F4 ： 查看类的继承关系

<br />

**还有一些快捷的，基本很少有人用的。**如下图，大家可能都见过，每次代码提示最下面都是这些东西，但是就是不知道这是干什么的。

"d".var + 回车 --> String d = “d”：快速生成 变量 及 变量名；（eclipse的快捷键是：Ctrl+2 然后右下角弹出选择后按 L。）
"d".null + 回车 --> if ("" == null) {    }：快速 生成 判空
"d".cast + 回车 --> (() "d")：快速生成 强制转换

等等等等，上面只是给一个例子，实际用起来会快很多的。尤其是第一个，本人在eclipse中经常用到，一直愁于AS快捷键到底是啥。终于在一个大神视频中才挖出来的。去尝试一下，你就知道了，保证你说：“爽”。

![](http://upload-images.jianshu.io/upload_images/1689895-08f4cc2babc11199.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###### 关于快捷键


![对照着看](http://upload-images.jianshu.io/upload_images/1689895-cd1c6fe5bdd502a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<br >
***
<br >

可能最新版的AS 跟上面的图上不太一样，不过不影响，都是直接用 搜索的。

有很多文章都是列出了关于AS的很多设置，但是每个人可能都用不了那么多，毕竟很多默认的配置也是很不错的。

本文会持续修改，感谢关注。