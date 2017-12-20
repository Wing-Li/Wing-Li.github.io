---
title: Last topic in group
excerpt: last topic
categories:
  - topics
  - my topic group
tag: Web  
date: 3006-09-22 00:02
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/u/320f9e8f7fc9
感谢您的关注。


Android 桌面小部件是我们经常看到的，比如时钟、天气、音乐播放器等等。
它可以让 App 的某些功能直接展示在桌面上，极大的增加了用户的关注度。

**首先纠正一个误区：**
当 App 的小部件被放到了桌面之后，并不代表你的 App 就可以一直在手机后台运行了。该被杀，它还是会被杀掉的。
所以如果你做小部件的目的是为了让程序常驻后台，那么你可以死心了。

**但是！！！**
虽然它还是能被杀掉，但是用户能看的见它了啊，用户可以点击就打开我们的 APP，所以还是很不错的。

***
#### Android 桌面小部件可以做什么？
小部件可以做什么呢？也就是我们需要实现什么功能。

1. 展示。每隔 N 秒/分钟，刷新一次数据；
2. 交互。点击操作 App 的数据；
3. 打开App。打开主页或指定页面。

这三个功能，大概就能满足我们绝大部分需求了吧。

#### 实现桌面小部件需要什么？
如果你从来没有做过桌面部件，那肯定总是感觉有点慌，无从下手，毫无逻辑。
所以，实现它到底需要什么呢？

> 1. **先声明 Widget 的一些属性。**在 res 新建 xml 文件夹，创建 appwidget-provider 标签的 xml 文件。
> 2. **创建桌面要显示的布局。** 在 layout 创建 app_widget.xml。
> 3. **然后来管理 Widget 状态。**实现一个继承 AppWidgetProvider 的类。
> 4. 最后在 **AndroidManifest.xml** 里，将 AppWidgetProvider类 和 xml属性 注册到一块。
> 5. 通常我们会加一个 **Service 来控制 Widget 的更新时间**，后面再讲为什么。

做完这些，如果不出错，就完成了桌面部件。
其实挺简单的，下面就让我们来看看具体的实现吧。
***
# 实现一个桌面计数器
先上效果图：

