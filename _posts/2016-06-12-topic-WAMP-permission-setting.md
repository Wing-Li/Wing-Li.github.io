---
title: WAMP权限设置
excerpt: Apache：You don't have permission to access / on this server.
categories:
  - topics
tag: PHP  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/users/320f9e8f7fc9/latest_articles
感谢您的关注。

<br />
WAMP官网：
[http://www.wampserver.com/#download-wrapper](http://www.wampserver.com/#download-wrapper)

写在最前面：
在网上写这个教程的一大片大片的，之所以还要写的原因是，我按照网上说的配置了，可是还是没有成功，我也是纳闷了很久，最后发现少了最后一步，也就是本篇的第三步。所以在此留以记录、分享。

**总步骤：**
> 1. 修改 D:\wamp\bin\apache\apache2.4.9\conf \ 路径下的 httpd.conf 文件，添加权限；
2. 修改 D:\wamp\bin\apache\apache2.4.9\conf \ 路径下的 httpd.conf 文件，加载虚拟 hosts
3. 修改 D:\wamp\bin\apache\apache2.4.9\conf\extra\ 路径下的 httpd-vhosts.conf ，添加工作目录


<br />
<hr />

#### 1. 修改 D:\wamp\bin\apache\apache2.4.9\conf \ 路径下的 httpd.conf 文件，添加权限；
需要修改的地方，用注释标出，

	#
	# DocumentRoot: The directory out of which you will serve your
	# documents. By default, all requests are taken from this directory, but
	# symbolic links and aliases may be used to point to other locations.
	#
	#======================================
	#这里是我们的工作目录，默认是“/www”，可以自定义修改
	#======================================
	DocumentRoot "E:\PHPProject"

	#
	# Each directory to which Apache has access can be configured with respect
	# to which services and features are allowed and/or disabled in that
	# directory (and its subdirectories). 
	#
	# First, we configure the "default" to be a very restrictive set of 
	# features.  
	#
	<Directory />
		AllowOverride All
		Require all denied
	</Directory>

	#======================================
	#在上面修改完了之后，在这里也要修改（可以不做修改，也就是使用默认的）
	#======================================
	<Directory "E:\PHPProject">
		#
		# Possible values for the Options directive are "None", "All",
		# or any combination of:
		#   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
		#
		# Note that "MultiViews" must be named *explicitly* --- "Options All"
		# doesn't give it to you.
		#
		# The Options directive is both complicated and important.  Please see
		# http://httpd.apache.org/docs/2.4/mod/core.html#options
		# for more information.
		#
		Options Indexes FollowSymLinks

		#
		# AllowOverride controls what directives may be placed in .htaccess files.
		# It can be "All", "None", or any combination of the keywords:
		#   AllowOverride FileInfo AuthConfig Limit
		#
		AllowOverride all

		#
		# Controls who can get stuff from this server.
		#

		# onlineoffline tag - don't remove
		Require all granted
		Order Deny,Allow
		Deny from all
		Allow from all
		#======================================
		#修改这几个权限
		#======================================		
	</Directory>

#### 2. 修改 D:\wamp\bin\apache\apache2.4.9\conf \ 路径下的 httpd.conf 文件，加载虚拟 hosts
还是在刚刚的文件里， Ctrl + F 搜索 “httpd-vhosts”；
把 #Include conf/extra/httpd-vhosts.conf 的注释放开，也就是删除前面的 #

	# Virtual hosts
	Include conf/extra/httpd-vhosts.conf

### 3. 修改 D:\wamp\bin\apache\apache2.4.9\conf\extra\ 路径下的 httpd-vhosts.conf ，添加工作目录
在 httpd-vhosts.conf 文件的最后添加以下设置。

	<VirtualHost _default_:80>
	#DocumentRoot "${SRVROOT}/htdocs"
	DocumentRoot "E:\PHPProject"
	#ServerName www.example.com:80
	</VirtualHost>


**然后重启Apache就可以了。**
