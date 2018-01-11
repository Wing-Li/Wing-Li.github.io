---
title: ViewFlipper探索与使用
excerpt: 顺便实现Android图片轮播
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

前段时间偶然看到一个使用 ViewFlipper 实现图片轮播的。<br />
我承认，之前我是没有听过 ViewFlipper 这个东西的。那么我脑海中就出现了一个问题：ViewFlipper 是个什么东西？为什么继承它能实现图片的轮播。然后有了之后的探索，我们一起来看看。

<hr />
### 分析 ViewFlipper 
在 studio 中 F4 查看层级关系，经过一番寻找之后有了下图。<br />
呦呵，看到了熟人。ViewFlipper 我没见过，但是我见过 TextSwitcher 和 ImageSwitcher 啊，他两都是内容改变时有个动画的效果。

![](http://upload-images.jianshu.io/upload_images/1689895-092513166c90f372.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 分析 TextSwitcher 、ImageSwitcher 

##### 层级关系
首先看上图最右边的层级关系，通过 ViewFlipper 找到了 TextSwitcher 和ImageSwitcher 。他们有一个共同的父类 ViewAnimator ，也就是说他们肯定是有关联的，而且应该是很相近。

##### 提供的功能
既然 TextSwitcher 和 ImageSwitcher ，我们认识，那么我们就小小的分析一下这两个类。他两提供的功能基本相同，只不过一个针对文字，一个对图片。这两个类本身也是非常简单，提供的方法在左边的目录可以看到，还有一些在父类里。

##### 内容
因为类本身比较简单，所以类里面的代码也没有多少。 <br />
我们看中间的代码窗口，除了构造器剩余的方法一目了然。 <br />
可以发现，最终都是调用了 showNext() 显示切换后的内容，而 showNext() 是由他们的共同父类 ViewAnimator 执行的，而 ViewAnimator 本身就是一个管理动画的类。

也就是说，我们今天的主角 ViewFlipper 最终应该也是调用 showNext() 来执行动画的。 <br />
那 ViewFlipper 跟他们到底有什么区别呢？ <br />
我们来看下面的图，对 ViewFlipper 的分析。

![](http://upload-images.jianshu.io/upload_images/1689895-bcf185e742ea6ef8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 层级关系
这个我们之前已经看过了。

##### 提供的功能
可以看到最左边的目录明显比上面要多很多东西（其实上面两个是有个父类帮他们分担了，但是这个功能还是比他多(*^__^*)）。

##### 内容
那最根本的区别到底是什么呢？ <br /> 
看中间的内容，我们就发现，在这个类当中是有个 Handler Message 存在的。也就是说我们可以设置定时播放动画，也正是基于此，该类才比上面多了一些功能。 <br />
开始、结束动画，是否自动播放，间隔时间，都是上面的所不具备的。

**这样一波看下来，我们大概就知道了 ViewFlipper 为什么能用来实现轮播了。**

那 ViewFlipper 到底该怎么使用呢？
<hr />
### 使用 ViewFlipper  实现轮播
嗯.......
还是直接上代码吧，注释很详细一目了然。

    private void setViewFlipper() {
        mViewFlipper = (ViewFlipper) findViewById(R.id.flipper);

        //添加要滚动的View
        mViewFlipper.addView(getImageView(R.drawable.abcde_a));
        mViewFlipper.addView(getImageView(R.drawable.abcde_b));
        mViewFlipper.addView(getImageView(R.drawable.abcde_c));

        //设置开始和结束动画
        mViewFlipper.setInAnimation(this, R.anim.push_up_in);
        mViewFlipper.setOutAnimation(this, R.anim.push_up_out);

        //设置间隔时间
        mViewFlipper.setFlipInterval(3000);

        //动画的监听
        mViewFlipper.getInAnimation().setAnimationListener(new Animation.AnimationListener() {
            @Override
            public void onAnimationStart(Animation animation) {
                //动画开始时
            }

            @Override
            public void onAnimationEnd(Animation animation) {
                //动画结束时
            }

            @Override
            public void onAnimationRepeat(Animation animation) {
                //重复
            }
        });

        //开始轮播
        mViewFlipper.startFlipping();
    }

    private ImageView getImageView(int res) {
        ImageView imageView = new ImageView(this);
        imageView.setBackgroundResource(res);
        return imageView;
    }

有了 setInAnimation() 、 setOutAnimation()，这两个方法的存在，我们就可以设置各种各样的自己想要的动画效果，而其余提供的方法，更是让我们非常方便的控制动画。有没有觉得很不错呢？

你有没有什么好的想法？<br />
可以自己去动手实践看看。

<hr />
### ViewFlipper 常用方法

    setInAnimation      设置View进入屏幕时候使用的动画
    setOutAnimation     设置View退出屏幕时候使用的动画
    showPrevious        显示ViewFlipper里面的上一个View
    showNext            显示ViewFlipper里面的下一个View
    setFlipInterval     设置View之间切换的时间间隔
    startFlipping       使用setFlipInterval方法设置的时间间隔来开始切换所有的View,切换会循环进行
    stopFlipping        停止View切换
    isFlipping          用来判断View切换是否正在进行
    setDisplayedChild   切换到指定子View


<br />
<hr />

之前在群里，听到有人问：已经有 ViewPager 了，ViewFlipper还有没有存在的必要？<br />
看完本篇之后，还有没有这样的疑问？<br />
ViewFlipper 是为了动画而生的，但是 ViewPager 呢？

好了，本篇就到这里。<br />
O(∩_∩)O

**相关代码**：[ViewAnimation](https://github.com/Wing-Li/AndroidPractice/tree/master/ViewAnimation)<br />
https://github.com/Wing-Li/AndroidPractice/tree/master/ViewAnimation

<br />