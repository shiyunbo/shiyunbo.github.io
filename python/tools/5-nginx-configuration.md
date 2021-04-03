---
layout: default
title: Nginx配置详解
parent: Python Web开发工具
nav_order: 5
---

# Nginx的安装及配置详解
{: .no_toc }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

---
Nginx是一个强大的免费开源的HTTP服务器和反向代理服务器。在Web开发项目中，nginx常用作为静态文件服务器处理静态文件，并负责将动态请求转发至应用服务器(Django, Flask, etc)。熟悉Nginx的配置对Web开发和运维人员来说至关重要。本文整理了Nginx的配置大全，可以作为开发者参考。
{: .fs-6 .fw-300 }

## Nginx的安装
Nginx的安装很简单，网上教程很多，这里仅以Ubuntu系统(Linux)演示：

```bash
# 安装nginx
sudo apt-get install nginx

# 启动nginx服务
sudo systemctl nginx start
```
Nginx启动时通常会使用默认设置, 使用如下命令可以让自己的配置文件生效。
```bash
# 删除/etc/nginx/sites-available/目录下默认自定义配置
sudo rm -rf /etc/nginx/sites-available/default

# sites-available目录下新建自定义配置文件,可以1个网站1个
sudo nano /etc/nginx/sites-available/myapp1

# 与sites-enabled目录建立软链，可让自定义配置文件生效
sudo ln -s /etc/nginx/sites-available/myapp1 /etc/nginx/sites-enabled

# 检查nginx配置文件是否有问题
sudo systemctl nginx –t

# 重启nginx服务
sudo systemctl nginx restart
```

**注意**：如果你不是使用`sudo apt-get`命令安装的nginx，而是直接使用`yum`命令或docker镜像安装的，nginx的默认自定义配置文件位于`/etc/nginx/conf.d/default.conf`。如果你希望让自定义配置文件生效，你需要先删除这个默认配置文件，然后在``/etc/nginx/conf.d/`目录下新建一个`nginx.conf`。

如果你使用docker安装nginx，Dockerfile里可以设置将宿主机的当前目录下的`nginx.conf`复制一份到的容器内的`/etc/nginx/conf.d/`的目录下，如下所示：

```
# nginx镜像Dockerfile
FROM nginx:latest

# 删除原有配置文件，创建静态资源文件夹和ssl证书保存文件夹
RUN rm /etc/nginx/conf.d/default.conf 

# 将自定义配置文件nginx.conf复制到容器内/etc/nginx/conf.d/目录
ADD ./nginx.conf /etc/nginx/conf.d/

# 关闭守护模式
CMD ["nginx", "-g", "daemon off;"]
```

## Nginx配置文件构成

一个Nginx配置文件通常包含3个模块：

- 全局块：比如工作进程数，定义日志路径；
- Events块：设置处理轮询事件模型，每个工作进程最大连接数及http层的keep-alive超时时间；
- http块：路由匹配、静态文件服务器、反向代理、负载均衡等。

其中http块又可以进一步分成3块，http全局块里的配置对所有站点生效，server块配置仅对单个站点生效，而location块的配置仅对单个页面或url生效。

### Nginx配置文件示例

```bash
# 全局块
user www-data;
worker_processes  2;  ## 默认1，一般建议设成CPU核数1-2倍
error_log  logs/error.log; ## 错误日志路径
pid  logs/nginx.pid; ## 进程id

# Events块
events {
  # 使用epoll的I/O 模型处理轮询事件。
  # 可以不设置，nginx会根据操作系统选择合适的模型
  use epoll;
  
  # 工作进程的最大连接数量, 默认1024个
  worker_connections  2048;
  
  # http层面的keep-alive超时时间
  keepalive_timeout 60;
  
  # 客户端请求头部的缓冲区大小
  client_header_buffer_size 2k;
}

