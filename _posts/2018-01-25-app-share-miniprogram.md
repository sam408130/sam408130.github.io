---
layout: post
title: App分享微信小程序功能介绍和业务方案分析
categories: iOS
description: 今天微信小程序新增了支持跳转App的功能，算是一次比较大的突破，我也第一时间体验了一下该功能，App和小程序之间的跳转还是比较灵活的。
keywords: iOS, 小程序
---

今天微信小程序新增了支持跳转App的功能，算是一次比较大的突破，我也第一时间体验了一下该功能，App和小程序之间的跳转还是比较灵活的。

![屏幕快照 2018-01-25 下午7.05.46.png](http://upload-images.jianshu.io/upload_images/1224641-36d22d5d96b82d17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[小程序支持打开移动应用](https://mp.weixin.qq.com/s/DkbDa2506-dJQ8HY6JFfBg)

![逻辑图](http://upload-images.jianshu.io/upload_images/1224641-4f9664b5fe9e1cc9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为需要用户主动触发才能打开 APP，所以该功能不由 API 来调用，需要用 `open-type` 的值设置为 `launchApp` 的 `<button>` 组件的点击来触发。

当小程序从 APP 分享消息卡片的场景打开时（场景值 1036，APP 分享小程序文档 iOS [参见](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419317332&token=&lang=zh_CN)，Android [参见](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419317340&token=&lang=zh_CN)），小程序会获得打开 APP 的能力，此时用户点击按钮可以打开分享该卡片的 APP。即小程序不能打开任意 APP，只能 `跳回` 分享该小程序卡片的 APP。

在一个小程序的生命周期内，只有在特定条件下，才具有打开 APP 的能力。 `打开 APP 的能力` 可以理解为由小程序框架在内部管理的一个状态，为 true 则可以打开 APP，为 false 则不可以打开 APP。

在小程序的生命周期内，这个状态的初始值为 false，之后会随着小程序的每次打开（无论是启动还是切到前台）而改变：

*   当小程序从 1036（App 分享消息卡片） 打开时，该状态置为 true。
*   当小程序从 1089（微信聊天主界面下拉）或 1090（长按小程序右上角菜单唤出最近使用历史）的场景打开时，该状态不变，即保持上一次打开小程序时该状态的值。
*   当小程序从非 1036/1089/1090 的场景打开，该状态置为 false。

## 使用方法

#### iOS

下载最新的[微信SDK](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419317332&token=&lang=zh_CN)，或使用CocoaPods更新。

分享微信小程序：

![屏幕快照 2018-01-25 下午7.32.50.png](http://upload-images.jianshu.io/upload_images/1224641-607a67e769bcf93c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


1. 第一个字段WebpageUrl为备用链接，兼容低版本的微信客户端
2. userName为小程序的原始id，可以在小程序中查看
3. path为小程序中页面路径
4. Description为描述，等同于小程序中onShareAppMessage方法中的title
5. ThumbImage和hdImageData为分享图片的信息，需要小于128k，因此在分享前需要先下载要分享的图片，并压缩，最后resize到500*400(小程序卡片图片要求比例是5比4)

```
- (void)downloadCoverImage{
    SDWebImageManager *manager = [SDWebImageManager sharedManager];
    
    __weak typeof(self) weakSelf = self;
    
    [manager loadImageWithURL:self.item.itemCover.url
                          options:0
                         progress:^(NSInteger receivedSize, NSInteger expectedSizem, NSURL *targetUrl) {
                             //NSLog(@"receiveSize:%ld,expectedSize:%ld",(long)receivedSize,(long)expectedSize);
                         }
                        completed:^(UIImage *image,NSData *data, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
                            if (image) {
                                NSData *imageData = UIImageJPEGRepresentation(image, 0.1);
                                weakSelf.hdImageData = UIImageJPEGRepresentation(image, 0.1);
                                weakSelf.thumbImage = [[[UIImage alloc]initWithData:data] scaledToSize:CGSizeMake(500, 400)];
                                weakSelf.shareImage = [[[UIImage alloc]initWithData:imageData] scaledToSize:CGSizeMake(100, 100)];
                                
                            }
                        }];
    
    
    self.sharestr = [NSString stringWithFormat:@"只需%@元，广州女装源头货，买手精选，一件起批，赶紧来上新进货吧！",self.item.salePrice];
    
}
```

6. withShareTicked为YES时，是否带shareTicket，可以通过wx.getShareInfo方法获取群对当前小程序的唯一ID(OpenGid)
7. miniProgramType表示小程序类型，0是正式版，1是开发版，2是体验版

分享效果：

![131516879901_.pic.jpg](http://upload-images.jianshu.io/upload_images/1224641-14c92cd1efd60b5c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 小程序端

需要将 <button> 组件 open-type 的值设置为 launchApp。如果需要在打开 APP 时向 APP 传递参数，可以设置 app-parameter 为要传递的参数。通过 binderror 可以监听打开 APP 的错误事件。

```
<button open-type="launchApp" app-parameter="itemId=12345&userId=1234" binderror="launchAppError">打开APP</button>
```

若分享成功后，微信唤起App，并传递app-parameter参数到App:

![App中处理微信小程序传递参数](http://upload-images.jianshu.io/upload_images/1224641-502bbd52d35d29f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在App中添加WXApiDelegate中的onReq方法，处理参数。

如果唤起App失败，在binderror对应的方法中处理唤起失败后的逻辑：

```
Page({ 
    launchAppError: function(e) { 
        console.log(e.detail.errMsg) 
    } 
})
```

如果是没有安装App，可以使用wx.previewImage的方法弹出引导关注公众号的图片文案。

![未命名.gif](http://upload-images.jianshu.io/upload_images/1224641-02c580fec99f460a.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

都知道从公众号文章或者底部菜单是可以调转到小程序的，从小程序是不可以调转回公众号的，从上面的动画中，你有没有发现亮点？这里有个小技巧，在微信小程序中长按previewImage打开的图片，是可以识别二维码的，接着直接跳转到公众号！

同学们不要开心的太早，这个功能不是每次都能实现的，有兴趣的可以自己试一下。

![131516886582_.pic.jpg](http://upload-images.jianshu.io/upload_images/1224641-aebf96d79a2b65dd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