![](http://upload-images.jianshu.io/upload_images/1689895-c8aef759115d6952.gif?imageMogr2/auto-orient/strip)

#### 1. 声明 Widget 的属性
在 res 新建 xml 文件夹，创建一个 app_widget.xml 的文件。
如果 res 下没有 xml 文件，则先创建。

app_widget.xml 内容如下：

	<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
						android:initialLayout="@layout/app_widget"
						android:minHeight="110dp"
						android:minWidth="110dp"
						android:previewImage="@mipmap/ic_launcher"
						android:resizeMode="horizontal|vertical"
						android:widgetCategory="home_screen|keyguard">

		<!--
		android:minWidth : 最小宽度
		android:minHeight ： 最小高度
		android:updatePeriodMillis ： 更新widget的时间间隔(ms)，"86400000"为1个小时，值小于30分钟时，会被设置为30分钟。可以用 service、AlarmManager、Timer 控制。
		android:previewImage ： 预览图片，拖动小部件到桌面时有个预览图
		android:initialLayout ： 加载到桌面时对应的布局文件
		android:resizeMode ： 拉伸的方向。horizontal表示可以水平拉伸，vertical表示可以竖直拉伸
		android:widgetCategory ： 被显示的位置。home_screen：将widget添加到桌面，keyguard：widget可以被添加到锁屏界面。
		android:initialKeyguardLayout ： 加载到锁屏界面时对应的布局文件
		 -->
	</appwidget-provider>

属性的注释在上面写的很清楚了，这里需要说两点。
1. 关于宽度和高度的数值定义是很有讲究的，在桌面其实是按照“格子”排列的。
看 Google 给的图。上面我们代码定义 110dp 也就是说，它占了2*2的空间。

![](http://upload-images.jianshu.io/upload_images/1689895-25d745085fba9ae0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 第二点很重要。有个 updatePeriodMillis 属性，更新widget的时间间隔(ms)。
官方给提供了小部件的自动更新时间，但是却给了限制，你更新的时间必须大于30分钟，如果小于30分钟，那默认就是30分钟。
可以我们就是要5分钟更新啊，怎么办呢？
所以就不能使用这个默认更新，我们要自己来通过发送广播控制更新时间，也就是一开始总步骤里面第4步，加一个 Service 来控制 Widget 的更新时间，这个在最后一步添加。

#### 2. 创建布局文件
在 layout 创建 app_widget.xml 文件。

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                  android:layout_width="match_parent"
                  android:layout_height="match_parent"
                  android:gravity="center_horizontal"
                  android:orientation="vertical">

        <TextView
            android:id="@+id/widget_txt"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="0"
            android:textSize="36sp"
            android:textStyle="bold"/>

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal">

            <Button
                android:id="@+id/widget_btn_reset"
                style="@style/Widget.AppCompat.Toolbar.Button.Navigation"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="恢复"/>

            <Button
                android:id="@+id/widget_btn_open"
                style="@style/Widget.AppCompat.Toolbar.Button.Navigation"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginLeft="8dp"
                android:text="打开页面"/>
        </LinearLayout>
    </LinearLayout>


这里要注意的就是 **桌面部件并不支持 Android 所有的控件**。
支持的控件如下：

    App Widget支持的布局：
           　　FrameLayout
           　　LinearLayout
           　　RelativeLayout
           　　GridLayout
    App Widget支持的控件：
           　　AnalogClock
           　　Button
           　　Chronometer
           　　ImageButton
           　　ImageView
           　　ProgressBar
           　　TextView
           　　ViewFlipper
           　　ListView
           　　GridView
           　　StackView
           　　AdapterViewFlipper

#### 3. 管理 Widget 状态
这里代码看起来可能有点多，先听我讲几个逻辑，再来看代码。
1. Android 的各种东西都有自己的生命周期，Widget 也不例外，它有几个方法来管理自己的生命周期。

![](http://upload-images.jianshu.io/upload_images/1689895-67dea8f78532c196.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 同一个小部件是可以添加多次的，所以更新控件的时候，要把所有的都更新。

3. onReceive() 用来接收广播，它并不在生命周期里。**但是，其实 onReceive() 是掌控生命周期的。**
如下是 onReceive() 父类的源码，右边是每个广播对应的方法。
上面我画的生命周期的图，也比较清楚。

![](http://upload-images.jianshu.io/upload_images/1689895-a2a97d2fd85460c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们再来看代码。
新建一个 WidgetProvider 类，继承 AppWidgetProvider。
主要逻辑在 onReceive() 里，其他的都是生命周期切换时，所处理的事情。
我们在下面分析 onReceive()。

	public class WidgetProvider extends AppWidgetProvider {

		// 更新 widget 的广播对应的action
		private final String ACTION_UPDATE_ALL = "com.lyl.widget.UPDATE_ALL";
		// 保存 widget 的id的HashSet，每新建一个 widget 都会为该 widget 分配一个 id。
		private static Set idsSet = new HashSet();

		public static int mIndex;

		/**
		 * 接收窗口小部件点击时发送的广播
		 */
		@Override
		public void onReceive(final Context context, Intent intent) {
			super.onReceive(context, intent);
			final String action = intent.getAction();

			if (ACTION_UPDATE_ALL.equals(action)) {
				// “更新”广播
				updateAllAppWidgets(context, AppWidgetManager.getInstance(context), idsSet);
			} else if (intent.hasCategory(Intent.CATEGORY_ALTERNATIVE)) {
				// “按钮点击”广播
				mIndex = 0;
				updateAllAppWidgets(context, AppWidgetManager.getInstance(context), idsSet);
			}
		}

		// 更新所有的 widget
		private void updateAllAppWidgets(Context context, AppWidgetManager appWidgetManager, Set set) {
			// widget 的id
			int appID;
			// 迭代器，用于遍历所有保存的widget的id
			Iterator it = set.iterator();

			// 要显示的那个数字，每更新一次 + 1
			mIndex++; // TODO:可以在这里做更多的逻辑操作，比如：数据处理、网络请求等。然后去显示数据

			while (it.hasNext()) {
				appID = ((Integer) it.next()).intValue();

				// 获取 example_appwidget.xml 对应的RemoteViews
				RemoteViews remoteView = new RemoteViews(context.getPackageName(), R.layout.app_widget);

				// 设置显示数字
				remoteView.setTextViewText(R.id.widget_txt, String.valueOf(mIndex));

				// 设置点击按钮对应的PendingIntent：即点击按钮时，发送广播。
				remoteView.setOnClickPendingIntent(R.id.widget_btn_reset, getResetPendingIntent(context));
				remoteView.setOnClickPendingIntent(R.id.widget_btn_open, getOpenPendingIntent(context));

				// 更新 widget
				appWidgetManager.updateAppWidget(appID, remoteView);
			}
		}

		/**
		 * 获取 重置数字的广播
		 */
		private PendingIntent getResetPendingIntent(Context context) {
			Intent intent = new Intent();
			intent.setClass(context, WidgetProvider.class);
			intent.addCategory(Intent.CATEGORY_ALTERNATIVE);
			PendingIntent pi = PendingIntent.getBroadcast(context, 0, intent, 0);
			return pi;
		}

		/**
		 * 获取 打开 MainActivity 的 PendingIntent
		 */
		private PendingIntent getOpenPendingIntent(Context context) {
			Intent intent = new Intent();
			intent.setClass(context, MainActivity.class);
			intent.putExtra("main", "这句话是我从桌面点开传过去的。");
			PendingIntent pi = PendingIntent.getActivity(context, 0, intent, 0);
			return pi;
		}

		/**
		 * 当该窗口小部件第一次添加到桌面时调用该方法，可添加多次但只第一次调用
		 */
		@Override
		public void onEnabled(Context context) {
			// 在第一个 widget 被创建时，开启服务
			Intent intent = new Intent(context, WidgetService.class);
			context.startService(intent);
			Toast.makeText(context, "开始计数", Toast.LENGTH_SHORT).show();
			super.onEnabled(context);
		}

		// 当 widget 被初次添加 或者 当 widget 的大小被改变时，被调用
		@Override
		public void onAppWidgetOptionsChanged(Context context, AppWidgetManager appWidgetManager, int appWidgetId, Bundle
				newOptions) {
			super.onAppWidgetOptionsChanged(context, appWidgetManager, appWidgetId, newOptions);
		}

		/**
		 * 当小部件从备份恢复时调用该方法
		 */
		@Override
		public void onRestored(Context context, int[] oldWidgetIds, int[] newWidgetIds) {
			super.onRestored(context, oldWidgetIds, newWidgetIds);
		}

		/**
		 * 每次窗口小部件被点击更新都调用一次该方法
		 */
		@Override
		public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
			super.onUpdate(context, appWidgetManager, appWidgetIds);
			// 每次 widget 被创建时，对应的将widget的id添加到set中
			for (int appWidgetId : appWidgetIds) {
				idsSet.add(Integer.valueOf(appWidgetId));
			}
		}

		/**
		 * 每删除一次窗口小部件就调用一次
		 */
		@Override
		public void onDeleted(Context context, int[] appWidgetIds) {
			// 当 widget 被删除时，对应的删除set中保存的widget的id
			for (int appWidgetId : appWidgetIds) {
				idsSet.remove(Integer.valueOf(appWidgetId));
			}
			super.onDeleted(context, appWidgetIds);
		}

		/**
		 * 当最后一个该窗口小部件删除时调用该方法，注意是最后一个
		 */
		@Override
		public void onDisabled(Context context) {
			// 在最后一个 widget 被删除时，终止服务
			Intent intent = new Intent(context, WidgetService.class);
			context.stopService(intent);
			super.onDisabled(context);
		}
	}

##### onReceive(Context context, Intent intent)
它传了两个值回来，Context 是跳转、发广播用的。
我们用来判断的是 Intent ，这里用到了 Intent 的两种方式。

> Intent 作为信息传递者。
它要把信息传给谁，可以有三个匹配依据：一个是action，一个是category，一个是data。

String ACTION_UPDATE_ALL = "com.lyl.widget.UPDATE_ALL";
这个最后会在 AndroidManifest.xml 里面注册时写进去。
当每隔 N 秒/分钟，就发送一次这个广播，更新所有UI。

intent.hasCategory(Intent.CATEGORY_ALTERNATIVE)
是广播事件里携带的 Intent 里设置的，用来匹配。
点击“恢复”按钮，计数器清零。


然后是 updateAllAppWidgets() 这个方法，更新 UI。
更新 UI 用到了一个新东西——RemoteViews。

> 怎么来理解 RemoteViews 呢？
因为，桌面部件并不像平常布局直接展示，它需要通过某种服务去更新UI。但是我们的App怎么能去控制桌面上的布局呢？
所以就需要有一个中间人，类似传递者。
我告诉传递者，你让他把我的 R.id.widget_txt ，更新成 “hello world”。
你让他把我的 R.id.widget_btn_open 按钮点击之后去响应 PendingIntent 这件事。
RemoteViews 就是承担着一个这样的角色。

然后再去理解代码，是不是稍微好一点了？

#### 4. 最后就是 Service 控制 Widget 的更新时间
说好的 当每隔 N 秒/分钟，就发送一次这个广播。
那到底在哪发呢？也就是我们刚开始说的，用 Service 来控制时间。

新建一个 WidgetService 类，继承 Service。代码如下：

    /**
     * 控制 桌面小部件 更新
     * Created by lyl on 2017/8/23.
     */
    public class WidgetService extends Service {

        // 更新 widget 的广播对应的 action
        private final String ACTION_UPDATE_ALL = "com.lyl.widget.UPDATE_ALL";
        // 周期性更新 widget 的周期
        private static final int UPDATE_TIME = 1000;

        private Timer mTimer;
        private TimerTask mTimerTask;


        @Override
        public void onCreate() {
            super.onCreate();

            // 每经过指定时间，发送一次广播
            mTimer = new Timer();
            mTimerTask = new TimerTask() {
                @Override
                public void run() {
                    Intent updateIntent = new Intent(ACTION_UPDATE_ALL);
                    sendBroadcast(updateIntent);
                }
            };
            mTimer.schedule(mTimerTask, 1000, UPDATE_TIME);
        }

        @Nullable
        @Override
        public IBinder onBind(Intent intent) {
            return null;
        }

        @Override
        public void onDestroy() {
            super.onDestroy();
            mTimerTask.cancel();
            mTimer.cancel();
        }

        /*
         *  服务开始时，即调用startService()时，onStartCommand()被执行。
         *
         *  这个整形可以有四个返回值：start_sticky、start_no_sticky、START_REDELIVER_INTENT、START_STICKY_COMPATIBILITY。
         *  它们的含义分别是：
         *  1):START_STICKY：如果service进程被kill掉，保留service的状态为开始状态，但不保留递送的intent对象。随后系统会尝试重新创建service，
         *     由于服务状态为开始状态，所以创建服务后一定会调用onStartCommand(Intent,int,int)方法。如果在此期间没有任何启动命令被传递到service，那么参数Intent将为null;
         *  2):START_NOT_STICKY：“非粘性的”。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统不会自动重启该服务;
         *  3):START_REDELIVER_INTENT：重传Intent。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统会自动重启该服务，并将Intent的值传入;
         *  4):START_STICKY_COMPATIBILITY：START_STICKY的兼容版本，但不保证服务被kill后一定能重启。
         */
        @Override
        public int onStartCommand(Intent intent, int flags, int startId) {
            super.onStartCommand(intent, flags, startId);
            return START_STICKY;
        }
    }

在 onCreate 开启一个计时线程，每1秒发送一个广播，广播就是我们自己定义的类型。

### 5. 在 AndroidManifest.xml 注册 桌面部件 和 服务
然后就只剩最后一步了，注册相关信息

    <!-- 声明widget对应的AppWidgetProvider -->
    <receiver android:name=".WidgetProvider">
        <intent-filter>
            <!--这个是必须要有的系统规定-->
            <action android:name="android.appwidget.action.APPWIDGET_UPDATE"/>
            <!--这个是我们自定义的 action ，用来更新UI，还可以自由添加更多 -->
            <action android:name="com.lyl.widget.UPDATE_ALL"/>
        </intent-filter>
        <!--要显示的布局-->
        <meta-data
            android:name="android.appwidget.provider"
            android:resource="@xml/app_widget"/>
    </receiver>

    <!-- 用来计时，发送 通知桌面部件更新 -->
    <service android:name=".WidgetService" >
        <intent-filter>
            <!--用来启动服务-->
            <action android:name="android.appwidget.action.APP_WIDGET_SERVICE" />
        </intent-filter>
    </service>

相应的注释都在上面，如果我们的App进程被杀掉，服务也被关掉，那就没办法更新UI了。
也可以再创建一个 BroadcastReceiver 监听系统的各种动态，来唤醒我们的通知服务，这就属于进程保活了。

至此，以上代码写完，如果不出问题，运行之后直接去桌面看小工具，我们的App就在里面了，可以添加到桌面。
***

对于需要定时更新的桌面部件，保证自己的服务在后台运行也是一件比较重要的事情。
这个我们还是可以好好做一下，毕竟用户都已经愿意把我们的程序放到桌面上，所以只要友好的引导用户给你一定的权限，存活概率还是很大。
再不济，让用户主动点开App，也不失为一种办法。

> 好的创意才能造就好的App，代码只是实现。

最后放上项目地址：
https://github.com/Wing-Li/Widget