http { # http全局块
 
  include mime.types;  # 导入文件扩展名与文件类型映射表
  default_type application/octet-stream;  # 默认文件类型
  
  # 日志格式及access日志路径
  log_format   main '$remote_addr - $remote_user [$time_local]  $status '
    '"$request" $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';
  access_log   logs/access.log  main;
  
  # 允许sendfile方式传输文件，默认为off。
  sendfile     on;
  tcp_nopush   on; # sendfile开启时才开启。

  # http server块
  # 简单反向代理
  server {
    listen       80;
    server_name  domain2.com www.domain2.com;
    access_log   logs/domain2.access.log  main;
   
    # 转发动态请求到web应用服务器
    location / {
      proxy_pass      http://127.0.0.1:8000;
      deny 192.24.40.8;  # 拒绝的ip
      allow 192.24.40.6; # 允许的ip   
    }
    
    # 错误页面
    error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
  }

  # 负载均衡
  upstream backend_server {
    server 192.168.0.1:8000 weight=5; # weight越高，权重越大
    server 192.168.0.2:8000 weight=1;
    server 192.168.0.3:8000;
    server 192.168.0.4:8001 backup; # 热备
  }

  server {
    listen          80;
    server_name     big.server.com;
    access_log      logs/big.server.access.log main;
    
    charset utf-8;
    client_max_body_size 10M; # 限制用户上传文件大小，默认1M

    location / {
      # 使用proxy_pass转发请求到通过upstream定义的一组应用服务器
      proxy_pass      http://backend_server;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_redirect off;
      proxy_set_header X-Real-IP  $remote_addr;
    }
    
  }
}
```

接下来，我们仔细分析下Nginx各个模块的配置选项。

## Nginx Location配置
Nginx Location配置是Nginx的核心配置，它负责匹配请求的url, 并根据Location里定义的规则来处理这个请求，比如拒绝、转发、重定向或直接提供文件下载。

### URL匹配方式及优先级

Nginx的Location配置支持普通字符串匹配和正则匹配，不过url的各种匹配方式是有优先级的，如下所示：

| 匹配符 | 匹配规则                     | 优先级 |
| ------ | ---------------------------- | ------ |
| =      | 精确匹配                     | 1      |
| ^~     | 以某个字符串开头             | 2      |
| ~      | 区分大小写的正则匹配         | 3      |
| ~*     | 不区分大小写的正则匹配       | 4      |
| !~     | 区分大小写的不匹配正则       | 5      |
| !~*    | 不区分大小写的不匹配正则     | 6      |
| /      | 通用匹配，任何请求都会匹配到 | 7      |

为了加深你的理解，我们来看如下一个例子。由于规则2的优先级更高，当用户访问`/static/`或则`/static/123.html`时，Nginx会优先执行规则2里的操作，其它的的请求则会交由规则1执行。

```bash
# 规则1：通用匹配
location / {
}

# 规则2：处理以/static/开头的url
location ^~ /static {                         
    alias /usr/share/nginx/html/static; # 静态资源路径
}
```

**注意**：上例中我们使用了`alias`别名设置了静态文件所在目录，我们还可以使用`root`指定静态文件目录。注意：`alias`和`root`是有区别的。

- `root`对路径的处理：root路径 ＋ location路径
- `alias`对路径的处理：使用alias路径替换location路径

如果用`root`设置静态文件资源路径，可以按如下代码设置。两者是等同的。

```bash
# 规则2：处理以/static/开头的url
location ^~ /static {                         
    root /usr/share/nginx/html; # 静态资源路径
}
```

Location还支持正则匹配，比如下例可以禁止用户访问所有的图片格式文件。

```bash
# 拒绝访问所有图片格式文件
location ~* .*\.(jpg|gif|png|jpeg)$ {
        deny all;
}
```

### 请求转发和重定向

另一个我们在Location块里经常配置的就是转发请求或重定向，如下例所示：

```bash
# 转发动态请求
server {  
    listen 80;                                                         
    server_name  localhost;                                               
    client_max_body_size 1024M;

    location / {
        proxy_pass http://localhost:8080;   
        proxy_set_header Host $host:$server_port;
    }
}

