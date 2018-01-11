---
title: Android 开发实用代码收集
excerpt: 持续更新
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/u/320f9e8f7fc9
感谢您的关注。

![](http://upload-images.jianshu.io/upload_images/1689895-246358f9548c446c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

本文都是 Android 开发过程中，非常简单的代码就可以实现的功能，均可直接复制使用。

<hr />
### Android 分享，不使用第三方SDK实现分享功能

**设置包名即可指定到某平台。**<br />
**最好加上异常处理机制，因为部分手机可能会报错。**

    // 纯文字
    try {
        Intent textIntent = new Intent(Intent.ACTION_SEND);
        // 指定分享到QQ，不加这句，会列出所有可分享的 APP
        textIntent.setPackage("com.tencent.mobileqq");
        textIntent.setType("text/plain");
        textIntent.putExtra(Intent.EXTRA_TEXT, "这是一段分享的文字");
        startActivity(Intent.createChooser(textIntent, "分享"));
    } catch (Exception e) {
        ClipboardManager clipboardManager = (ClipboardManager) getSystemService(Context.CLIPBOARD_SERVICE);
        clipboardManager.setPrimaryClip(ClipData.newPlainText("text", "这是一段分享的文字"));
        Toast.makeText(getApplicationContext(),"抱歉，分享出现异常。要分享的文字已复制到您的粘贴板中，你可以直接去想要分享的平台 粘贴 即可。",Toast.LENGTH_SHORT).show();
    }

    // 单张图片
    String path = getResourcesUri(R.drawable.shu_1);
    Intent imageIntent = new Intent(Intent.ACTION_SEND);
    imageIntent.setType("image/jpeg");
    imageIntent.putExtra(Intent.EXTRA_STREAM, Uri.parse(path));
	imageIntent.putExtra(Intent.EXTRA_SUBJECT, ""分享"");//主题
    if (dlgTitle != null && !"".equals(dlgTitle)) { // 自定义标题
        startActivity(Intent.createChooser(imageIntent, dlgTitle));
    } else { // 系统默认标题
        startActivity(imageIntent);
    }

    // 多图片分享
    ArrayList<Uri> imageUris = new ArrayList<>();
    Uri uri1 = Uri.parse(getResourcesUri(R.drawable.dog));
    Uri uri2 = Uri.parse(getResourcesUri(R.drawable.shu_1));
    imageUris.add(uri1);
    imageUris.add(uri2);
    Intent mulIntent = new Intent(Intent.ACTION_SEND_MULTIPLE);
    mulIntent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, imageUris);
    mulIntent.setType("image/jpeg");
    startActivity(Intent.createChooser(mulIntent,"多文件分享"));

<br />
<hr />
### Android 设置文字到剪切板，Android 获取剪切板的文字
实现起来非常非常简单。
除了设置文字，还可以设置其他的。

##### 设置剪切板

    ClipboardManager clipboardManager= (ClipboardManager) getSystemService(CLIPBOARD_SERVICE);
    ClipData clipData = ClipData.newPlainText("test", "我是被复制的内容");
    clipboardManager.setPrimaryClip(clipData);

##### 获取剪切板

    ClipData clipData = mClipboardManager.getPrimaryClip();
    ClipData.Item item = clipData.getItemAt(0);
    String text = item.getText().toString();

##### 我们可以创建以下三种类型的ClipData：
**String：** newPlainText(label, text)<br />
**URI：** newUri(resolver, label, URI)<br />
**Intent：** newIntent(label, intent)

##### Android 监听剪切板的改变
比如你开着淘宝App，在微信里复制了一条淘口令，然后手机顶部就会显示出可以跳转到淘宝的一个提示框，就是监听了我们手机的剪切板。

    clipboardManager.addPrimaryClipChangedListener(new ClipboardManager.OnPrimaryClipChangedListener() {
        @Override
        public void onPrimaryClipChanged() {
            Log.d("提示", "剪切板有变化了，快判断一下是不是我们想要的。 ");
        }
    });

<br />
<hr />
### Android 退出 APP

