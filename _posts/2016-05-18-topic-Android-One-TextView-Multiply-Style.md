---
title: 一个TextView设置多种格式
excerpt: 类似于“评论”的样式
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。


先看一张效果图。 <br />
主要功能： <br />
1.只有一个 TextView，但是显示多个样式，并且，前几个字是可以点击。 <br />
2.修改 Toast 的弹出位置。

![](http://upload-images.jianshu.io/upload_images/1689895-910a15e0fc1cf581.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这类效果，使用场景最多的，应该就是评论了吧。

#### 主角介绍 - SpannableString 
主要用的到就是 SpannableString 这个类，其实还有个SpannableStringBuilder，他们两个作用跟String其实是很像的。不同之处就是他们俩可以给字符串设置各种样式。

SpannableString 和 SpannableStringBuilder 的区别也就是字面意思，多了一个Builder。类似于 String 和 StringBuilder。SpannableStringBuilder 是可以通过append()方法进行拼接。而SpannableString 通过构造器创建了之后就固定。

#### 使用方法

    String name = "小可爱：";
    String msg = name + "小今天心情不错，买张彩票。";

    SpannableString ss = new SpannableString(msg);
    ss.setSpan(new ForegroundColorSpan(Color.RED), name.length(), msg.length() - 1, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
    mTxt.setText(ss);

因为SpannableString 实现了CharSequence的接口，所以这一步之后，我们定义的 ss，就可以作为字符串设置给TextView了。 <br />
显示效果是：除了“小可爱:"，后面的都是红色字。 <br />
当然，我们是可以同时给一个字符串设置多个样式的。 <br />

很明显我们可以看到通过setSpan()这个方法，我们就将指定的字符更换了样式。 <br />
我们具体来看看 setSpan(); 到底有多么的神通广大。

    /**
     * 设置字符串的样式
     * @param what 对应的样式的类（往下看）
     * @param start 样式开始的位置
     * @param end 样式结束的位置
     * @param flags 包含的范围（往下看）
     */
    public void setSpan (Object what, int start, int end, int flags)

起始结束位置就不用说了，就是符合行业的标准模式，含头不含尾。 <br />
这里需要介绍是第一个和第四个参数。 <br />
我们先来看第四个：
**int flags**可设置为：

    Spannable.SPAN_EXCLUSIVE_EXCLUSIVE：前后都不包括，即在指定范围的前面和后面插入新字符都不会应用新样式
    Spannable.SPAN_EXCLUSIVE_INCLUSIVE：前面不包括，后面包括。即仅在范围字符的后面插入新字符时会应用新样式
    Spannable.SPAN_INCLUSIVE_EXCLUSIVE：前面包括，后面不包括。
    Spannable.SPAN_INCLUSIVE_INCLUSIVE：前后都包括。

这里系统给定了四个参数，含义已经注明。 <br />
需要说明的是，SpannableString 和 SpannableStringBuilder 的区别 在开头提到了，不知道大家还记得吗？ <br />
只有 SpannableStringBuilder 是可以通过append()方法，往后面拼接字符串的，而这几个标示都是当字符串改变之后的效果，所以后面三个样式的效果，可能只有使用了SpannableStringBuilder 添加文字的时候才看得到效果。 <br />
使用SpannableString ，添加了其他的flags也是没有什么用的，因为，只要一替换文字，整个对象全换了。 <br />
所以这个参数，我们看着用 就好。

**Object what** 这个参数，那可是相当强大。先看功能

        1、BackgroundColorSpan 背景色
        2、ClickableSpan 文本可点击，有点击事件
        3、ForegroundColorSpan 文本颜色（前景色）
        4、MaskFilterSpan 修饰效果，如模糊(BlurMaskFilter)、浮雕(EmbossMaskFilter)
        5、MetricAffectingSpan 父类，一般不用
        6、RasterizerSpan 光栅效果
        7、StrikethroughSpan 删除线（中划线）
        8、SuggestionSpan 相当于占位符
        9、UnderlineSpan 下划线
        10、AbsoluteSizeSpan 绝对大小（文本字体）
        11、DynamicDrawableSpan 设置图片，基于文本基线或底部对齐。
        12、ImageSpan 图片
        13、RelativeSizeSpan 相对大小（文本字体）
        14、ReplacementSpan 父类，一般不用
        15、ScaleXSpan 基于x轴缩放
        16、StyleSpan 字体样式：粗体、斜体等
        17、SubscriptSpan 下标（数学公式会用到）
        18、SuperscriptSpan 上标（数学公式会用到）
        19、TextAppearanceSpan 文本外貌（包括字体、大小、样式和颜色）
        20、TypefaceSpan 文本字体
        21、URLSpan 文本超链接

例如刚开始，我们用到的：设置文本的颜色

	SpannableString ss = new SpannableString(msg); 
	ss.setSpan(new ForegroundColorSpan(Color.RED), name.length(), msg.length() - 1, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);

上面的功能相当的多，不一一介绍，用的时候再看，因为用法都差不多。

***
#### 效果图代码实现
现在放上刚开始的时候，实现的那张图代码。因为代码也相当简单，所以直接贴上。

	public class MainActivity extends AppCompatActivity {

	    private TextView mTxt;

	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);

			mTxt = (TextView) findViewById(R.id.txt);
			initView();
	    }

	    private void initView() {
			String name = "小可爱：";//模拟名称
			String msg = name + "今天心情不错，买张彩票。";//模拟说说

			SpannableString ss = new SpannableString(msg);
			//名称的点击事件
			ss.setSpan(clickableSpan, 0, name.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
			//说说的字体样式
			ss.setSpan(new ForegroundColorSpan(Color.RED), name.length(), msg.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

			mTxt.setText(ss);
			mTxt.setMovementMethod(LinkMovementMethod.getInstance());//设置超链接为可点击状态
	    }

	    /**
	     * 名称的点击事件
	     */
	    private ClickableSpan clickableSpan = new ClickableSpan() {
			@Override
			public void updateDrawState(TextPaint ds) {
			    super.updateDrawState(ds);
			    ds.setUnderlineText(false); //去掉下划线
			    ds.setColor(Color.BLUE);//设置点击前的颜色
			}

			@Override
			public void onClick(View widget) {
			    toast(((TextView) widget).getText());
			}
	    };

	    private void toast(CharSequence str) {
			//如果这里加了 String.valueOf(str)，弹出的提示，就会没有样式
			Toast toast = Toast.makeText(MyApplication.getContext(), str, Toast.LENGTH_SHORT);
			toast.setGravity(Gravity.TOP, 0, 200);//改变Toast弹出的位置
			toast.show();
	    }
	}

在最后顺便用到了修改Toast提示的位置。<br />
坐标可以根据屏幕的分辨率的尺寸的百分比来进行计算，会更合理。

好了，本次分享就到这里。<br />
项目地址：<br />
https://github.com/Wing-Li/PracticeDemos/tree/master/Hack10_CustomizeText

欢迎指正。