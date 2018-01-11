---
title: Glide 使用必须知道的基础属性
excerpt: Google 推荐的图片加载库
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

Glide图片加载 已经 是Google官方推荐使用的加载了。
如果把这个适当的用好了，会有出乎意料的效果，本人亲身体验。

#### Gradle

      compile 'com.github.bumptech.glide:glide:4.4.0'

<hr />

##### with(context)，context的重要性

Glide.with(context)；with() 方法中的context到底是哪种类型是不清楚的。有一点很重要需要记住，就是传入的context类型影响到Glide加载图片的优化程度，Glide可以监视activity的生命周期，在activity销毁的时候自动取消等待中的请求。但是如果你使用Application context，你就失去了这种优化效果。

##### scaleType的重要性

当设置为fitXY时，虽然ImageView显示那么点尺寸，但是，但是Glide加载图片时，却是以全分辨率加载的，于是加载几张，就OOM了。<br />
改成fitCenter或者centerCrop(试了一下fitStart、fitEnd也行，总之看需求了)，就好了，会自动缓存小图，滚动起来也非常流畅。

可以在 Imageview加上这个：

<ImageView android:adjustViewBounds="true"/> 保持宽高比

##### 加载圆形图片

    Glide.with(this)
         .load(url)
         .apply(RequestOptions.circleCropTransform())
         .into(imageView);

##### 带淡入淡出的动画效果

    Glide.with(this)
         .load(url)
         .transition(DrawableTransitionOptions.withCrossFade())
         .into(imageView);

##### 不用 placeholder 占位符，加载列表 item会乱跑
placeholder(R.mipmap.ic_launcher) 的行为是一个APP 去显示一个占位符直到这张图片加载处理完成。

##### 指定加载格式
Glide 特别强大的一点就是可以显示 GIF 图片，但是如果你不想让一张 GIF 图显示成动态图，而是显示成静态的。则可以指定他显示的类型。

    // 在with()方法的后面加入了一个asBitmap()方法，这个方法的意思就是说这里只允许加载静态图片
    // 注意，顺序不能错。with() 完 再 asBitmap() 
    Glide.with(this) 
         .asBitmap() 
         .load("http://guolin.tech/test.gif") 
         .into(imageView);
         
也可以强制显示成其他类型比如：

    asGif()
    asFile() // 文件格式，用于下载
    asDrawable() // Drawable格式，显示图片

##### 简单的缩略图
.thumbnail( 0.1f )
传了一个 0.1f 作为参数，Glide 将会显示原始图像的10%的大小。如果原始图像有 1000x1000 像素，那么缩略图将会有 100x100 像素。
[复杂的缩略图](http://mrfu.me/2016/02/27/Glide_Thumbnails/)
***
###缓存

##### 跳过内存缓存
.skipMemoryCache( true )

这意味着 Glide 将不会把这张图片放到内存缓存中去。这里需要明白的是，这只是会影响内存缓存！Glide 将会仍然利用磁盘缓存来避免重复的网络请求。

##### 跳过磁盘缓存
.diskCacheStrategy( DiskCacheStrategy.NONE )

这段代码片段中将不会被保存在磁盘缓存中。然而，默认的它将仍然使用内存缓存！

方法里面参数意义：
1. **DiskCacheStrategy.NONE**  什么都不缓存
2. **DiskCacheStrategy.DATA** 仅仅只缓存原来的全分辨率的图像。
3. **DiskCacheStrategy.RESOURCE**  仅仅缓存最终的图像，即，降低分辨率后的（或者是转换后的）
4. **DiskCacheStrategy.ALL** 缓存所有版本的图像
5. **DiskCacheStrategy.AUTOMATIC** 让Glide根据图片资源智能地选择使用哪一种缓存策略（**默认选项**）

##### 清除缓存
Glide.get(this).clearDiskCache();不能在UI线程里跑，得另开一个线程。

Glide.get(this).clearMemory();只能在主线程里跑

<hr />

##### 得到 Bitmap

	private SimpleTarget target = new SimpleTarget<Bitmap>() {
	    @Override
	    public void onResourceReady(Bitmap bitmap, Transition<? super Bitmap> transition) {
		  imageView1.setImageBitmap( bitmap );
	    }
	};

	private void loadImageSimpleTarget() {
	    Glide
		.with( context ) // could be an issue!
		.load( eatFoodyImages[0] )
		.into( target );
	}

##### 下载图片

    public static void downloadImg(final Context context, final String url) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    FutureTarget<File> target = Glide.with(context)//
                            .asFile()//
                            .load(url)//
                            .apply(baseOptions)//
                            .submit();
                            
                    // 文件的地址。注意：这个 get() 方法是同步的，所以放在子线程中        
                    File file = target.get();
                    
                } catch (Exception e) {
                }
            }
        }).start();
    }

### Gilde 截取视频某一秒
Gilde 截取视频指定时间的屏幕，这的时间千万千万要注意，单位是 微秒！！！

    /**
     * 显示视频 第三秒 那一帧
     *
     * @param context
     * @param uri
     * @param imageView
     * @param frameTimeMicros 要截取得时间。单位：微秒
     */
    public static void loadVideoScreenshot(final Context context, String uri, ImageView imageView, long frameTimeMicros) {
        // 这里的时间是以微秒为单位
        RequestOptions requestOptions = RequestOptions.frameOf(frameTimeMicros);
        requestOptions.set(FRAME_OPTION, MediaMetadataRetriever.OPTION_CLOSEST);
        requestOptions.transform(new BitmapTransformation() {
            @Override
            protected Bitmap transform(@NonNull BitmapPool pool, @NonNull Bitmap toTransform, int outWidth, int outHeight) {
                return toTransform;
            }

            @Override
            public void updateDiskCacheKey(MessageDigest messageDigest) {
                try {
                    messageDigest.update((context.getPackageName() + "RotateTransform").getBytes("utf-8"));
                } catch (Exception e) {
                    e.printStackTrace();
                }

            }
        });
        Glide.with(context).load(uri).apply(requestOptions).into(imageView);
    }

如果不用 Gilde ，也可以直接截取视频指定时间，代码如下：

    /**
     * 获得视频某一帧的缩略图
     *
     * @param videoPath 视频地址
     * @param timeUs 微秒，注意这里是微秒 1秒 = 1 * 1000 * 1000 微秒
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


<hr />

以上是我自己选出的一些觉得经常用到的，还会更新。

**想要看更详细的 Glide 介绍可以看看这里：**<br />
http://blog.csdn.net/guolin_blog/article/details/78582548


***
##### Android关于图片内存计算

Android中有四种，分别是：<br />
ALPHA_8：每个像素占用1byte内存<br />
ARGB_4444:每个像素占用2byte内存<br />
ARGB_8888:每个像素占用4byte内存<br />
RGB_565:每个像素占用2byte内存<br />
Android默认的颜色模式为ARGB_8888，这个颜色模式色彩最细腻，显示质量最高。但同样的，占用的内存也最大。<br />

举例说明一个32位的PNG也就是ARGB_8888,像素是1204x1024,那么占用空间是:<br />
1024x1024x(32/8) = 4,194,304，也就是4M多。<br />

因为8bit = 1 byte, 32位就是4byte. 我们在解析图片的时候为了方式oom最好使用ARGB_4444模式. 节省一半的内存空间.