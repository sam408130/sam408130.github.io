---
layout: post
title: Python爬虫系列（四）： Project 1:下厨房(Web端)
categories: Python
description: Project 1:下厨房(Web端)
keywords: Python, Scrapy
---


#### 实战第一篇，以下厨房网页端为例，任务目标：

1. 爬取下厨房网页端所有的菜品
2. 创建基本的工具类，数据管理工具
3. 将爬取的数据结构化保存到数据库中

#### 以下是下厨房的首页：

![屏幕快照 2017-07-01 下午12.09.13.png](http://upload-images.jianshu.io/upload_images/1224641-1dbbd1af2a6eb496.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从网页结构上分析，分类是个很好的爬取所有菜品的入口，点开菜谱分类：

![屏幕快照 2017-07-01 下午12.12.28.png](http://upload-images.jianshu.io/upload_images/1224641-1afb5b5b0f3ef9d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击其中一个分类：

![屏幕快照 2017-07-01 下午12.13.45.png](http://upload-images.jianshu.io/upload_images/1224641-d36a2fe8b147c10c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 到此，基本思路已经很清晰：

1. 爬取所有的分类
2. 通过分类进入菜品列表，爬取该分类下所有菜品

#### 难点有两个:

1. 分类页有个【展开全部】的action，如何得到一个大分类下的所有二级分类？
2. 如何爬取一个二级分类下的所有页数据？

#### 问题1

打开浏览器，查看分类页面的源码：

![屏幕快照 2017-07-01 下午12.25.02.png](http://upload-images.jianshu.io/upload_images/1224641-6d604381ccac11fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不难发现，点击『展开全部』后隐藏的数据都是存放在```<div class='cates-list-all clearfix hidden'>```下的，只要取该```div```下的数据，变可以得到全量的分类，其结构如下图：

![image.png](http://upload-images.jianshu.io/upload_images/1224641-10541919468db96a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
到此问题1已解决

#### 问题2

首先查看分页部分的源代码：

![image.png](http://upload-images.jianshu.io/upload_images/1224641-2f70e746a454d90f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

只要从每页中取出下一页的链接，依次爬取每一页的内容即可，知道下一页的链接为空时，及表示已是最后一页，该二级分类下的所有菜品都爬完了。


#### 开始写代码

##### 1. 创建project

```
scrapy startproject cook
```

文件结构

```
(env) ➜  cook tree
.
├── scrapy.cfg
└── cook
    ├── __init__.py
    ├── items.py
    ├── middlewares.py
    ├── pipelines.py
    ├── settings.py
    └── spiders
        └── __init__.py

2 directories, 7 files
```

关于scrapy的基本内容这里不多做介绍，不熟悉的同学可以查阅本系列第二篇文章：[Python爬虫系列（三）：python scrapy介绍和使用](http://www.jianshu.com/p/bf4198b224be)


##### 2. 创建分类表结构

```
        command = (
            "CREATE TABLE IF NOT EXISTS {} ("
            "`id` INT(8) NOT NULL AUTO_INCREMENT,"
            "`name` CHAR(20) NOT NULL COMMENT '分类名称',"
            "`url` TEXT NOT NULL COMMENT '分类url',"
            "`category` CHAR(20) NOT NULL COMMENT '父级分类',"
            "`category_id` INT(8) NOT NULL COMMENT '分类id',"
            "`create_time` DATETIME NOT NULL,"
            "PRIMARY KEY(id),"
            "UNIQUE KEY `category_id` (`category_id`)"
            ") ENGINE=InnoDB".format(config.category_urls_table)
        )

        self.sql.create_table(command)
```

##### 3. 解析部分

```
    def parse_all(self, response):
        if response.status == 200:
            file_name = '%s/category.html' % (self.dir_name)
            self.save_page(file_name, response.body)
            categorys = response.xpath("//div[@class='cates-list-all clearfix hidden']").extract()
            for category in categorys:
                sel_category = Selector(text = category)
                category_father = sel_category.xpath("//h4/text()").extract_first().strip()
                items = sel_category.xpath("//ul/li/a").extract()
                for item in items:
                    sel = Selector(text = item)
                    url = sel.xpath("//@href").extract_first()
                    name = sel.xpath("//text()").extract_first()
                    _id = re.compile('/category/(.*?)/').findall(url)[0]
                    dt = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                    msg = (None , name, url, category_father, _id, dt)
                    command = ("INSERT IGNORE INTO {} "
                                "(id, name, url, category, category_id, create_time)"
                                "VALUES(%s,%s,%s,%s,%s,%s)".format(config.category_urls_table)
                    )
                    self.sql.insert_data(command, msg)
```

根据问题1里面的描述，我们要从```http://www.xiachufang.com/category/```中找出所有的二级分类以及其连接，每个一级分类对应的数据都在```<div class='cates-list-all clearfix hidden'>```下，参考代码：

```
categorys = response.xpath("//div[@class='cates-list-all clearfix hidden']").extract()
```

接着我们将所有二级分类的名称，对应url取出存入数据中。

##### 4.爬取菜品 

先取出之前存储到数据控的所有分类：

```
        command = "SELECT * from {}".format(config.category_urls_table)
        data = self.sql.query(command)
```

解析部分：

```
    def parse_all(self, response):
        utils.log(response.url)
        if response.status == 200:
            file_name = '%s/category.html' % (self.dir_name)
            self.save_page(file_name, response.body)
            recipes = response.xpath("//div[@class='normal-recipe-list']/ul/li").extract()
            self.parse_recipes(recipes)
            nextPage = response.xpath("//div[@class='pager']/a[@class='next']/@href").extract_first()
            if nextPage:
                yield Request(
                    url = self.base_url + nextPage,
                    headers = self.header,
                    callback = self.parse_all,
                    errback = self.error_parse,
                )

    def parse_recipes(self, recipes):
        for recipe in recipes:
            sel = Selector(text = recipe)
            name = sel.xpath("//p[@class='name']/text()").extract_first().strip()
            url = sel.xpath("//a[1]/@href").extract_first()
            img = sel.xpath("//div[@class='cover pure-u']/img/@data-src").extract_first()
            item_id = re.compile("/recipe/(.*?)/").findall(url)[0]
            source = sel.xpath("//p[@class='ing ellipsis']/text()").extract_first().strip()
            score = sel.xpath("//p[@class='stats']/span/text()").extract_first().strip()
            dt = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            msg = (None, name, url, img, item_id, source, score, dt)
            command = ("INSERT IGNORE INTO {} "
                        "(id, name, url, img, item_id, source, score, create_time)"
                        "VALUES(%s,%s,%s,%s,%s,%s,%s,%s)".format(config.item_list_table)
            )
            self.sql.insert_data(command, msg)
```

通过分析源代码，先取出每一页的所有菜品信息部分：

```
recipes = response.xpath("//div[@class='normal-recipe-list']/ul/li").extract()
```

通过方法```parse_recipes```解析出该页所有的菜品信息，并存储。
同时取出『下一页』对应的url（nextPage）,如果nextPage不为空（还有下一页），则接着爬取下一页内容。

##### 5.执行爬虫程序

依次执行以上的两个spider程序，下厨房所有的菜品就到手了，有兴趣的同学也可以接着爬取菜品的详情页内容。

![屏幕快照 2017-07-01 下午1.05.23.png](http://upload-images.jianshu.io/upload_images/1224641-58974433dba48a82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![屏幕快照 2017-07-01 下午1.05.31.png](http://upload-images.jianshu.io/upload_images/1224641-6b20d21815c67395.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


源码地址：https://github.com/sam408130/xcf_crawler
交流学习qq:197329984

## 再次声明，本篇文章仅用于交流学习，请勿用于任何商业用途

接下来文章内容预告：
1. 如果爬取京东app（https通信）数据
2. 如何添加代理
3. scrapy的管理，部署