# http请求重定向到https请求
server {
    listen 80;
    server_name domain.com;
    return 301 https://$server_name$request_uri;
}
```

无论是转发请求还是重定向，我们都使用了以`$`符号开头的变量，这些都是Nginx提供的全局变量。它们的具体含义如下所示：

```bash
$args, 请求中的参数;
$content_length, HTTP请求信息里的"Content-Length";
$content_type, 请求信息里的"Content-Type";
$document_root, 针对当前请求的根路径设置值;
$document_uri, 与$uri相同;
$host, 请求信息中的"Host"，如果请求中没有Host行，则等于设置的服务器名;
$limit_rate, 对连接速率的限制;
$request_method, 请求的方法，比如"GET"、"POST"等;
$remote_addr, 客户端地址;
$remote_port, 客户端端口号;
$remote_user, 客户端用户名，认证用;
$request_filename, 当前请求的文件路径名
$request_body_file,当前请求的文件
$request_uri, 请求的URI，带查询字符串;
$query_string, 与$args相同;
$scheme, 所用的协议，比如http或者是https，比如rewrite ^(.+)$ $scheme://example.com$1 redirect;
$server_protocol, 请求的协议版本，"HTTP/1.0"或"HTTP/1.1";
$server_addr, 服务器地址;
$server_name, 请求到达的服务器名;
$server_port, 请求到达的服务器端口号;
$uri, 请求的URI，可能和最初的值有不同，比如经过重定向之类的。
```

知道这些全局变量的含义后，我们就可以限制用户的请求方法。比如下例中配置了只允许用户通过POST方法访问，其他的请求方法则返回405。

```bash
if ($request_method !~ ^(GET|POST)$ ) { return 405; }
```

## Nginx静态文件配置

Nginx可直接作为强大的静态文件服务器使用，支持对静态文件进行缓存还可以直接将Nginx作为文件下载服务器使用。

### 静态文件缓存
缓存可以加快下次静态文件加载速度。我们很多与网站样式相关的文件比如css和js文件一般不怎么变化，缓存有效器可以通过`expires`选项设置得长一些。

```bash
    # 使用expires选项开启静态文件缓存，10天有效
    location ~ ^/(images|javascript|js|css|flash|media|static)/  {
      root    /var/www/big.server.com/static_files;
      expires 10d;
    }
```

### 静态文件压缩

Nginx可以对网站的css、js 、xml、html 文件在传输前进行压缩，大幅提高页面加载速度。经过Gzip压缩后页面大小可以变为原来的30%甚至更小。使用时仅需开启Gzip压缩功能即可。你可以在http全局块或server块增加这个配置。

```bash
http {
    
    # 开启gzip压缩功能
    gzip on;
    
    # 设置允许压缩的页面最小字节数; 这里表示如果文件小于10k，压缩没有意义.
    gzip_min_length 10k; 
    
    # 设置压缩比率，最小为1，处理速度快，传输速度慢；
    # 9为最大压缩比，处理速度慢，传输速度快; 推荐6
    gzip_comp_level 6; 
    
    # 设置压缩缓冲区大小，此处设置为16个8K内存作为压缩结果缓冲
    gzip_buffers 16 8k; 
    
    # 设置哪些文件需要压缩,一般文本，css和js建议压缩。图片视需要要锁。
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript; 
    
} 
```

### 文件下载服务器

Nginx也可直接做文件下载服务器使用，在location块设置`autoindex`相关选项即可。

```bash
server {

    listen 80 default_server;
    listen [::]:80 default_server;
    server_name  _;
    
    location /download {    
        # 下载文件所在目录
        root /usr/share/nginx/html;
        
        # 开启索引功能
        autoindex on;  
        
        # 关闭计算文件确切大小（单位bytes），只显示大概大小（单位kb、mb、gb）
        autoindex_exact_size off; 
        
        #显示本机时间而非 GMT 时间
        autoindex_localtime on;   
                
        # 对于txt和jpg文件，强制以附件形式下载，不要浏览器直接打开
        if ($request_filename ~* ^.*?\.(txt|jpg|png)$) {
            add_header Content-Disposition 'attachment';
        }
    }
}
```
## Nginx配置HTTPS

```
# 负载均衡，设置HTTPS
upstream backend_server {
    server APP_SERVER_1_IP;
    server APP_SERVER_2_IP;
}

