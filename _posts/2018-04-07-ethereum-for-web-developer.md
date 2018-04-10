---
layout: post
title: web开发者看以太坊
categories: 区块链
description: web开发者看以太坊
keywords: Ethereum, 区块链
---

学习[Ethereum blockchain platform](https://ethereum.org/)有一段时间了，越看越兴奋，根本停不下来。关于Ethereum的学习资料，包括文章、视频、文档等有很多，但无法满足系统化的学习需求，有点懵逼。还有一点是，Ethereum发展很快，很多文档或资料已经过时了。经过一段时间的学习和总结，对Ethereum的整体概念和运作方式有了基础的了解。本篇文章会以web开发者的角度介绍一下Ethereum，作为开发者，如何基于以太坊，创建去中心化的应用？

如果你是一个web开发人员，你会很了解web应用的整体架构：

![webapp](https://upload-images.jianshu.io/upload_images/1224641-2ee86274065234a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

web应用部署在服务器上，用户们会和一个中心化的应用进行交互。这里用户可以是浏览器，app，api接口等。当用户向服务端发起请求时，服务端通过对数据库或缓存的增删改查来响应用户的请求。

这样的架构在目前来讲是很合理的。如果这个中心化的数据库是公开的，每个人都有权限访问，且你不需要这个应用的开发者来掌管你的数据就能保证数据的安全性，这些条件在一些场景下构建的应用会更合理一些。

比如淘宝。如果你在淘宝上业绩很好，且拥有庞大的用户积累，但是有一天你的账号因为某些原因被禁了，你在淘宝上卖不了东西了，这会严重影响你的生意，因为脱离的淘宝，你的一切信用积累，用户积累就都没了。假如你可以方便的把历史成交数据，评价数据，用户数据等移植到另外一个平台上，继续做你的生意，听起来是不是很好？这就是去中心化应用的由来。Ethereum使得开发者很方便的创建去中心化的应用(Dapp, decentralized application)。

Dapp的整体架构大概如下：

![dapp](https://upload-images.jianshu.io/upload_images/1224641-48003f937988eacb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


你会发现，每个client会跟他所属的应用实例进行交互，没有一个中心的server。这就意味着，用户在使用Dapp前，需要将该应用已有的blockchain完全复制到自己的电脑或者手机上。乍一听很荒唐，这多费流量，多影响用户体验啊。但这是一个去中心化应用必要的流程，这样做可以使得应用不依赖一于一个中心化的server。

其实复制整个blockchain消耗不了多少时间和容量，已经有很完善的技术方案在保证去中心化特点的同时，又使得交互变得很简单，很流畅，这里就不展开了（我还没看懂- -!）

那么，在Ethereum中，blockchain的作用是什么呢？

## Database

在以太坊网络中所有的transactions都被存住在block中(这里牵扯到hash pointer, merklin tree,区块头等btc里面的知识点，就不展开了)，block之间通过hash pointer指向另一个block，形成一条不可篡改的blockchain。回到淘宝的例子，用户在淘宝的交易，不管是买家还是卖家，产生的一切行为都被记录在区块中，且所有人都有权限访问很验证。

## Code

blockchain用来存储数据，如何进行增删改查的逻辑操作呢？以太坊提供了[Solidity](https://solidity.readthedocs.io/en/develop/)语言编写dapp，或叫做contract。编译，部署你的contract到以太坊网络中，你的去中心化应用就可以使用了。

Ethereum提供了web3.js框架方便开发者使用javascript处理逻辑层和blockchain节点之间的通讯和逻辑关系。所以，你只需要在熟悉的框架下，如reactjs, angularjs 等引用web3.js，就可以方便的创建web based 去中心化应用了。

这里只是介绍了以太坊的基础知识，很多知识点没有展开，后续学习过程中有了更深刻的理解，我会及时补充。

接下来我会通过一个投票应用来完整的介绍基于Ethereum的去中心化应用的开发流程:

代码地址：[https://github.com/sam408130/ethereum_voting_dapp](https://github.com/sam408130/ethereum_voting_dapp)

* [Full Stack Hello World Voting Ethereum Dapp Tutorial — Part 1](http://geeksai.com/2018/04/08/full-stack-voting-part1/)
* [Full Stack Hello World Voting Ethereum Dapp Tutorial — Part 2](http://geeksai.com/2018/04/09/full-stack-voting-part2/)
* [Full Stack Hello World Voting Ethereum Dapp Tutorial — Part 3](http://geeksai.com/2018/04/10/full-stack-voting-part3/)


原文地址： [Ethereum for web developers, Mahesh Murthy](https://medium.com/@mvmurthy/ethereum-for-web-developers-890be23d1d0c)





