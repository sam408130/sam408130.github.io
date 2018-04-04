---
layout: post
title: Python爬虫系列（二）：Charles和Postman
categories: Python
description: Charles和Postman
keywords: Python, Scrapy
---

# 1.Charles

![image.png](http://upload-images.jianshu.io/upload_images/1224641-cb91c0644ef98cca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以下内容引自：[Charles 从入门到精通](http://blog.devtang.com/2015/11/14/charles-introduction/)

[Charles](http://www.charlesproxy.com/) 是在 Mac 下常用的网络封包截取工具，在做移动开发时，我们为了调试与服务器端的网络通讯协议，常常需要截取网络封包来分析。
Charles 通过将自己设置成系统的网络访问代理服务器，使得所有的网络访问请求都通过它来完成，从而实现了网络封包的截取和分析。
除了在做移动开发中调试端口外，Charles 也可以用于分析第三方应用的通讯协议。配合 Charles 的 SSL 功能，Charles 还可以分析 Https 协议。

Charles 主要的功能包括：
* 截取 Http 和 Https 网络封包。
* 支持重发网络请求，方便后端调试。
* 支持修改网络请求参数。
* 支持网络请求的截获并动态修改。
* 支持模拟慢速网络。

##  安装 Charles

***

去 Charles 的官方网站（[http://www.charlesproxy.com](http://www.charlesproxy.com/)）下载最新版的 Charles 安装包，是一个 dmg 后缀的文件。打开后将 Charles 拖到 Application 目录下即完成安装。安装后可免费试用30天，如果你想试用破解版，可以查阅一些破解文章。

## 将 Charles 设置成系统代理

***

之前提到，Charles 是通过将自己设置成代理服务器来完成封包截取的，所以使用 Charles 的第一步是将其设置成系统的代理服务器。
启动 Charles 后，第一次 Charles 会请求你给它设置系统代理的权限。你可以输入登录密码授予 Charles 该权限。你也可以忽略该请求，然后在需要将 Charles 设置成系统代理时，选择菜单中的 “Proxy” -> “Mac OS X Proxy” 来将 Charles 设置成系统代理。如下所示：

![image.png](http://upload-images.jianshu.io/upload_images/1224641-05bf3a00b441cbb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

之后，你就可以看到源源不断的网络请求出现在 Charles 的界面中。
需要注意的是，请先关闭你的代理工具（如shadowsocks），再启动Charles

## Charles 主界面介绍
***

![image.png](http://upload-images.jianshu.io/upload_images/1224641-84f101160115ad6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 截取 iPhone 上的网络封包
***

Charles 通常用来截取本地上的网络封包，但是当我们需要时，我们也可以用来截取其它设备上的网络请求。下面我就以 iPhone 为例，讲解如何进行相应操作。

在 iPhone 的 “ 设置 “->” 无线局域网 “ 中，可以看到当前连接的 wifi 名，通过点击右边的详情键，可以看到当前连接上的 wifi 的详细信息，包括 IP 地址，子网掩码等信息。在其最底部有「HTTP 代理」一项，我们将其切换成手动，然后填上 Charles 运行所在的电脑的 IP，以及端口号 8888，如下图所示：

![image.png](http://upload-images.jianshu.io/upload_images/1224641-fe31fabb2470a295.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

请确保mac和iphone在同一个局域网内。

设置好之后，我们打开 iPhone 上的任意需要网络通讯的程序，就可以看到 Charles 弹出 iPhone 请求连接的确认菜单（如下图所示），点击 “Allow” 即可完成设置。

![image.png](http://upload-images.jianshu.io/upload_images/1224641-0ff3f991e6ac6d81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 截取 Https 通讯信息
***

手机端操作：
下载Charles证书[http://www.charlesproxy.com/ssl.zip](http://www.charlesproxy.com/ssl.zip)，解压后导入到iOS设备中（将crt文件作为邮件附件发给自己，再在iOS设备中点击附件即可安装；也可上传至百度之类的网盘，通过safari下载安装）

电脑端操作：
1、在Charles的工具栏上点击设置按钮，选择Proxy Settings…
切换到SSL选项卡，选中Enable SSL Proxying。（别急，选完先别关掉）
2、SSL选项卡的Locations里填写要抓包的域名和端口，点击Add按钮，在弹出的表单中Host填写域名。比如填api.instagram.com，Port填443，可以用*.*.*来代替所有域名，如下图：


![image.png](http://upload-images.jianshu.io/upload_images/1224641-9d5dd9929edb307d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

更多内容请查看原文[Charles 从入门到精通](http://blog.devtang.com/2015/11/14/charles-introduction/)


# 2.Postman
我们使用Charles了解了待爬取网站或app的通讯协议后，使用Postman便可以轻松模拟发送一些请求，[Postman下载地址](https://www.getpostman.com/apps)

## 一、基本功能
***

Postman的功能在[文档](https://www.getpostman.com/docs)中有介绍。这里简单介绍一下主界面，入门功能就都提到了。

![image.png](http://upload-images.jianshu.io/upload_images/1224641-afe01c62877e807b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. Collections：在Postman中，Collection类似文件夹，可以把同一个项目的请求放在一个Collection里方便管理和分享，Collection里面也可以再建文件夹。如果做API文档的话，可以每个API对应一条请求，如果要把各种输入都测到的话，就需要每条测试一条请求了。这里我新建了一个example用于介绍整个流程，五个API对应五条请求。这个Collection可以通过https://www.getpostman.com/collections/96b64a7c604072e1e4ee导入你自己的Postman中。

2. 上面的黑字注册是请求的名字，如果有Request description的话会显示在这下面。下面的蓝字是保存起来的请求结果，点击可以载入某次请求的参数和返回值。我会用这个功能给做客户端的同事展示不同情况下的各种返回值。保存请求的按钮在15.

3. 选择HTTP Method的地方，各种常见的不常见的非常全。

4. 请求URL，两层大括号表示这是一个环境变量，可以在16的位置选择当前的environment，环境变量就会被替换成该environment里variable的值。

5. 点击可以设置URL参数的key和value

6. 点击发送请求

7. 点击保存请求到Collection，如果要另存为的话，可以点击右边的下箭头

8. 设置鉴权参数，可以用OAuth之类的

9. 自定义HTTP Header，有些因为Chrome愿意不能自定义的需要另外装一个插件Interceptor，在16上面一行的卫星那里

10. 设置Request body，13那里显示的就是body的内容

11. 在发起请求之前执行的脚本，例如request body里的那两个random变量，就是每次请求之前临时生成的。

12. 在收到response之后执行的测试，测试的结果会显示在17的位置

13. 有四种形式可以选择，form-data主要用于上传文件。x-www-form-urlencoded是表单常用的格式。raw可以用来上传JSON数据

14. 返回数据的格式，Pretty可以看到格式化后的JSON，Raw就是未经处理的数据，Preview可以预览HTML页面

15. 点击这里把请求保存到2的位置

16. 设置environment variables和global variables，点击右边的x可以快速查看当前的变量。

17. 测试执行的结果，一共几个测试，通过几个。