# 禁止未绑定域名访问，比如通过ip地址访问
# 444:该网页无法正常运作，未发送任何数据
server {
    listen 80 default_server;
    server_name _;
    return 444;
}

# HTTP请求重定向至HTTPS请求
server {
    listen 80;
    listen [::]:80;
    server_name your_domain.com;
    
    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://backend_server; 
     }
    
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name your_domain.com;

    # ssl证书及密钥路径
    ssl_certificate /path/to/your/fullchain.pem;
    ssl_certificate_key /path/to/your/privkey.pem;

    # SSL会话信息
    client_max_body_size 75MB;
    keepalive_timeout 10;

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://django; # Django+uwsgi不在本机上，使用代理转发
    }

}
```

## Nginx日志配置

Nginx的日志主要包括访问日志`access_log`和错误日志`error_log`，你还可以通过`log_format`定义日志格式。你可以在全局块，Server块或Location块定义日志。比如下例在http块中定义了一个名为`main`的日志格式，所有站点的日志都会按这个格式记录。

```bash
http {
 # 日志格式及access日志路径
  log_format main '$remote_addr - $remote_user [$time_local]  $status '
    '"$request" $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';
  access_log   logs/access.log  main;
}
```

`access_log`文件随着访问记录增多有可能变得非常大，我们可以使用`access_log off`关闭一些不需要记录的访问。比如当一个站点没有设置favicon.ico时，`access_log`会记录了大量favicon.ico 404信息, 这是没有必要的, 可以按如下方式关闭访问日志记录。

```bash
location = /favicon.ico {
  log_not_found off; 
  access_log off; # 不在access_log记录该项访问
}
```


## Nginx超时设置

Nginx提供了很多超时设置选项，目的是保护服务器资源，CPU，内存并控制连接数。你可以根据实际项目需求在全局块、Server块和Location块进行配置。

### 请求超时设置

```bash
# 客户端连接保持会话超时时间，超过这个时间，服务器断开这个链接。
keepalive_timeout 60;

# 设置请求头的超时时间，可以设置低点。
# 如果超过这个时间没有发送任何数据，nginx将返回request time out的错误。
client_header_timeout 15;

# 设置请求体的超时时间，可以设置低点。
# 如果超过这个时间没有发送任何数据，nginx将返回request time out的错误。
client_body_timeout 15;

# 响应客户端超时时间
# 如果超过这个时间，客户端没有任何活动，nginx关闭连接。
send_timeout 15;

# 上传文件大小限制
client_max_body_size 10m;

# 也是防止网络阻塞，不过要包涵在keepalived参数才有效。
tcp_nodelay on;

# 客户端请求头部的缓冲区大小，这个可以根据你的系统分页大小来设置。
# 一般一个请求头的大小不会超过 1k，不过由于一般系统分页都要大于1k
client_header_buffer_size 2k;

# 这个将为打开文件指定缓存，默认是没有启用的。
# max指定缓存数量，建议和打开文件数一致，inactive 是指经过多长时间文件没被请求后删除缓存。
open_file_cache max=102400 inactive=20s;

# 这个是指多长时间检查一次缓存的有效信息。
open_file_cache_valid 30s;

