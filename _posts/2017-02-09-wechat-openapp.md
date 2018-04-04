---
layout: post
title: 如何从微信浏览器中跳转到APP指定页面？
categories: iOS
description: 如何从微信浏览器中跳转到APP指定页面？
keywords: iOS, 微信
---

产品开发过程中难免会遇到这样的需求：

```

产品说从app内部分享出去的网页上要加一个打开按钮
用户点击打开，能直接唤醒app跳转到这一页去浏览。

明确的需求就是：
微信中打开我们的分享页面，点击商品购买按钮，
用户如果已经安装app，跳转到app做相应的操作。
如果没有安装app，则跳转到应用商店提示用户下载安装。

```

### 方法一：URL Scheme

将参数配置在```url scheme```后面，唤醒app再进行页面跳转逻辑。

但残酷的现实是QQ和微信都把url scheme 唤醒app这种方式给禁了。

###方法二：meta标签

meta标签的格式如下：

```<meta charset="UTF-8" name="apple-itunes-app" content="app-id=1234567890, affiliate-data=myAffiliateData, app-argument=yourScheme://">```

这样添加meta标签后的网页，使用safari打开的时候，就会在顶部显示自己app的导航条。如果没有安装app点击能够跳转到appstore去下载，如果安装了app就能直接通过顶部的meta标签唤醒app了。


上面两种方式都能实现某一方面的需求，但无法完美解决。于是就想到了iOS9之后的Universal Links。

以下内容主要包括：

1.  介绍Universal Links
2.  配置Universal Links
3.  app IDs配置
4.  项目配置
5.  https证书申请


#### 什么是Universal Links呢？

Universal Links就是一个通用链接，iOS9以上的用户，可以通过点击这个链接无缝的重定向到一个app应用，而不需要通过safari打开跳转。
如果用户没有安装这个app，则会在safari中打开这个链接指向的网页。

#### 配置Universal Links

```
1.创建一个名字叫做apple-app-site-association，包含固定格式的json文件

2.将这个文件上传到你的服务器，可以将这个文件放到服务器的根目录下，也可以放到.well-known这个子目录下。

3.配置app，然后在app里面添加代理方法
```

###### 配置流程
1.`apple-app-site-association`文件

```
{
  "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "teamID.bundleId”,
                "paths": ["*"]
            }
        ]
    }
}

```

创建一个包含上述格式的`json`文件，文件名字必须为`apple-app-site-association`，不能带后缀名。

###### appID

appID 的 格式为` teamID.bundleId`形式。


###### 如何获取teamID呢？

登录[开发者网站 ](https://developer.apple.com/cn/)，找到Membership选项卡。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1224641-79139e6d55f15e97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

appID具体显示就是：xxxxxxxxxxx.com.shangxinpifa.app 这种

###### aths

paths配置，实际上就是限制哪些路径可以唤醒app，哪些路径不能唤醒app。格式如下：

```
"paths": [ "/wwdc/news/", "NOT /videos/wwdc/2010/*", "/videos/wwdc/201?/*"]
```

1.使用*配置，则整个网站都可以使用

2.使用特定的URL，例如/wwdc/news/来指定某一个特殊的链接

3.在特定URL后面添加*，例如 /videos/wwdc/2015/*, 来指定网站的某一部分

4.除了使用`*`来匹配任意字符，你也可以使用 ?来匹配单个字符，你可以在路径当中结合这两个字符使用，例如 `/foo/*/bar/201?/mypage`

###### 验证apple-app-site-association文件

文件配置完成之后，将其上传到你的服务器根目录或者.well-known这个子目录下。

1.确保使用`https://域名.com/apple-app-site-association`这个链接可以访问到。

2.也可以使用[苹果的验证网站](https://search.developer.apple.com/appsearch-validation-tool/)，验证文件是否能被苹果请求到。如果是未上线的应用，使用验证网站时可能出现如下提示：


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1224641-7b02fb39ba3a6465.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1224641-87c40709ab0573a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


出现该提示为apple-app-site-association文件配置正确。

出现404错误码提示，则为apple-app-site-association文件未上传成功，或者使用 https://域名.com/apple-app-site-association路径无法访问。

###### app IDs 配置

进入[开发者网站](https://developer.apple.com/)，找到你自己的bundleId，可以点击edit按钮，开启associate domains，并创建相应的provisioning Profiles，如下图：


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1224641-d1fd6e27e90b0931.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 项目配置

在项目的Capablities中开启Associated domains，如下图：


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1224641-313ec84965ff06c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意`domains`可以添加多个，前缀必须为`applinks:`，`applinks:`后为你的服务器的域名。

###### 代码接收Universal Links唤醒


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1224641-464d09f46a711156.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在appdelegate中实现上面这个方法，当使用Universal Links唤醒app时就执行这个方法。

###### 验证配置

快捷验证，在备忘录中输入https://yourdomain.com/goods/129893，长按这个链接，出现下图提示则配置成功。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1224641-d1566cb80057faf6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 证书申请

[阿里云](https://common-buy.aliyun.com/?spm=5176.2020520163.cas.1.ImZFM9&commodityCode=cas#/buy)提供了免费的ssl证书申请


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1224641-e331dd14c1c758ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

购买之后到控制台补全信息：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1224641-f688975523ed9a18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

补全信息后点击进度查看下一步的配置工作

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1224641-87987f8705ffb90b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按照步骤完成配置后，10分钟左右就会通过审核，服务器配置按照阿里云提供的文档继续操作即可



## 问题补充：

#### 1.在没有安装app的情况下，如何跳转下载页面？

对于apple手机，在nginx上，将你的universal link指向一个文件，例如download.html

```
<!DOCTYPE html>
<html lang="en" >
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate" />
    <meta http-equiv="Pragma" content="no-cache" />
    <meta http-equiv="Expires" content="0" />
    <meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no, width=device-width">
    <title>上新-跳转中...</title>

</head>
<body>
<script type="text/javascript">
        setTimeout(function(){

            window.location = "http://a.app.qq.com/o/simple.jsp?pkgname=com.shangxin"
        },3000)

</script>

</body>
</html>
```

这样做的目的可以理解为，给手机3秒的时间去唤醒app，如果3秒内没有唤醒，则跳转到下载页面


#### 2.安卓上有没有这种功能？

我目前没找到很好的deep link方案，安卓上还是采用传统的提示打开浏览器，使用scheme跳转。

#### 3.安卓上怎么处理用户没有下载app的情况？

同样的思路，如果浏览器在一定时间段内没有唤起app，则直接跳转下载页面，可参考以下代码

```

        if(!isWeixin){
           	if(browser.versions.android == true){

           		var ifr = document.createElement('iframe');
           		ifr.src = 'your android awake up scheme'
           		ifr.style.display = 'none';
           		document.body.appendChild(ifr);
           		var aa = Date.now();
           		setTimeout(function(){
           			var cc = Date.now();
           			if(cc - aa <(1500 + 200)){
           				window.location = "http://a.app.qq.com/o/simple.jsp?pkgname=com.shangxin"
           			}else{
           				document.body.removeChild(ifr);
           			}
           		},1500)
           	}
        }
```

#### 4.唤醒app后，屏幕右上角的返回按钮如何去除？
不知道。。。- - !

