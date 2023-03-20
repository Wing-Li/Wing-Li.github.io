---
title: linux字符串的截取
excerpt: shell脚本里面可用
categories:
  - topics
tag: 扩展了解 
---


我是这么用的：

    sh 'curl ${CFG_LAUNCH_URL%%|*} -o Nodetech_sys_app/app/src/main/res/drawable-xxhdpi/launch_screen.png'
    sh 'curl ${CFG_LAUNCH_URL#*|} -o Nodetech_sys_app/app/src/main/res/drawable-xxhdpi-2160x1080/launch_screen.png'

如下为字符串截取方式，在此记录。

***

### 第一种使用cut
一般常用的命令

    echo "hello world" |cut -d" " -f1
    #hello
    echo "hello world" |cut -d" " -f2
    #world
    echo "hello world" |cut -d" " -f1-
    #hello world
    
    echo "hello world" |cut -c 1
    #h
    echo "hello world" |cut -c 1-4
    #hell
    echo "hello world" |cut -c 1,2,5
    #heo
    
    echo "测试字符串" |cut -bn 1
    #测
    echo "测试字符串" |cut -bn 1-
    #测试字符串

-d 表示分割的字符，这里只能使用单字符，比较有局限性。与-f 配合使用

-c 表示以字符分割，1表示获取第一个字符，1-4,表示获取1到4的字符，1-表示获取第一个字符以及后边所有的字符

-b 表示以字节分割，和-c的用法相同，但是是以一字节（8bit）分割，如果是中文，则可能出现乱码。一般与n配置使用，表示不允许将多字节的字符按字节分开

-f 表示输出的字段（域）（从1开始计数），1表示输出分割后的第一个字段 1-表示输出字段1以及后边的所有字段


***

### 第二种办法 shell变量中切割字符串

1.获取字符串的长度

    string="hello world"
    ${#string} # 11

2.删除从左边开始到指定字符第一次出现的字符并保留右边字符

    string="http://www.test.com"
    echo ${string#*//} # www.test.com

3.删除从左边开始到指定字符最后一次出现的字符并保留右边字符

    string="http://www.test.com/index.html"
    echo ${string##*/} # index.html

4.删除从右边开始到指定字符第一次出现的字符并保留左边字符

    string="http://www.test.com/index.html"
    echo ${string%/*} # http://www.test.com

5.删除从右边开始到指定字符最后一次出行字符并保留左边字符

    string="http://www.test.com"
    echo ${string%%/*} # http:

6.从左开始到指定字节数获取字符串中的子串（按照字节截取）格式（${string:起始字节：字节数}）

    string="hello world"
    echo ${string:0:7} # hello w

7.从左开始到指定字节数获取字符串中的子串（从起始字节开始到结束）格式（${string:起始字节}）

    string="hello world"
    echo ${string:7} # orld

8.从右开始到指定字节数获取字符串中的子串（按照字节获取）格式 （${string:0-起始：字节数}）

    string="hello world"
    echo ${string:0-5:3} # wor

9.从右开始到指定字节数获取字符串中的子串 (从起始字节开始到字符串末尾) ${string:0-5}

    string="hello world"
    echo ${string:0-5} # world
