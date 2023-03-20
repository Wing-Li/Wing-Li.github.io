---
title: Mac配置环境变量之后，zsh: command not found: 解决方法
excerpt: ,
categories:
  - topics
tag: 扩展了解  
---


Mac 添加了环境变量之后， 执行 source ~/.bash_profile 之后，依然没有效果。

##### 解决：

把 bash shell 中.bash_profile 全部环境变量加入zsh shell里就好

#### 第一步

在终端执行

> open ~/.zshrc

#### 第二步

在最后一行加入

> source ~/.bash_profile

#### 第三步

在终端里执行

> source ~/.zshrc