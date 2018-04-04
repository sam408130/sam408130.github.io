---
layout: post
title: Python爬虫系列（五）：京东App(如何分析https下的api结构，及数据爬取)
categories: Python
description: 京东App(如何分析https下的api结构，及数据爬取)
keywords: Python, Scrapy
---

## Charles证书安装

如果不进行下面的设置，https的reqeust和response都是乱码，设置完之后https就可以抓包了。

手机端操作：
下载Charles证书[http://www.charlesproxy.com/ssl.zip](http://www.charlesproxy.com/ssl.zip)，解压后导入到iOS设备中（将crt文件作为邮件附件发给自己，再在iOS设备中点击附件即可安装；也可上传至百度之类的网盘，通过safari下载安装）

电脑端操作：
1、在Charles的工具栏上点击设置按钮，选择SSL Proxy Settings…

![屏幕快照 2017-07-20 上午7.25.16.png](http://upload-images.jianshu.io/upload_images/1224641-6e8a647efedd4649.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

切换到SSL选项卡，选中Enable SSL Proxying。（别急，选完先别关掉）
2、SSL选项卡的Locations里填写要抓包的域名和端口，点击Add按钮，在弹出的表单中Host填写域名。比如填api.instagram.com，Port填443

## 抓包

手机配置好http代理后，便可以开始抓包了
例如京东『分类』页面的结构

![image.png](http://upload-images.jianshu.io/upload_images/1224641-0782b05adf5c5849.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 通过抓包数据可以查看api规则

![image.png](http://upload-images.jianshu.io/upload_images/1224641-66545600c7e07956.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过接口https://api.m.jd.com/client.action?function=entraceCatalog 获取所有的一级分类，观察左侧的url可以很清楚看到jd app启动时请求了哪些接口作为初始化配置。

一级分类下的内容通过cid获得：

![image.png](http://upload-images.jianshu.io/upload_images/1224641-72607f4a9bc1dc23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在模拟请求爬取数据时，注意要根据示例接口，补全请求所需的参数信息

不难发现jd的api里面都有一个sign参数，这个是通过时间戳（st参数）和uuid通过一定规则md5出来的，用于数据安全。因此想要爬取jd的数据，需要知道这个加密规则（某商城里有出售的，需要的同学可以去购买）。