##### 1. 设置 MainActivity 的加载模式为 singleTask

    android:launchMode="singleTask"

##### 2. 重写 MainActivity 的 onNewIntent()方法

    public static final String TAG_EXIT = "exit";

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        if (intent != null) {
            boolean isExit = intent.getBooleanExtra(TAG_EXIT, false);
            if (isExit) {
                this.finish();
            }
        }
    }

##### 3. 在想退出的地方调用

    Intent intent = new Intent(this,MainActivity.class);
    intent.putExtra(MainActivity.TAG_EXIT, true);
    startActivity(intent);

<br />
<hr />
### Android 调用系统相册选择图片并显示

    //调用系统相册-选择图片
    private static final int IMAGE = 1;
    //所需权限
    // <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

    public void onClick(View v) {
        //调用相册
        Intent intent = new Intent(Intent.ACTION_PICK,
                android.provider.MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
        startActivityForResult(intent, IMAGE);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        //获取图片路径
        if (requestCode == IMAGE && resultCode == Activity.RESULT_OK && data != null) {
            Uri selectedImage = data.getData();
            String[] filePathColumns = {MediaStore.Images.Media.DATA};
            Cursor c = getContentResolver().query(selectedImage, filePathColumns, null, null, null);
            c.moveToFirst();
            int columnIndex = c.getColumnIndex(filePathColumns[0]);
            // 图片地址
            String imagePath = c.getString(columnIndex);
            showImage(imagePath);
            c.close();
        }
    }

<br />
<hr />
### Android 截图、截屏
可以选择性的去掉标题栏，当然，如果你要截得图是全屏的，可以把去掉标题栏那一段删掉。

    private Bitmap shot(Activity activity) {
        //View 是你需要截图的 View
        View view = activity.getWindow().getDecorView();
        view.setDrawingCacheEnabled(true);
        view.buildDrawingCache();
        Bitmap fullScreen = view.getDrawingCache();

        // 获取状态栏高度
        Rect frame = new Rect();
        activity.getWindow().getDecorView().getWindowVisibleDisplayFrame(frame);
        int statusBarHeight = frame.top;

        // 获取屏幕长和高
        int width = activity.getWindowManager().getDefaultDisplay().getWidth();
        int height = activity.getWindowManager().getDefaultDisplay().getHeight();

        // 去掉标题栏
        Bitmap bitmap = Bitmap.createBitmap(fullScreen, 0, statusBarHeight, width, height - statusBarHeight);
        view.destroyDrawingCache();
        return bitmap;
    }

<br />
<hr />
### Android 截取视频某一帧的缩略图
Android 截取视频第一帧的缩略图，就是本方法的最后一行代码，也就是不指定参数，默认截取第一帧。

**注意：**如果指定时间，单位是微妙，不是毫秒。

	/**
	 * 得视频某一帧的缩略图
	 *
	 * @param videoPath 视频地址
	 * @param timeUs 微秒，注意这里是微秒 1秒 = 1 * 1000 * 1000 微妙
	 *
	 * @return 截取的图片
	 */
	public static Bitmap getVideoThumnail(String videoPath, long timeUs) {
		MediaMetadataRetriever media = new MediaMetadataRetriever();
		media.setDataSource(videoPath);
		// 获取第一个关键帧
		// OPTION_CLOSEST 在给定的时间，检索最近一个帧，这个帧不一定是关键帧。
		// OPTION_CLOSEST_SYNC 在给定的时间，检索最近一个关键帧。
		// OPTION_NEXT_SYNC 在给定时间之后，检索一个关键帧。
		// OPTION_PREVIOUS_SYNC 在给定时间之前，检索一个关键帧。
		return media.getFrameAtTime(timeUs, MediaMetadataRetriever.OPTION_CLOSEST_SYNC);

		// 得到视频第一帧的缩略图
        // return media.getFrameAtTime();
	}

<br />
<hr />
### Android 将图片加入系统相册
有很多手机只要新下载了一张图片，就会被自动添加到相册。但是也有一些手机不会，或者比较慢。<br />
我们可以发一个通知，告诉系统我们新加了一张图片：

    String path = "";// 图片路径
    context.sendBroadcast(new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE, Uri.parse("file://" + path)));