# 告诉nginx关闭不响应的客户端连接。这将会释放那个客户端所占有的内存空间。
reset_timedout_connection on;
```

### Proxy反向代理超时设置

```bash
# 该指令设置与upstream服务器的连接超时时间，这个超时建议不超过75秒。
proxy_connect_timeout 60;

# 该指令设置应用服务器的响应超时时间，默认60秒。
proxy_read_timeout 60；

# 设置了发送请求给upstream服务器的超时时间
proxy_send_timeout 60;

# max_fails设定Nginx与upstream服务器通信的尝试失败的次数。
# 在fail_timeout参数定义的时间段内，如果失败的次数达到此值，Nginx就认为服务器不可用。
upstream big_server_com {
   server 192.168.0.1:8000 weight=5  max_fails=3 fail_timeout=30s; # weight越高，权重越大
   server 192.168.0.2:8000 weight=1  max_fails=3 fail_timeout=30s;
   server 192.168.0.3:8000;
   server 192.168.0.4:8001 backup; # 热备
}
```

## Nginx负载均衡

Nginx提供了多种负载均衡算法, 最常见的有5种。我们只需修改对应upstream模块即可。

### 轮询(默认)

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除;

```hash
# 轮询，大家权重一样
upstream backend_server {
   server 192.168.0.1:8000;
   server 192.168.0.2:8000;
   server 192.168.0.3:8000 down; # 不参与负载均衡
   server 192.168.0.4:8001 backup; # 热备
}

