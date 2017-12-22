---
title: Android跳转到应用商店的APP详情页面，以及 Google GMS 各个apk的包
excerpt: 评价
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

#### 需求：
从App内部点击按钮或链接，跳转到应用商店的某个APP的详情页面。
让用户 下载 或 评论。

#### 实现：

    /**
     * 启动到应用商店app详情界面
     *
     * @param appPkg    目标App的包名
     * @param marketPkg 应用商店包名 ,如果为""则由系统弹出应用商店列表供用户选择,否则调转到目标市场的应用详情界面，某些应用商店可能会失败
     */
    public void launchAppDetail(String appPkg, String marketPkg) {
        try {
            if (TextUtils.isEmpty(appPkg)) return;

            Uri uri = Uri.parse("market://details?id=" + appPkg);
            Intent intent = new Intent(Intent.ACTION_VIEW, uri);
            if (!TextUtils.isEmpty(marketPkg)) {
                intent.setPackage(marketPkg);
            }
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            startActivity(intent);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

**注意：**如果 应用商店包名为空 就会将手机上已下载的应用商店都列出来，让你选择一个进行跳转。

#### 主流应用商店对应的包名如下：

|包名|商店|
|:-----|:-----|
|com.android.vending    |    Google Play|
|com.tencent.android.qqdownloader|    应用宝|
|com.qihoo.appstore    |360手机助手|
|com.baidu.appsearch    |    百度手机助|
|com.xiaomi.market|    小米应用商店|
|com.wandoujia.phoenix2 |    豌豆荚|
|com.huawei.appmarket|    华为应用市场|
|com.taobao.appcenter|    淘宝手机助手|
|com.hiapk.marketpho    |    安卓市场|   
|cn.goapk.market|        安智市场| 

#### 列出 Google GMS 各个apk的包名和类名，记录一下：
此处转自：
http://blog.csdn.net/zheng_buding/article/details/42149379

    Facebook    [com.facebook.katana  /  com.facebook.katana.LoginActivity]

    Chrome      [com.Android.chrome  /  com.google.android.apps.chrome.Main]

    Gmail       [com.google.android.gm  /  com.google.android.gm.ConversationListActivityGmail]

    Google+     [com.google.android.apps.plus  /  com.google.android.apps.plus.phone.HomeActivity]

    Maps        [com.google.android.apps.maps  /  com.google.android.maps.MapsActivity]

    Play Movies [com.google.android.videos  /  com.google.android.youtube.videos.EntryPoint]

    Play Books  [com.google.android.apps.books  /  com.google.android.apps.books.app.BooksActivity]

    Play Games  [com.google.android.play.games  /  com.google.android.gms.games.ui.destination.main.MainActivity]

    Drive       [com.google.android.apps.docs  /  com.google.android.apps.docs.app.NewMainProxyActivity]

    YouTube     [com.google.android.youtube  /  com.google.android.apps.youtube.app.WatchWhileActivity]

    Photos      [com.google.android.apps.plus  /  com.google.android.apps.plus.phone.ConversationListActivity]

    Hangouts    [com.google.android.talk  /  com.google.android.talk.SigningInActivity]

    Play Store  [com.android.vending  /  com.android.vending.AssetBrowserActivity]

    Opera Mini  [com.opera.mini.android  /  com.opera.mini.android.Browser]

    Deezer      [deezer.android.app  /  com.deezer.android.ui.activity.LauncherActivity]

#### 链接到 Google Play 的一些额外方法
https://developer.android.com/distribute/tools/promote/linking.html

结果    |网页链接|    Android 应用链接(Google Play)
------------- |-------------| -----
显示特定应用的商品详情页    |http://play.google.com/store/apps/details?id=<package_name>    |market://details?id=<package_name>
显示特定发布者的应用    |http://play.google.com/store/search?q=pub:<publisher_name>    |market://search?q=pub:<publisher_name>
使用常规字符串查询搜索应用    |http://play.google.com/store/search?q=<query>    |market://search?q=<query>