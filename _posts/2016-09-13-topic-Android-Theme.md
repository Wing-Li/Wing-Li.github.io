---
title: Android夜间模式探索
excerpt: 多主题
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

主题功能老早就想研究了，可是看了很多之后，发现想要很好的实现，并没有什么简单的办法。<br />
但是如果要求不高，实现起来还是挺简单的。<br />
一起来看一下。

***

#### 先说说简单的实现步骤

> 1. 在 styles.xml 中定义不同的主题风格；
2. 在 attrs.xml 文件中，定义自己的属性；
3. 在 layout 布局文件中，使用 styles.xml 中定义的属性；
4. 在 Activity 中切换主题，就好了。

然后来看每一步的详细操作：

#### 1. 在 styles.xml 中定义不同的主题风格
也就是定义多种风格的主题，设置不同的颜色等。

	<resources>
		<style name="AppTheme" parent="Theme.AppCompat.Light">
			<!--Toolbar的背景颜色-->
			<item name="colorPrimary">@color/colorPrimary</item>
			<!--StatusBar的颜色-->
			<item name="colorPrimaryDark">@color/colorPrimaryDark</item>
			<!--主标题的字体颜色-->
			<item name="android:textColorPrimary">@color/textColorPrimary</item>
			<!--控制元件的默认状态颜色-->
			<item name="android:colorControlNormal">@color/colorControlNormal</item>
			<!--控制元件在选中状态的颜色-->
			<item name="colorAccent">@color/colorAccent</item>
			<!--Activity的背景颜色-->
			<item name="android:windowBackground">@color/windowBackground</item>
			<!--自定义的属性-->
			<item name="Text_bg_Color">@color/Text_bg_Color</item>
		</style>

		<style name="AppThemeNight" parent="Theme.AppCompat.Light">
			<item name="colorPrimary">@color/NcolorPrimary</item>
			<item name="colorPrimaryDark">@color/NcolorPrimaryDark</item>
			<item name="android:textColorPrimary">@color/NtextColorPrimary</item>
			<item name="android:colorControlNormal">@color/NcolorControlNormal</item>
			<item name="colorAccent">@color/NcolorAccent</item>
			<item name="android:windowBackground">@color/NwindowBackground</item>
			<item name="Text_bg_Color">@color/NText_bg_Color</item>
		</style>
	</resources>

大多时候，系统自带的属性并不够用，所以就需要自定义一些属性，比如说上面的最后一个属性。<br />
自定义方法如下

#### 2. 在 attrs.xml 文件中，定义自己的属性
在values文件夹下，新建attrs.xml文件，定义自己的属性。

    <resources>
        <attr name="Text_bg_Color" format="color"/>
    </resources>

#### 3. 在 layout 布局文件中，使用 styles.xml 中定义的属性
布局文件中，在需要改变色调的位置，设置相应的属性。

![布局文件](http://upload-images.jianshu.io/upload_images/1689895-283493d74dab59c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4. 最后一步，在 Activity 中切换主题

	public class MainActivity extends AppCompatActivity {

		/**
		 * recreate() 会使 Activity 重新刷新，所以 这个标志必须是静态的。
		 */
		private static boolean night = false;

		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			changeTheme();
			setContentView(R.layout.activity_main);

			findViewById(R.id.email_sign_in_button).setOnClickListener(new OnClickListener() {
				@Override
				public void onClick(View view) {
					change();
				}
			});
		}

		private void change() {
			night = !night;
			changeTheme();
			recreate();// 重启 Activity
		}

		/**
		 * 根据标志改变主题
		 */
		private void changeTheme() {
			if (night) {
				setTheme(R.style.AppThemeNight);
			} else {
				setTheme(R.style.AppTheme);
			}
		}
	}


到此，就可以实现一个最基本的切换主题了，这样也是会有一个问题，也就是闪屏。<br />
但是如果要求不是很高，这也是可以满足需求的。
**此项目代码：<br />
[https://github.com/Wing-Li/AndroidPractice/tree/master/ThemeChange](https://github.com/Wing-Li/AndroidPractice/tree/master/ThemeChange)**

<hr />

闪屏也是大家想尽各种办法想要解决的一个问题。

在此写下大家解决的思路。

##### 目前思路是：
1. 利用 setTheme(); 改变主题；
2. 用 recreate() 刷新当前Activity，主题就改变了。

##### 解决闪屏办法：
不再使用 setTheme();<br />
而是将当前 Activity 上的每个控件，分别改成自己设定的颜色。

**对，没错，就是修改每个控件。**
各种解决办法，也就是看使用什么办法可以更方便的修改多个控件。

<hr />

找到了两个切换主题的框架，从源码中很明显的可以看出，他们都是把常用的基本控件重写的一遍，而我们要使用这个框架的时候，也必须使用框架所提供的控件。

当调用了切换主题的方法后，会遍历使用的控件修改相应的颜色，从而达到 不闪屏切换主题的效果。

#### [https://github.com/Bilibili/MagicaSakura](https://github.com/Bilibili/MagicaSakura)
![MagicaSakura](http://upload-images.jianshu.io/upload_images/1689895-1ea07887ba074415.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### [https://github.com/dersoncheng/MultipleTheme](https://github.com/dersoncheng/MultipleTheme)
![MultipleTheme](http://upload-images.jianshu.io/upload_images/1689895-662337e7a8eccd0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<hr />

<br />
如果是有多个主题，用框架实现还能接受，毕竟很麻烦。<br />
但是如果仅仅只是要实现夜间模式，就使用个框架还是不太舒服，而且必须要使用框架带的基本控件。

所以还是有一些其他的方式，目的还是那句话：使用更好的方式修改多个控件的颜色。<br />
这篇文章介绍的方法，感觉不错：

[知乎和简书的夜间模式实现套路](http://www.jianshu.com/p/3b55e84742e5)

前面都大同小异，主要是修改颜色的那一段代码。


将其项目提炼后，只剩修与修改主题相关代码：<br />
[https://github.com/Wing-Li/AndroidPractice/tree/master/SeamlessChangeTheme](https://github.com/Wing-Li/AndroidPractice/tree/master/SeamlessChangeTheme)

<br />
好了，本文就分享到这里。<br />
如果有什么更好的方式，请一定告知。