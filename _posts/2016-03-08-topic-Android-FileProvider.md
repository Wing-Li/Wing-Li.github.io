---
title: FileProvider 共享文件、缓存
excerpt: Android 推荐
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

在一个项目中突然看到了如下的代码，就很好奇这个东西是这么用的。然后搜了搜，也没发现什么讲这个东西的。

官方是这样说的 ：FileProvider 是一个特殊的 ContentProvider 的子类，它使用 content:// Uri 代替了 file:/// Uri. ，更便利而且安全的为另一个app分享文件。

        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.android.ted.gank.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/paths"/>
        </provider>

官方也提供了一个非常简单的例子：

##### 1. 在AndroidManifest.xml里面配置

	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	    package="com.example.myapp">

	    <permission
		    android:name="com.example.myapp..ACCESS_UPDATE_RESULT"
	    	android:protectionLevel="signature"/>

	    <uses-permission android:name="com.example.myapp.ACCESS_UPDATE_RESULT"/>

	    <application
	    ...>
        <!--在这里定义共享信息-->
	    <provider
	    	android:name="android.support.v4.content.FileProvider"
	    	android:authorities="com.example.myapp.fileprovider"
	    	android:grantUriPermissions="true"
	    	android:exported="false">
	      	<meta-data
	          	android:name="android.support.FILE_PROVIDER_PATHS"
	          	android:resource="@xml/filepaths" />
	    </provider>
	    ...
	    </application>
	</manifest>


**注意要添加权限**
我们可以看到在<meta-data中，定义了一个资源路径，然后就是第二步

##### 2.创建res/xml/filepaths.xml文件

	<paths>
	    <files-path path="images/" name="myimages" />
	</paths>
	
	其中属性的意思：
    path=“images/” 就是你所要共享的文件路径。
    name="myimages" 就是告诉FileProvider 用 myimages 添加进URIs 内容字段去访问 files/images/ 的子目录。

在这个文件中，为每个目录添加一个XML元素指定目录。
paths 可以添加多个子路径：

    <files-path> 分享app内部的存储；
    <external-path> 分享外部的存储；
    <cache-path> 分享内部缓存目录。(我遇到的就是分享的缓存)


##### 3.然后就可以通过URI访问app 的文件了
      content://com.example.myapp.fileprovider/myimages/default_image.jpg

可以看到：<br />
com.example.myapp.fileprovider：前面是我们在AndroidManifest.xml中指定的；<br />
myimages：是我们指定的 name；<br />
default_image.jpg：就是我们想要访问的图片了。

**例如，我看到到这个项目，分享的是缓存路径下的图片，然后用Uri让系统的壁纸来打开自己项目的图片。**

            //得到缓存路径的Uri
            Uri contentUri = FileProvider.getUriForFile(getActivity(), "com.android.ted.gank.fileprovider", file);
            //壁纸管理的意图
            Intent intent = WallpaperManager.getInstance(getActivity()).getCropAndSetWallpaperIntent(contentUri);
            //开启一个Activity显示图片，可以将图片设置为壁纸。调用的是系统的壁纸管理。
            getActivity().startActivityForResult(intent, ViewerActivity.REQUEST_CODE_SET_WALLPAPER);

<hr />

**如果哪里有什么问题，请一定批评指正。**

![来一张图](http://upload-images.jianshu.io/upload_images/1689895-9d5dbea88a5f6728.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)