server {
   listen          80;
   server_name     big.server.com;
   access_log      logs/big.server.access.log main;
    
   charset utf-8;
   client_max_body_size 10M; # 限制用户上传文件大小，默认1M

   location / {
     # 使用proxy_pass转发请求到通过upstream定义的一组应用服务器
     proxy_pass      http://backend_server;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header Host $http_host;
     proxy_redirect off;
     proxy_set_header X-Real-IP  $remote_addr;
   }
```

### 权重(weight)

通过weight指定轮询几率，访问比率与weight成正比，常用于后端服务器性能不均的情况。不怎么忙的服务器可以多承担些任务。

```hash
# 权重，weight越大，承担任务越多
upstream backend_server {
   server 192.168.0.1:8000 weight=3;
   server 192.168.0.2:8000 weight=1;
}
```

### ip_hash ###

每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

```bash
# 权重，weight越大，承担任务越多
upstream backend_server {
   ip_hash;
   server 192.168.0.1:8000;
   server 192.168.0.2:8000;
}
```

### url_hash

按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。 

```bash
# URL Hash
upstream backend_server {
   hash $request_uri;
   server 192.168.0.1:8000;
   server 192.168.0.2:8000;
}
```

###  fair(第三方)

按后端服务器的响应时间来分配请求，响应时间短的优先分配。使用这个算法需要安装`nginx-upstream-fair`这个库。

```bash
# Fair
upstream backend_server {
   server 192.168.0.1:8000;
   server 192.168.0.2:8000;
   fair;
}
```

## Nginx与uWSGI服务器的沟通

在前面的案例中，Nginx都是使用`proxy_pass`转发的动态请求，`proxy_pass`使用普通的HTTP协议与应用服务器进行沟通。如果你部署的是Python Web应用(Django, Flask), 你的应用服务器(`uwsgi`, `gunicorn`)一般是遵守uwsgi协议的，对于这种情况，建议使用`uwsgi_pass`转发请求。

### Python Web应用部署负载均衡Nginx配置文件参考

如果你部署的是Django或则Flask Web应用，一个完整的nginx配置文件如下所示：

```bash
# nginx配置文件，nginx.conf

# 全局块
user www-data;
worker_processes  2;  ## 默认1，一般建议设成CPU核数1-2倍

# Events块
events {
  # 使用epoll的I/O 模型处理轮询事件。
  # 可以不设置，nginx会根据操作系统选择合适的模型
  use epoll;
  
  # 工作进程的最大连接数量, 默认1024个
  worker_connections  2048;
  
  # http层面的keep-alive超时时间
  keepalive_timeout 60;
  
}

http {    
    # 开启gzip压缩功能
    gzip on;
    
    # 设置允许压缩的页面最小字节数; 这里表示如果文件小于10k，压缩没有意义.
    gzip_min_length 10k; 
    
    # 设置压缩比率，最小为1，处理速度快，传输速度慢；
    # 9为最大压缩比，处理速度慢，传输速度快; 推荐6
    gzip_comp_level 6; 
    
    # 设置压缩缓冲区大小，此处设置为16个8K内存作为压缩结果缓冲
    gzip_buffers 16 8k; 
    
    # 设置哪些文件需要压缩,一般文本，css和js建议压缩。图片视需要要锁。
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript; 
    
    
    upstream backend_server {
        server 192.168.0.1:8000; # 替换成应用服务器或容器实际IP及端口
        server 192.168.0.2:8000;
    }

    server {
        listen 80; # 监听80端口
        server_name localhost; # 可以是nginx容器所在ip地址或127.0.0.1，不能写宿主机外网ip地址

        charset utf-8;
        client_max_body_size 10M; # 限制用户上传文件大小
        
         # 客户端请求头部的缓冲区大小
        client_header_buffer_size 2k;
        client_header_timeout 15;
        client_body_timeout 15;
    
        access_log /var/log/nginx/mysite1.access.log main;
        error_log /var/log/nginx/mysite1.error.log warn;
        
        # 静态资源路径
        location /static {
            alias /usr/share/nginx/html/static; 
        }
        
        # 媒体资源路径，用户上传文件路径
        location /media {
            alias /usr/share/nginx/html/media;
        }

        location / {     
            include /etc/nginx/uwsgi_params;
            uwsgi_pass backend_server;   # 使用uwsgi_pass, 而不是proxy_pass
            uwsgi_read_timeout 600; # 指定接收uWSGI应答的超时时间
            uwsgi_connect_timeout 600;  # 指定连接到后端uWSGI的超时时间。
            uwsgi_send_timeout 600; # 指定向uWSGI传送请求的超时时间

            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_redirect off;
            proxy_set_header X-Real-IP  $remote_addr;
        }
    }
    
} 


```

如果你的nginx与uwsgi在同一台服务器上，用不到负载均衡，你还可以通过本地机器的unix socket进行通信，这样速度更快，如下所示：

```bash
location / {     
    include /etc/nginx/uwsgi_params;
    uwsgi_pass unix:/run/uwsgi/django_test1.sock;
}
```

**注意**：取决于Nginx采用那种方式与uWSGI服务器进行通信(本地socket, 网络TCP socket和http协议)，uWSGI的配置文件也会有所不同。这里以`uwsgi.ini`为例展示了不同。

```bash
# uwsgi.ini配置文件

# 对于uwsgi_pass转发的请求，使用本地unix socket通信
# 仅适用于nginx和uwsgi在同一台服务器上的情形
socket=/run/uwsgi/django_test1.sock

# 对于uwsgi_pass转发的请求，使用TCP socket通信
socket=0.0.0.0:8000

# 对于proxy_pass HTTP转发的请求，使用http协议
http=0.0.0.0:8000
```

## 小结
本文总结了Nginx常用配置选项，包括url匹配优先级、请求转发、日志配置、超时配置、静态文件处理以及负载均衡的各项算法。下篇我们将专门介绍Nginx的好搭档，负责接收动态请求的uWSGI Web服务器。

我是大江狗，一名Python Web技术开发爱好者。您可以通过搜索【<a href="https://blog.csdn.net/weixin_42134789">CSDN大江狗</a>】、【<a href="https://www.zhihu.com/people/shi-yun-bo-53">知乎大江狗</a>】和搜索微信公众号【Python Web与Django开发】关注我！

![Python Web与Django开发](../../assets/images/django.png)