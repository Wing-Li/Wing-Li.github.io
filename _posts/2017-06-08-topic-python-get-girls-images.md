---
title: 爬虫入门修行
excerpt: 爬取美女图片，分目录多级存储
categories:
  - topics
tag: Python
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/u/320f9e8f7fc9
感谢您的关注。


最近有个需求：下载某个网站的图片。
所以简单研究了一下爬虫。
在此整理一下结果，一为自己记录，二给后人一些方向。
整体研究周期 2-3 天，看完之后，在加上看的时候或多或少也会自己搜到一些其他知识。
顺着看下来，应该会对爬虫技术有一个初步的认识。

> 利用 Python 的 Scrapy 框架。

<hr />

### 1. 首先，安装爬虫需要的环境
Python爬虫系列之----Scrapy下载安装
http://blog.csdn.net/u011781521/article/details/70177291

这里面非常详细的介绍了每一步的下载，按照步骤下载安装，完全没有问题。

### 2. 一个非常简单的下载图片的例子
结合 [Scarpy 框架的中文官网](http://scrapy-chs.readthedocs.io/zh_CN/0.24/intro/tutorial.html)
来看这个例子：下载图片
http://www.jianshu.com/p/b5ae15cb131d

这个项目跑通之后，也就大概能了解 Scarpy 框架。

但是这个项目还不够完善，因为所爬取的图片全部都在一个文件夹下，并且图片的名字也被修改了。

### 3. 多页爬取，并且分目录保存到本地
首先看这篇文章，讲解了如何多级爬取。
https://my.oschina.net/lpe234/blog/342741

不太明白的话，可以多看几遍。如果，还是实在不明白，也没关系，直接看接下来的例子，可能更直观。

##### 例子一：
scrapy中使用ImagePipeline自定义图片文件名
http://mazih.com/post/cibuwb7bu0000rfa2m169vzv9/

这个例子介绍了在往本地下载的时候，如何多级存储。

##### 例子二：
爬取妹子图
http://codingpy.com/article/scrapy-01-meizitu/
跟上面差不多，但也能加深理解，加强印象。


### 4. 反爬虫
Scrapy爬取美女图片第四集 突破反爬虫(上)
http://www.jianshu.com/p/9159111bcd87
Python爬虫系列之----Scrapy(七)使用IP代理池
http://blog.csdn.net/u011781521/article/details/70194744?locationNum=4&fps=1
scrapy下载图片遇到反盗链的设置，解决方法两种
https://zhuanlan.zhihu.com/p/26251969


最后再分享一个系列，非常棒。解决了不少问题。
http://blog.csdn.net/u011781521/article/category/6560282

<hr />
以上就是学习的时候，看到的一些资料。
然后贴出一篇我自己写的，爬取的时候分了三级目录，并且，最后一级还有 下一页。


    import scrapy
    from znns.items import ZnnsItem


    class NvshenSpider(scrapy.Spider):
        name = 'znns'
        allowed_domains = ['']
        start_urls = ['https://www.nvshens.com/rank/sum/']
        headers = {
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
            'Accept-Language': 'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
        }

        # 排行榜循环
        def parse(self, response):
            exp = u'//div[@class="pagesYY"]//a[text()="下一页"]/@href'  # 下一页的地址
            _next = response.xpath(exp).extract_first()
            yield scrapy.Request(response.urljoin(_next), callback=self.parse, dont_filter=True)

            for p in response.xpath('//li[@class="rankli"]//div[@class="rankli_imgdiv"]//a/@href').extract():  # 某一个妹子简介详情页
                item_page = "https://www.nvshens.com/" + p + "album/"  # 拼接 全部相册页面
                yield scrapy.Request(item_page, callback=self.parse_item, dont_filter=True)

        # 单个介绍详情页
        def parse_item(self, response):
            item = ZnnsItem()
            # 某个人的名字，也就是一级文件夹
            item['name'] = response.xpath('//div[@id="post"]//div[@id="map"]//div[@class="browse"]/a[2]/@title').extract()[
                0].strip()

            exp = '//li[@class="igalleryli"]//div[@class="igalleryli_div"]//a/@href'
            for p in response.xpath(exp).extract():  # 遍历妹子全部相册
                item_page = "https://www.nvshens.com/" + p  # 拼接图片的详情页
                yield scrapy.Request(item_page, meta={'item': item}, callback=self.parse_item_details, dont_filter=True)

        # 图片主页，开始抓取
        def parse_item_details(self, response):
            item = response.meta['item']
            item['image_urls'] = response.xpath('//ul[@id="hgallery"]//img/@src').extract()  # 图片链接
            item['albumname'] = response.xpath('//h1[@id="htilte"]/text()').extract()[0].strip()  # 二级文件夹
            yield item

            new_url = response.xpath('//div[@id="pages"]//a[text()="下一页"]/@href').extract_first()  # 翻页
            new_url = "https://www.nvshens.com/" + new_url
            if new_url:
                yield scrapy.Request(new_url, meta={'item': item}, callback=self.parse_item_details, dont_filter=True)


爬取结果如图：

![](http://upload-images.jianshu.io/upload_images/1689895-bcb5d654e7fadc0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<hr />

最后放上我写的项目地址：
https://github.com/Wing-Li/znns

欢迎 Star，欢迎关注。

<br />
收集：
[收集各种爬虫 （默认爬虫语言为 python）](https://github.com/facert/awesome-spider?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
[用 PHP 爬虫做旅游数据分析]()