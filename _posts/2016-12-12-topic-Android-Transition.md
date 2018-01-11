---
title: Android 炫酷的Activity切换效果，共享元素
excerpt: Transition
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

本文原项目地址为：<br />
https://github.com/lgvalle/Material-Animations <br />
在文章最后，有我自己对着这个项目手敲的一份。 <br />
代码基本一模一样，只有略微的修改，加了一些注释，以及将其中大多数英文翻译成了中文。

***

**此篇 API 均为 Android 5.0（API 级别 21） 以上才可支持。**
此demo一共分为四部分：

> 1.1 普通过渡 Transition； <br />
> 1.2 Shared Elements Transition 共享元素； <br />
> 2.0 TransitionManager 控制动画； <br />
> 3.0 ViewAnimationUtils 显示或隐藏效果。

### 过渡效果 Transition
Material Design 为应用中的切换页面时，提供了非常优雅的视觉切换效果。<br />
您可为进入、退出转换、页面之间的共享元素转换设置特定的动画。

#####  1. Transition 动画都包含哪些？

Android 5.0（API 级别 21）支持的进入与退出转换有三个：

| Explode | Slide | Fade |
|:------:|:------:|:------:|
|从中心移入或移出|从边缘移入或移出|调整透明度产生渐变|
|![](http://upload-images.jianshu.io/upload_images/1689895-4c839936d9139327.gif?imageMogr2/auto-orient/strip)|![](http://upload-images.jianshu.io/upload_images/1689895-0bc9c94a12b1e000.gif?imageMogr2/auto-orient/strip)|![](http://upload-images.jianshu.io/upload_images/1689895-ff586e54872bcd4e.gif?imageMogr2/auto-orient/strip)|

一会看到使用场景的时候，就会发现上面的三张图，每张图都经历了：(此处可以一会再回过头来看)

    退出 -> 进入  -> 返回   -> 重新进入
    Exit -> Enter -> Return -> Reenter

    **第一个页面设置：**
    android:windowExitTransition      启动新 Activity ，此页面退出的动画
    android:windowReenterTransition   重新进入的动画。即第二次进入，可以和首次进入不一样。
    **第二个页面设置：**
    android:windowEnterTransition     首次进入显示的动画
    android:windowReturnTransition    调用 finishAfterTransition() 退出时，此页面退出的动画

    如此即可达到以上效果。


explode：从场景的中心移入或移出 <br />
slide：从场景的边缘移入或移出 <br />
fade：调整透明度产生渐变效果

这三个类都继承于 Transition ，所有有一些属性都是共同的。 <br />
常用属性如下：

    // 设置动画的时间。类型：long
    transition.setDuration();
    // 设置修饰动画，定义动画的变化率，具体设置往下翻就看到了
    transition.setInterpolator();
    // 设置动画开始时间，延迟n毫秒播放。类型：long
    transition.setStartDelay();
    // 设置动画的运行路径
    transition.setPathMotion();
    // 改变动画 出现/消失 的模式。Visibility.MODE_IN:进入；Visibility.MODE_OUT：退出。
    transition.setMode();

    // 设置动画的监听事件
    transition.addListener()

至于例子，在下一个给出。<br />
喏，这不就是了。

##### 2. 修饰动画，定义动画的变化率（Interpolator）

在 Java 代码中定义：

	Explode transition = new Explode();
    transition.setDuration(500);
    transition.setInterpolator(new AccelerateInterpolator());

在 Xml 资源定义：

	<explode
        android:duration="@integer/anim_duration_long"
        android:interpolator="@android:interpolator/bounce"
        />

可选类型：

    AccelerateDecelerateInterpolator 在动画开始与结束的地方速率改变比较慢，在中间的时候加速

    AccelerateInterpolator  在动画开始的地方速率改变比较慢，然后开始加速

    AnticipateInterpolator 开始的时候向后然后向前甩

    AnticipateOvershootInterpolator 开始的时候向后然后向前甩一定值后返回最后的值

    BounceInterpolator   动画结束的时候弹起

    CycleInterpolator 动画循环播放特定的次数，速率改变沿着正弦曲线

    DecelerateInterpolator 在动画开始的地方快然后慢

    LinearInterpolator   以常量速率改变

    OvershootInterpolator    向前甩一定值后再回到原来位置

##### 3. 设置 Transition 的时机
到底该什么时候，设置什么样的过渡呢？

以下为动画的设置场景：

**首先打开页面A ：**
页面A -> Enter      首次进入

**从 A 打开 B ：**
页面A -> Exit        退出 <br />
页面B -> Enter       首次进入

**从 B 返回 A ：**
页面B -> Return      返回 <br />
页面A -> Reenter     重新进入

可设置的方法如下：

    android:windowContentTransitions				允许使用transitions

    android:windowAllowEnterTransitionOverlap		是否覆盖执行，其实可以理解成前后两个页面是同步执行还是顺序执行

    android:windowAllowReturnTransitionOverlap		与上面相同。即上一个设置了退出动画，这个设置了进入动画，两者是否同时执行。

    android:windowContentTransitionManager			引用TransitionManager XML资源，定义不同窗口内容之间的所需转换。



    android:windowEnterTransition					首次进入显示的动画

    android:windowExitTransition					启动新 Activity ，此页面退出的动画

    android:windowReenterTransition					重新进入的动画。即第二次进入，可以和首次进入不一样。

    android:windowReturnTransition					调用 finishAfterTransition() 退出时，此页面退出的动画



    android:windowSharedElementsUseOverlay			指示共享元素在转换期间是否应使用叠加层。

    android:windowSharedElementEnterTransition		首次进入显示的动画

    android:windowSharedElementExitTransition		启动新 Activity ，此页面退出的动画

    android:windowSharedElementReenterTransition	重新进入的动画。即第二次进入，可以和首次进入不一样。

    android:windowSharedElementReturnTransition		调用 finishAfterTransition() 退出时，此页面退出的动画

以上为 style 中设置属性，在代码中设置为：

     getWindow().setEnterTransition(visibility);
     // 其余的都是类似


##### 4. 跳转页面
至此，用以上知识基本可以设置出绝大多数的过渡效果。<br />
然后，跳转页面跟普通的跳转也有点不一样。

**跳转页面：**

    protected void transitionTo(Intent i) {
        ActivityOptionsCompat transitionActivityOptions = ActivityOptionsCompat.makeSceneTransitionAnimation(this);
        startActivity(i, transitionActivityOptions.toBundle());
    }

**退出页面：**

    private void closeActivity(){
        // 如果定义了 return transition ，将使用 定义的动画过渡
        Visibility returnTransition = buildReturnTransition();
        getWindow().setReturnTransition(returnTransition);

        // 如果没有 return transition 被定义，将使用 反进入 的动画
        finishAfterTransition();
    }

注意：退出时，一定要调用：finishAfterTransition();

<br />
通过以上设置，就能够完成一个基本的过渡效果了。
<hr />
### Shared Elements Transition 共享元素
效果如图：

![Shared Elements Transition](http://upload-images.jianshu.io/upload_images/1689895-b55d7383dc2f0aaa.gif?imageMogr2/auto-orient/strip)


共享元素的各种设置 与 普通Transition 差不多。<br />
来说说不同的地方。

##### 携带需要共享的 View 进行跳转
也就是在跳转的参数中，增加了要共享的 View控件。<br />
代码如下：

    protected void transitionTo(Intent i) {
        ActivityOptions options = ActivityOptions.makeSceneTransitionAnimation(this,
                                                        Pair.create(view1, "agreedName1"),
                                                        Pair.create(view2, "agreedName2"));
        startActivity(i, options.toBundle());
    }

增加的参数的介绍：

    Pair.create（
       View view,      // 本页面要共享的 View
       String resId    // 下一个页面的 View 的 id,注意是 id 的字符串
    ）

##### 页面切换时，动画的效果设置

Android 5.0（API 级别 21）支持转换效果如下：

changeBounds -  改变目标视图的布局边界<br />
changeClipBounds - 裁剪目标视图边界<br />
changeTransform - 改变目标视图的缩放比例和旋转角度<br />
changeImageTransform - 改变目标图片的大小和缩放比例

设置代码：

    Slide slide = new Slide();
    slide.setDuration(500);

    ChangeBounds changeBounds = new ChangeBounds();
    changeBounds.setDuration(500);

    getWindow().setEnterTransition(slide);
    getWindow().setSharedElementEnterTransition(changeBounds);


<br />
Shared Elements Transition 就是 特殊的 Transition 用法，都是一样的。
<hr />
### TransitionManager 控制动画
这个框架可以让一些复杂的动画特别简单的被实现。

![TransitionManager](http://upload-images.jianshu.io/upload_images/1689895-f293a48a7f769d70.gif?imageMogr2/auto-orient/strip)


简单的说明一下步骤：
> 1. 定义需要切换 layout xml页面；
2. 调用 Scene.getSceneForLayout() 保存每个Layout；
3. 调用 TransitionManager.go(scene1, new ChangeBounds()) 切换。

相当于定义了不同的 xml 布局，然后通过简单的调用，就完成了较为复杂的动画。

以下为代码片段：

    private void setupLayout() {
        scene0 = Scene.getSceneForLayout(binding.sceneRoot, R.layout.activity_animations_scene0, this);
        scene1 = Scene.getSceneForLayout(binding.sceneRoot, R.layout.activity_animations_scene1, this);
        scene2 = Scene.getSceneForLayout(binding.sceneRoot, R.layout.activity_animations_scene2, this);
        scene3 = Scene.getSceneForLayout(binding.sceneRoot, R.layout.activity_animations_scene3, this);
        scene4 = Scene.getSceneForLayout(binding.sceneRoot, R.layout.activity_animations_scene4, this);
        binding.sample3Button1.setOnClickListener(this);
        binding.sample3Button2.setOnClickListener(this);
        binding.sample3Button3.setOnClickListener(this);
        binding.sample3Button4.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.sample3_button1:
                TransitionManager.go(scene1, new ChangeBounds());
                break;
            case R.id.sample3_button2:
                TransitionManager.go(scene2, TransitionInflater.from(this).inflateTransition(R.transition.slide_and_changebounds));
                break;
            case R.id.sample3_button3:
                TransitionManager.go(scene3,TransitionInflater.from(this).inflateTransition(R.transition.slide_and_changebounds_sequential));
                break;
            case R.id.sample3_button4:
                TransitionManager.go(scene3,TransitionInflater.from(this).inflateTransition(R.transition.slide_and_changebounds_sequential_with_interpolators));
                break;
        }
    }

只需要定义布局，调用调转，<br />
就实现了，点击四个 Button ，分别切换四个布局。<br />
TransitionManager.go() 中可以设置各种动画。
<br />

***
### CircularReveal 显示或隐藏 的效果

ViewAnimationUtils.createCircularReveal() <br />
当您显示或隐藏一组 UI 元素时，Circular Reveal 可为用户提供视觉连续性

![CircularReveal](http://upload-images.jianshu.io/upload_images/1689895-d44ca7efa847edbc.gif?imageMogr2/auto-orient/strip)

参数说明：

	Animator createCircularReveal (View view, // 将要变化的 View
				int centerX,                  // 动画圆的中心的x坐标
				int centerY,                  // 动画圆的中心的y坐标
				float startRadius,            // 动画圆的起始半径
				float endRadius               // 动画圆的结束半径
	)

显示 View ：

    private void animShow() {
        View myView = findViewById(R.id.my_view);
        // 从 View 的中心开始
        int cx = (myView.getLeft() + myView.getRight()) / 2;
        int cy = (myView.getTop() + myView.getBottom()) / 2;
        int finalRadius = Math.max(myView.getWidth(), myView.getHeight());

        //为此视图创建动画设计(起始半径为零)
        Animator anim = ViewAnimationUtils.createCircularReveal(myView, cx, cy, 0, finalRadius);
        // 使视图可见并启动动画
        myView.setVisibility(View.VISIBLE);
        anim.start();
    }


隐藏 View ：

    private void animHide() {
        final View myView = findViewById(R.id.my_view);
        int cx = (myView.getLeft() + myView.getRight()) / 2;
        int cy = (myView.getTop() + myView.getBottom()) / 2;

        int initialRadius = myView.getWidth();

        // 半径 从 viewWidth -> 0
        Animator anim = ViewAnimationUtils.createCircularReveal(myView, cx, cy, initialRadius, 0);

        anim.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationEnd(Animator animation) {
                super.onAnimationEnd(animation);
                myView.setVisibility(View.INVISIBLE);
            }
        });
        anim.start();
    }

<hr />

因为，这些炫酷的效果都只支持 API 23 以上，所以在我们常用的 APP 中都还不常见。但是效果真的很不错。 <br />

值得大家研究一下。 <br />
此文是我的一个总结。

项目地址： <br />
https://github.com/Wing-Li/Material-Animations-CN

这个项目是我对着原项目手敲的， <br />
基本一模一样，只是加了一些注释，以及将其中英文翻译成了中文。 <br />
大家可以参考参考。