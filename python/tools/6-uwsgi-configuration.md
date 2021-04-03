---
layout: default
title: uWSGI配置详解
parent: Python Web开发工具
nav_order: 6
---

# uWSGI的安装及配置详解
{: .no_toc }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---
uWSGI是一个Python Web服务器,它实现了WSGI协议、uwsgi、http等协议，常在部署Django或Flask开发的Python Web项目时使用，作为连接Nginx与应用程序之间的桥梁。本章总结了uWSGI服务器的作用以及在部署Python Web项目时如何安装和配置uWSGI。
{: .fs-6 .fw-300 }

## 为什么需要uWSGI?
在生产环境中部署Python Web项目时，uWSGI负责处理Nginx转发的动态请求，并与我们的Python应用程序沟通，同时将应用程序返回的响应数据传递给Nginx。

 ```bash
客户端 <-> Nginx <-> uWSGI <-> Python应用程序(Django, Flask)
 ```

或许你要问了，Nginx本身就是Web服务器，我们为什么还需要uWSGI这个Web服务器呢? Django不是自带runserver服务器?Flask不是自带Werkzeug吗? 答案是Nginx处理静态文件非常优秀，却不能直接与我们的Python Web应用程序进行交互。Django和Flask本身是Web框架，并不是Web服务器，它们自带的runserver和Werkzeug也仅仅用于开发测试环境，生产环境中处理并发的能力太弱。

为了解决Web 服务器与应用程序之间的交互问题，就出现了Web 服务器与应用程序之间交互的规范。最早出现的是CGI,后来又出现了改进 CGI 性能的FasgCGI，Java 专用的 Servlet 规范。在Python领域，最知名的就是WSGI规范了。

WSGI 全称是 Web Server Gateway Interface，也就是 Web 服务器网关接口，是一个web服务器（如uWSGI服务器）与web应用（如用Django或Flask框架写的程序）通信的一种规范。WSGI包含了很多自有协议，其中一个是uwsgi，它用于定义传输信息的类型。

现在你清楚uWSGI, WSGI和uwsgi的区别了吗?

- uWSGI是Python Web服务器，实现了WSGI通信规范和uwsgi协议；
- WSGI全名Web Server Gateway Interface，是一个Web服务器（如uWSGI服务器）与web应用（如用Django或Flask框架写的程序）通信的一种规范；
- uwsgi是WSGI通信规范中的一种自有协议。

## uWSGI的安装

```bash
pip install uwsgi
```

为了测试uWSGI安装是否成功，可以编写一个`test.py`的测试文件，添加如下代码：

```python
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"]
```

然后使用如下命令启动uWSGI Web服务器, 端口8080.

```bash
uwsgi --http :8080 --wsgi-file test.py
```

如果你已经有了一个现成的Django项目，你可以使用如下命令启动Web服务。

```bash
# 使用uwsgi命令行启动Django项目，端口8000
$ uwsgi --http :8000 --module myproject.wsgi
```

在生产环境中我们通常不会使用命令行启动Python Web项目，而是通常编辑好uWSGI配置文件`uwsgi.ini`, 然后使用如下命令启动Python Web项目。

```bash
# 使用uwsgi.ini配置文件启动Django应用程序
$ uwsgi --ini uwsgi.ini
```

## uWSGI 常用命令

```bash
# 启动uWSGI服务器
$ uwsgi --ini uwsgi.ini

# 重启uWSGI服务器
$ sudo service uwsgi restart

# 查看所有uWSGI进程
$ ps aux | grep uwsgi

# 停止所有uWSGI进程
$ sudo pkill -f uwsgi -9
```

## uWSGI常用配置

uWSGI常用配置选项如下所示，稍加修改(项目名，项目根目录)即可部署大部分Python Web项目。

