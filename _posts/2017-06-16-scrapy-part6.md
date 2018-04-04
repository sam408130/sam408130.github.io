---
layout: post
title: Python爬虫系列（六）：supervisor + scrapyd + spiderkeeper的scrapy部署与管理
categories: Python
description: 京东App(如何分析https下的api结构，及数据爬取)
keywords: Python, Scrapy
---

##  1.scrapyd
scrapyd 是由scrapy 官方提供的爬虫管理工具，使用它我们可以非常方便地上传、控制爬虫并且查看运行日志。
参考官方文档   [http://scrapyd.readthedocs.org/en/latest/api.html](http://scrapyd.readthedocs.org/en/latest/api.html)

###### Installing

``` pip install scrapyd ```

###### Usage

``` scrapyd ```


## 2.spiderkeeper

主要实现 scrapy 工程的部署，抓取任务状态监控，定时启动爬虫等功能。支持多个 scrapyd 服务 ，方便爬虫集群的管理
[Github](https://github.com/DormyMo/SpiderKeeper)

###### Installing

``` pip install spiderkeeper ```

###### Deployment

```
spiderkeeper [options]

Options:

  -h, --help            show this help message and exit
  --host=HOST           host, default:0.0.0.0
  --port=PORT           port, default:5000
  --username=USERNAME   basic auth username ,default: admin
  --password=PASSWORD   basic auth password ,default: admin
  --type=SERVER_TYPE    access spider server type, default: scrapyd
  --server=SERVERS      servers, default: ['http://localhost:6800']
  --database-url=DATABASE_URL
                        SpiderKeeper metadata database default: sqlite:////home/souche/SpiderKeeper.db
  --no-auth             disable basic auth
  -v, --verbose         log level
  

example:

spiderkeeper --server=http://localhost:6800

```

###### Usage

```
Visit: 

- web ui : http://localhost:5000

1. Create Project

2. Use [scrapyd-client](https://github.com/scrapy/scrapyd-client) to generate egg file 

   scrapyd-deploy --build-egg output.egg

2. upload egg file (make sure you started scrapyd server)

3. Done & Enjoy it

- api swagger: http://localhost:5000/api.html

```

###### Screenshot

![image.png](http://upload-images.jianshu.io/upload_images/1224641-74ead7d83d7fe6b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](http://upload-images.jianshu.io/upload_images/1224641-3e86b0134d75162a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![image.png](http://upload-images.jianshu.io/upload_images/1224641-acdd2a79b2ca9129.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](http://upload-images.jianshu.io/upload_images/1224641-65e334192d0bca0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





## 3.supervisor

Supervisor ([http://supervisord.org](http://supervisord.org/)) 是一个用 Python 写的进程管理工具，可以很方便的用来启动、重启、关闭进程（不仅仅是 Python 进程）。除了对单个进程的控制，还可以同时启动、关闭多个进程，比如很不幸的服务器出问题导致所有应用程序都被杀死，此时可以用 supervisor 同时启动所有应用程序而不是一个一个地敲命令启动。

###### Installing

```
pip install supervisor
```

###### Setting

Supervisor 相当强大，提供了很丰富的功能，不过我们可能只需要用到其中一小部分。安装完成之后，可以编写配置文件，来满足自己的需求。为了方便，我们把配置分成两部分：supervisord（supervisor 是一个 C/S 模型的程序，这是 server 端，对应的有 client 端：supervisorctl）和应用程序（即我们要管理的程序）。
首先来看 supervisord 的配置文件。安装完 supervisor 之后，可以运行echo_supervisord_conf 命令输出默认的配置项，也可以重定向到一个配置文件里：

```
echo_supervisord_conf > /etc/supervisord.conf
```

去除里面大部分注释和“不相关”的部分，我们可以先看这些配置：

```
[unix_http_server]
file=/tmp/supervisor.sock   ; UNIX socket 文件，supervisorctl 会使用
;chmod=0700                 ; socket 文件的 mode，默认是 0700
;chown=nobody:nogroup       ; socket 文件的 owner，格式： uid:gid

;[inet_http_server]         ; HTTP 服务器，提供 web 管理界面
;port=127.0.0.1:9001        ; Web 管理后台运行的 IP 和端口，如果开放到公网，需要注意安全性
;username=user              ; 登录管理后台的用户名
;password=123               ; 登录管理后台的密码

[supervisord]
logfile=/tmp/supervisord.log ; 日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB        ; 日志文件大小，超出会 rotate，默认 50MB
logfile_backups=10           ; 日志文件保留备份数量默认 10
loglevel=info                ; 日志级别，默认 info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ; pid 文件
nodaemon=false               ; 是否在前台启动，默认是 false，即以 daemon 的方式启动
minfds=1024                  ; 可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ; 可以打开的进程数的最小值，默认 200

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; 通过 UNIX socket 连接 supervisord，路径与 unix_http_server 部分的 file 一致
;serverurl=http://127.0.0.1:9001 ; 通过 HTTP 的方式连接 supervisord

; 包含其他的配置文件
[include]
files = relative/directory/*.ini    ; 可以是 *.conf 或 *.ini
```

我们把上面这部分配置保存到 /etc/supervisord.conf（或其他任意有权限访问的文件），然后启动 supervisord（通过 -c 选项指定配置文件路径：

``` supervisord -c /etc/supervisord.conf ```

查看 supervisord 是否在运行：

``` ps aux | grep supervisord```

###### program setting

上面我们已经把 supervisrod 运行起来了，现在可以添加我们要管理的进程的配置文件。可以把所有配置项都写到 supervisord.conf 文件里，但并不推荐这样做，而是通过 include 的方式把不同的程序（组）写到不同的配置文件里。
为了举例，我们新建一个目录 /etc/supervisor/ 用于存放这些配置文件，相应的，把 /etc/supervisord.conf 里 include 部分的的配置修改一下：

```
[include]
files = /etc/supervisor/*.conf
```

接下来将scrapyd和spiderkeeep的部署命令填写到配置文件里

```

[program:spiderkeeper]
command=spiderkeeper --server=http://localhost:6800 --username=sam --password=*****
directory=/srv/www/python/
autostart=true
autorestart=true
startretries=3


[program:scrapyd]
command=source /srv/www/python/pyenv/bin/activate
directory=/srv/www/python/captain
command=scrapyd
autostart=true
autorestart=true
redirect_stderr=true                     
```
一份配置文件至少需要一个 [program:x] 部分的配置，来告诉 supervisord 需要管理那个进程。[program:x] 语法中的 x 表示 program name，会在客户端（supervisorctl 或 web 界面）显示，在 supervisorctl 中通过这个值来对程序进行 start、restart、stop 等操作。

###### supervisorctl
Supervisorctl 是 supervisord 的一个命令行客户端工具，启动时需要指定与 supervisord 使用同一份配置文件，否则与 supervisord 一样按照顺序查找配置文件。

``` 
(pyenv)[root@iZ2597c5i5hZ supervisor.d]# supervisorctl
scrapyd                          RUNNING   pid 25070, uptime 1:09:28
spiderkeeper                     RUNNING   pid 25068, uptime 1:09:28
upload                           RUNNING   pid 25069, uptime 1:09:28
supervisor> 
```
上面这个命令会进入 supervisorctl 的 shell 界面，然后可以执行不同的命令了：
```
> status    # 查看程序状态
> stop usercenter   # 关闭 usercenter 程序
> start usercenter  # 启动 usercenter 程序
> restart usercenter    # 重启 usercenter 程序
> reread    ＃ 读取有更新（增加）的配置文件，不会启动新添加的程序
> update    ＃ 重启配置文件修改过的程序
```

上面这些命令都有相应的输出，除了进入 supervisorctl 的 shell 界面，也可以直接在 bash 终端运行：

```
$ supervisorctl status
$ supervisorctl stop usercenter
$ supervisorctl start usercenter
$ supervisorctl restart usercenter
$ supervisorctl reread
$ supervisorctl update 
```

## 4. nginx配置
设置80端口跳转到5000上

```
server {
        listen 80;
        server_name 你的域名;
        location / {
                proxy_pass http://127.0.0.1:5000;
                proxy_set_header Host $host:80;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

}
```

