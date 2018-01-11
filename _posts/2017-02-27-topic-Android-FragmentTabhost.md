---
title: 为 FragmentTabhost 添加 tab 点击事件，在页面跳转之前
excerpt: 自定义布局
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/u/320f9e8f7fc9
感谢您的关注。

利用 FragmentTabhost 来实现页面的切换，是我们经常用到的功能。最近有这么一个需求，底部的切换按钮中的一个点击要跳转到另一个APP，而不是跳转到本APP页面。

如下图：

![](http://upload-images.jianshu.io/upload_images/1689895-c62d00ab1b50fc38.gif?imageMogr2/auto-orient/strip)


<hr />

调查了之后，发现 FragmentTabhost 只能 onTabChanged 切换，在切换之前并没有提供相应的点击事件。<br />
然后就去看了看源码，源码也是挺简单的，随即自己写了一个类继承 FragmentTabhost ，在 onTabChanged 执行操作之前，写一个点击事件的监听。<br />
代码非常的简单：

创建一个 FragmentTabHost 类：

    public class FragmentTabHost extends android.support.v4.app.FragmentTabHost {

        private OnTabClickListener mOnTabClickListener = null;
        private boolean isClickLinstener = false;


        public FragmentTabHost(Context context) {
            super(context);
        }


        public FragmentTabHost(Context context, AttributeSet attrs) {
            super(context, attrs);
        }


        @Override
        public void onTabChanged(String tabId) {
            if (null != mOnTabClickListener) {
                isClickLinstener = mOnTabClickListener.onTabClick(tabId);
            }

            if (!isClickLinstener) {
                super.onTabChanged(tabId);
            }
        }


        public void setOnTabClickListener(OnTabClickListener mOnTabClickListener) {
            this.mOnTabClickListener = mOnTabClickListener;
        }


        public interface OnTabClickListener {
            /**
             * If you set the click event, according to the return value of the click event to determine whether to continue to perform
             *
             * @param tabId tabId
             * @return true：Interception event；false：Superclass continue
             */
            boolean onTabClick(String tabId);
        }
    }

然后，在是使用 FragmentTabHost 时，要用我们重写的这个类。

<hr />
不想自己重写的话，可以引入一个 Gradle 库：

	dependencies {
	    compile 'com.lyl.tabhost:tabhost:1.0.0'
	}

然后使用 com.lyl.tabhost.FragmentTabHost 即可。<br />
Demo地址：https://github.com/Wing-Li/FragmentTabhost

<br />

**注意：**
在 activity_main.xml 中写 FragmentTabHost 控件时，设定的资源id只能是 @android:id/tabhost 。<br />
例：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout
	    android:id="@+id/activity_main"
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:orientation="vertical">
	
	    <FrameLayout
	        android:id="@+id/main_content"
	        android:layout_width="match_parent"
	        android:layout_height="0dp"
	        android:layout_weight="1"/>
	
	    <com.lyl.tabhost.FragmentTabHost
	        android:id="@android:id/tabhost"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content">
	    </com.lyl.tabhost.FragmentTabHost>
	</LinearLayout>



<hr />

项目地址为：<br />
https://github.com/Wing-Li/FragmentTabhost