```bash
[uwsgi]
uid=www-data # Ubuntu系统下默认用户名
gid=www-data # Ubuntu系统下默认用户组
project=mysite1  # 项目名
base = /home/user1 # 项目根目录

home = %(base)/Env/%(project) # 设置项目虚拟环境,Docker部署时不需要
chdir=%(base)/%(project) # 设置工作目录
module=%(project).wsgi:application # wsgi文件位置

master=True # 主进程
processes=2 # 同时进行的进程数，一般

# 选项1, 使用unix socket与nginx通信，仅限于uwsgi和nginx在同一主机上情形
# Nginx配置中uwsgi_pass应指向同一socket文件
socket=/run/uwsgi/%(project).sock

# 选项2，使用TCP socket与nginx通信
# Nginx配置中uwsgi_pass应指向uWSGI服务器IP和端口
# socket=0.0.0.0:8000 或则 socket=:8000

# 选项3，使用http协议与nginx通信
# Nginx配置中proxy_pass应指向uWSGI服务器一IP和端口
# http=0.0.0.0:8000 

# socket权限设置
chown-socket=%(uid):www-data
chmod-socket=664

# 进程文件
pidfile=/tmp/%(project)-master.pid

# 以后台守护进程运行，并将log日志存于temp文件夹。
daemonize=/var/log/uwsgi/%(project).log 

# 服务停止时，自动移除unix socket和pid文件
vacuum=True

# 为每个工作进程设置请求数的上限。当处理的请求总数超过这个量，进程回收重启。
max-requests=5000

# 当一个请求花费的时间超过这个时间，那么这个请求都会被丢弃。
harakiri=60

#当一个请求被harakiri杀掉会，会输出一条日志
harakiri-verbose=true

# uWsgi默认的buffersize为4096，如果请求数据超过这个量会报错。这里设置为64k
buffer-size=65536

# 如果http请求体的大小超过指定的限制，打开http body缓冲，这里为64k
post-buffering=65536

#开启内存使用情况报告
memory-report=true

#设置平滑的重启（直到处理完接收到的请求）的长等待时间(秒)
reload-mercy=10

#设置工作进程使用虚拟内存超过多少MB就回收重启
reload-on-as=1024
```

注意：uWSGI和Nginx之间有多种通信方式, unix socket，http-socket和http。Nginx的配置必需与uwsgi配置保持一致。

```bash
# 选项1, 使用unix socket与nginx通信
# 仅限于uwsgi和nginx在同一主机上情形
# Nginx配置中uwsgi_pass应指向同一socket文件地址
socket=/run/uwsgi/%(project).sock

# 选项2，使用TCP socket与nginx通信
# Nginx配置中uwsgi_pass应指向uWSGI服务器IP和端口
socket==0.0.0.0:8000 或则 socket=:8000

# 选项3，使用http协议与nginx通信
# Nginx配置中proxy_pass应指向uWSGI服务器IP和端口
http==0.0.0.0:8000 
```

如果你的nginx与uwsgi在同一台服务器上，优先使用本地机器的unix socket进行通信，这样速度更快。此时nginx的配置文件如下所示：

```bash
location / {     
    include /etc/nginx/uwsgi_params;
    uwsgi_pass unix:/run/uwsgi/django_test1.sock;
}
```

如果nginx与uwsgi不在同一台服务器上，两者使用TCP socket通信，nginx可以使用如下配置：

```bash
location / {     
    include /etc/nginx/uwsgi_params;
    uwsgi_pass uWSGI_SERVER_IP:8000;
}
```

如果nginx与uwsgi不在同一台服务器上，两者使用http协议进行通信，nginx配置应修改如下：

```bash
location / {     
    # 注意：proxy_pass后面http必不可少哦！
    proxy_pass http://uWSGI_SERVER_IP:8000;
}
```

## 小结

本文介绍了uWSGI的作用及其与WSGI和uwgsi的区别，并详解介绍了如何安装, 配置和使用它。Python Web领域还有一个遵循WSGI通信规范的Gunicorn，它同样优秀，我们后面再做介绍。

我是大江狗，一名Python Web技术开发爱好者。您可以通过搜索【<a href="https://blog.csdn.net/weixin_42134789">CSDN大江狗</a>】、【<a href="https://www.zhihu.com/people/shi-yun-bo-53">知乎大江狗</a>】和搜索微信公众号【Python Web与Django开发】关注我！

![Python Web与Django开发](../../assets/images/django.png)