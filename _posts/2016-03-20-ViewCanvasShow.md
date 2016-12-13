---
title: 最基础的自定义View——用Canvas显示动画
permalink: /topics/my topic group/
categories:
  - topics 
  - my topic group
tags:
  - androidlearn
date: 2016-03-20 21:50
---
> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

用Canvas绘制一个简单的动画，效果图如图所示：

![](https://raw.githubusercontent.com/Wing-Li/PracticeDemos/master/Hack7_CanvasAnime/gif/canvasanime.gif)


黑色的是我们View的区域，可以设定View的宽高。原本是方块，后来我改成了小圆球，我觉得更好看了。嘻嘻...

其实逻辑还是非常简单的，还是先上总结：
> 1. 先创建Rectangle类，一个小圆球，以及它的一些属性
2. 在创建一个DrawView，以特定的规则 来显示Rectangle小圆球，
3. 使用DrawView作为一个View来显示到布局中，就OK了

其实就是 DrawView 类负责在屏幕上绘制 几个 Rectangle 方块，然后不断更新 Rectangle方块 的位置。就形成了动画。

关于Android绘制基本的几何图形，本人另写了一篇文章，大家可以看看，也非常简单：[Android最基本的几何图形](http://www.jianshu.com/p/356a997881a7)

好了，我们来写代码。还是那句话：看起来挺麻烦的，其实是非常简单。跟着看下去，不想写代码直接复制就可以，都是很简单的代码。看完基本就知道他是个怎么回事了。

**我们来写一些关键步骤，具体代码看项目即可，注释的很详细。**

#### 1. 先创建Rectangle类，一个小圆球，以及它的一些属性
###### 1). 在构造器里，创建画笔，设置小圆球的颜色，透明度等属性（初始化）

    public class Rectangle extends View {
                
        /**用来装小球的View，用到了他的宽高，防止圆球滚出控件**/
		private DrawView mDrawView;
		private Paint mInnerPaint;

		public Rectangle(Context context, DrawView drawView) {
			super(context);
			mDrawView = drawView;

			// 设置画笔的属性
			mInnerPaint = new Paint();
			mInnerPaint.setARGB(ALPHA, 255, 0, 0);// 设置成红色
			mInnerPaint.setAntiAlias(true);// 消除锯齿
		}

###### 2). 声明一些属性，作为小圆球的基本属性，然后绘制。

		public static final int MAX_SIZE = 20;
		/** 透明度 **/
		private static final int ALPHA = 255;
		/** 原始位置 **/
		private int mCoordX = 0;
		private int mCoordY = 0;
		/** 方块大小 **/
		private int mRealSize = 20;
		/** 之后的位置 **/
		private int mSpeedX = 3;
		private int mSpeedY = 3;

		@Override
		protected void onDraw(Canvas canvas) {
			super.onDraw(canvas);

			float toX = mCoordX + mRealSize;
			float toY = mCoordY + mRealSize;
		    // 绘制圆形图案
			canvas.drawCircle(toX, toY, mRealSize, mInnerPaint);
		}

###### 3). 以上两步就可以生成小圆球了。 但是我们的小圆球是会动的哦，所以要声明其他的一些方法来设置。

	    /**
	     * 在外部调用此方法即向指定位置移动
	     */
		public void move() {
			moveTo(mSpeedX, mSpeedY);
		}

		private void moveTo(int goX, int goY) {
		// 检查的边界，如果到达边界，改变方向
		if (mCoordX > (mDrawView.width - MAX_SIZE)) {
			goRight = false;
		}
		if (mCoordX < 0) {
			goRight = true;
		}

		if (mCoordY > (mDrawView.height - MAX_SIZE)) {
			goDown = false;
		}
		if (mCoordY < 0) {
			goDown = true;
		}

		// 设置移动的位置
		if (goRight) {
			mCoordX += goX;
		} else {
			mCoordX -= goX;
		}
		if (goDown) {
			mCoordY += goY;
		} else {
			mCoordY -= goY;
		}
  	}

###### 4). 设置完之后，我们还可以设置一些小圆球的基本属性，比如，设置原始位置，每次移动的位置(即速度)，小圆球的半径等等。都是set/get方法。

		public void setARGB(int a, int r, int g, int b) {
			mInnerPaint.setARGB(a, r, g, b);
		}

		public void setX(int newValue) {mCoordX = newValue;}
		public float getX() {return mCoordX;	}

		public void setY(int newValue) {mCoordY = newValue;}
		public float getY() {return mCoordY;	}

		public int getSpeedX() {return mSpeedX;}
		public void setSpeedX(int speedX) {mSpeedX = speedX;}

		public int getmSpeedY() { return mSpeedY;}
		public void setSpeedY(int speedY) {mSpeedY = speedY;}

		public void setSize(int newSize) {mRealSize = newSize;}
		public int getSize() {return mRealSize;}

这样我们就设置完了一个基本的小圆球的类。
> 1. 先在构造器里初始化了一个画笔，设置小圆球的颜色等属性；
2. 然后在设置一些小圆球的默认属性，在onDraw()中绘制；
3. 定义一个move()方法，可以改变这个小球的位置，以便小球运动；
4. 定义小球的各种属性，就是set/get方法。

直接去看项目的的话，代码还是非常简洁的。

#### 2. 在创建一个DrawView，以特定的规则 来显示Rectangle小球
###### 1). 定义全局属性，及初始化构造器

		public class DrawView extends View {
			/** 定义不同的小块 **/
			private Rectangle mRectangle1;
			private Rectangle mRectangle2;
			private Rectangle mRectangle3;
			/** 布局的宽高 **/
			public int width;
			public int height;
			/** 定义了一个开关 **/
			private boolean isRuning;

			private Context mContext;

			/** 程序内实例化时只会调这个方法，只传入Context即可。就是在代码中new 的时候 **/
			public DrawView(Context context) {
				this(context, null);
			}

			/** 用于layout文件实例化，会把XML内的参数通过AttributeSet带入到View内。就是在xml布局中用的时候 **/
			public DrawView(Context context, AttributeSet attrs) {
				this(context, attrs, 0);
			}

			/** 主题的style信息，也会从XML里带入 **/
			public DrawView(Context context, AttributeSet attrs, int defStyleAttr) {
				super(context, attrs, defStyleAttr);
				mContext = context;
				isRuning = true;// 默认小球是运动的
			}
###### 2). 设置控件的宽高给width ,height 。然后创建三个小球。
（这里我感觉不妥，但是宽高只能在这里才能获取到。各位有什么意见）

		@Override
		protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
			super.onMeasure(widthMeasureSpec, heightMeasureSpec);
			// 得到控件的宽高，为了不让小球滚出边界
			width = getMeasuredWidth();
			height = getMeasuredHeight();

			// 在不同位置，创建了三个小球
			mRectangle1 = createF(mContext, 0, 0, 3, 3);
			mRectangle2 = createF(mContext, 0, 300, 7, 10);
			mRectangle3 = createF(mContext, 500, 0, 20, 20);
		}

		private Rectangle createF(Context context, int x, int y, int toX, int toY) {
			Rectangle mRectangle = new Rectangle(context, this);
			mRectangle.setARGB(255, 255, 0, 0);

			// 设置初始位置
			mRectangle.setX(x);
			mRectangle.setY(y);

			// 每次移动的距离，可以看做是速度
			mRectangle.setSpeedX(toX);
			mRectangle.setSpeedY(toY);
			return mRectangle;
		}

###### 3. 然后 改变 小球的位置，并不断重新绘制界面，就会动起来。
最后设定了一个开关。isRuning 为 false 时，停止运动。

		@Override
		protected void onDraw(Canvas canvas) {
			// postInvalidate()子线程请求调用onDraw() ，系统自动调用，不允许手动调用。
			// invalidate();主线程请求调用onDraw()

			if (isRuning) {
				invalidate();// 不断的刷新界面

				mRectangle1.move();// 调用圆块，移动了位置
				mRectangle1.onDraw(canvas);// 重绘界面，即改变了圆块位置

				mRectangle2.move();
				mRectangle2.onDraw(canvas);

				mRectangle3.move();
				mRectangle3.onDraw(canvas);
			}
		}

		public boolean isRuning() {
			return isRuning;
		}

		public void setRuning(boolean isRuning) {
			this.isRuning = isRuning;
		}

	}

这样就完成了这个View的绘制。
还是跟上面的差不多：
> 1. 先初始化构造器，设定一些属性；
2. 定义内部要显示的图案等；
3. 在onDraw()中进行绘制

注意这两个方法：
// postInvalidate();子线程请求调用onDraw() 
// invalidate();主线程请求调用onDraw()

#### 3. 然后直接使用这个View就可以了。


	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent" >

	    <com.lyl.hack_canvasanime.view.DrawView
		android:layout_width="200dp"
		android:layout_height="300dp"
		android:layout_centerInParent="true"
		android:background="#000007" />

	</RelativeLayout>

项目地址：https://github.com/Wing-Li/PracticeDemos

这样就完成了整个动画。
可以完成一些没有交互的动画，比如：加载进度圈。
就是一个最基本的自定义View。
