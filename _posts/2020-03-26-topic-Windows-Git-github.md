---
title: Windows 配置 git 与 github 关联
excerpt: 留存待看
categories:
  - topics
tag: 扩展了解  
---


换了电脑之后要配置 git 再次记录配置方式，并且记录之后的常用 git 命令，不过我大多数都是用的 idea 用命令的时间也不多。


##### 1. 配置 git 账户名和邮箱
github 的邮箱和名称

    ~$ git config --global user.email "you@example.com"
    ~$ git config --global uesr.name "you Name"

##### 2. 配置 ssh
配置ssh 可以免去每次 需要输入密码的麻烦。
生成秘钥对（如果使用 idea 的话，中间不要设置密码，直接三次回车就好。）：

    ~$ ssh-keygen -t rsa -C "you@example.com"

    Generating public/private rsa key pair.

    Enter file in which to save the key (C:\Users\用户名/.ssh/id_rsa):

    Created directory 'C:\Users\用户名/.ssh'.

    Enter passphrase (empty for no passphrase):

    Enter same passphrase again:

**公钥生成的位置在 用户名/.ssh 文件夹**

##### 3. 将公钥配置到 github 上

登录到github上 ，点击头像，然后 Settings -> 左栏点击 SSH and GPG keys -> 点击 New SSH key 。
在title中输入一个名字，在key 文本域中输入你的公钥， 点击 Add SSH Key

##### 4. 验证 key是否正常工作
不用替换邮箱

    ~$ ssh -T git@github.com