---
title: 做一个自己的WiFi密码查看器
excerpt: Android 版
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

WiFi密码查看器 配合 WiFi万能钥匙 太爽了。
有时候用万能钥匙破解了密码之后想要让电脑连上却又不知道密码，或者想告诉别人密码装一把逼的时候，找不到密码，就蛋疼了。
之前在小米市场下载了一个，后来更新了就不能用了，每次去翻 re文件管理 也挺麻烦的。
干脆自己做一个，省的害怕别人的App做什么手脚。
**最终成品：** 当、当、当当~~~~~

![](http://upload-images.jianshu.io/upload_images/1689895-28ba9644515beb5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


目前市面流行的查看WiFi密码的方法基本相同，

就是想办法进入 data/misc/wifi 目录下，找到 wpa_supplicant.conf 文件，然后读取里面的数据。

所以我们的步骤也非常简单

> 1. 一切的前提是应用要获取到root权限；
2. 用命令行的方式，获取 /data/misc/wifi/wpa_supplicant.conf 文件的数据；
3. 解析数据，用列表显示；

这里要先声明一点，Android 上你获取到root权限，只是代表你可以使用 su 了，你依然无法直接读取 /data/misc/wifi/ 这个路径的，你想要做的一切“非法”操作，都必须通过 su 来完成，也就是通过 shell 命令。

关于 shell 的方法，有个工具类，挺好用。

[**ShellUtils.java**](https://github.com/Wing-Li/android-utils/blob/master/app/src/main/java/com/ihongqiqu/util/ShellUtils.java)

<hr />
然后来开始做：
##### 1. 一切的前提是应用要获取到root权限
这个就不多说了，个人的手机如何获取root权限，得靠自己了。

##### 2. 用命令行的方式，获取 /data/misc/wifi/wpa_supplicant.conf 文件的数据；
其实也特别简单，直接上代码，要点在注释中说。

    StringBuffer wifiConf = new StringBuffer();

	Process process = null;
    DataOutputStream dataOutputStream = null;
    DataInputStream dataInputStream = null;
    try {
		// 获取 root 环境
        process = Runtime.getRuntime().exec("su"); 
        dataOutputStream = new DataOutputStream(process.getOutputStream());
        dataInputStream = new DataInputStream(process.getInputStream());
		// cat file 打印文件的内容
		// 获取 /data/misc/wifi/wpa_supplicant.conf 数据，
        dataOutputStream.writeBytes("cat /data/misc/wifi/wpa_supplicant.conf\n"); 
        dataOutputStream.writeBytes("exit\n");
        dataOutputStream.flush();
        InputStreamReader inputStreamReader = new InputStreamReader(dataInputStream, "UTF-8");
        BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
        String line = null;
		// 保存数据，这里都是最基本的 IO 操作，不做过多介绍
        while ((line = bufferedReader.readLine()) != null) {
            wifiConf.append(line);
        }
        bufferedReader.close();
        inputStreamReader.close();
        process.waitFor(); // 线程等待
    } catch (Exception e) {
        return;
    } finally {
        try {
            if (dataOutputStream != null) {
                dataOutputStream.close();
            }
            if (dataInputStream != null) {
                dataInputStream.close();
            }
            process.destroy(); // 线程销毁
        } catch (Exception e) {
        }
    }

这样 /data/misc/wifi/wpa_supplicant.conf  文件的内容就被保存在刚开始定义的 StringBuffer 当中，接下来，我们通过自己喜欢的方式，将数据解析出来即可。
##### 3. 解析数据，用列表显示
都已经拿到数据了，解析完成之后，使用 ListView 进行显示即可。

<hr />
整体的核心就是使用 shell 获取到 /data/misc/wifi/wpa_supplicant.conf 文件的内容，拿到数据之后，一切都变得简单了，都是最基本的操作。

源码在这里： **[WifiPassword](https://github.com/Wing-Li/WifiPassword)** 

里面有 Apk 的下载链接哦~~

wifi密码查看我还是用的比较多的，每次给这个 apk root 权限的时候总害怕他干了什么坏事，现在终于不怕啦。