<br />
<hr />
### Android 检测某个APP是否安装
这个功能偶尔也会用到，检测安装需要知道目标APP的包名。

    /**
     * 检测某个APP是否安装
     *
     * @param context
     * @param packageName 程序的包名
     */
    public static boolean isAppAvilible(Context context, String packageName) {
        final PackageManager packageManager = context.getPackageManager();// 获取packagemanager
        List<PackageInfo> pinfo = packageManager.getInstalledPackages(0);// 获取所有已安装程序的包信息
        if (pinfo != null) {
            for (int i = 0; i < pinfo.size(); i++) {
                String pn = pinfo.get(i).packageName;
                if (pn.equals(packageName)) {
                    return true;
                }
            }
        }

        return false;
    }

###### 获取包名的方式：
1. 在手机打开APP，在正在运行的程序中找到该APP，然后进去在进程中可以看到包名；
2. 在[酷安](https://www.coolapk.com/)搜索目标APP，网址的最后就是包名。

###### 分享几个
微信：com.tencent.mm<br />
QQ：com.tencent.mobileqq<br />
QQ空间：com.qzone<br />
微博：com.sina.weibo

<br />
<hr />
### Android StrictMode使用(严苛模式)
StrictMode类是可以用来帮助开发者发现代码中的一些不规范的问题，以达到提升应用响应能力的目的。<br />
举个例子来说，如果开发者在UI线程中进行了网络操作或者文件系统的操作，而这些缓慢的操作会严重影响应用的响应能力，甚至出现ANR对话框。<br />
为了在开发中发现这些容易忽略的问题，我们使用StrictMode，系统检测出主线程违例的情况并做出相应的反应，最终帮助开发者优化和改善代码逻辑。

就是系统提供了这种方式可以让我们来使用，让开发者做出质量更好的APP。<br />
我们也可以判断APP是否为正式版，来决定是否执行这段代码。

    //线程策略（ThreadPolicy）
    StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
            .detectDiskReads()//检测在UI线程读磁盘操作
            .detectDiskWrites()//检测UI线程写磁盘操作
            .detectCustomSlowCalls()//发现UI线程调用的哪些方法执行得比较慢
            .detectResourceMismatches()//最低版本为API23  发现资源不匹配
            .detectNetwork() //检测在UI线程执行网络操作
            .penaltyDialog()//一旦检测到弹出Dialog
            .penaltyDeath()//一旦检测到应用就会崩溃
            .penaltyFlashScreen()//一旦检测到应用将闪屏退出 有的设备不支持
            .penaltyDeathOnNetwork()//一旦检测到应用就会崩溃
            .penaltyDropBox()//一旦检测到将信息存到DropBox文件夹中 data/system/dropbox
            .penaltyLog()//一旦检测到将信息以LogCat的形式打印出来
            .permitDiskReads()//允许UI线程在磁盘上读操作
            .build());

    //虚拟机策略（VmPolicy）
    StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
            .detectActivityLeaks()//最低版本API11 用户检查 Activity 的内存泄露情况
            .detectCleartextNetwork()//最低版本为API23  检测明文的网络
            .detectFileUriExposure()//最低版本为API18   检测file://或者是content://
            .detectLeakedClosableObjects()//最低版本API11  资源没有正确关闭时触发
            .detectLeakedRegistrationObjects()//最低版本API16  BroadcastReceiver、ServiceConnection是否被释放
            .detectLeakedSqlLiteObjects()//最低版本API9   资源没有正确关闭时回触发
            .setClassInstanceLimit(MyClass.class, 2)//设置某个类的同时处于内存中的实例上限，可以协助检查内存泄露
            .penaltyLog()//与上面的一致
            .penaltyDeath()
            .build());

<br />
<hr />
未完待续... <br />
本文将持续更新。 <br />
欢迎投稿，一起收录 <br />

相关收集：<br />
[Android开发人员不得不收集的代码](http://www.jianshu.com/p/72494773aace)这个是工具类集合，非常棒