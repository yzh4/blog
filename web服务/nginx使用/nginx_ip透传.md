## 1. 介绍

  在使用了CDN做加速站点静态资源加速后，当用户请求的静态资源没能命中，此时CDN会到源站请求内容，那么此时访问源站的IP为CDN节点的IP，不仅如此，可能经我们的WAF防火墙和前端的负载均衡(SLB)后更不容易获取到真实的用户IP信息，我们如果要统计用户的访问IP和地区就变得比较麻烦，因为可能不是真实的IP，必须使用一个什么机制将用户IP传递到最终后端的应用服务器才行。

## 2. 环境

| 主机   | 主机名/IP配置         | 备注             |
| :----- | :-------------------- | :--------------- |
| LB-01  | node01/192.168.71.133 | 一级代理         |
| LB-02  | node02/192.168.71.134 | 二级代理         |
| LB-03  | node03/192.168.71.135 | 三级代理         |
| LB-04  | node04/192.168.71.136 | Web`[nginx]`服务 |
| Chrome | 192.168.5.6           | windows本机      |

> Nginx 版本 `nginx version: nginx/1.16.1`
>
> windows 是宿主机电脑，LB几个机器是在宿主机上使用的vmware的几个虚拟机

## 3. X-Real-IP透传方式

### 3.1 配置

  在每个HTTP请求头中加入X-Real-IP信息，若只存在1级的代理服务器，则该参数的确就是客户端的真实IP,但若是存在多级代理时，此信息为上级代理的IP信息，并不能获取到真实的用户IP，故此法目前已弃用。

> nginx配置

```
[root@node01 nginx]# cat /etc/nginx/conf.d/test.conf
server {
    listen 80;
    # server_name ip.test.com;
    location / {
        proxy_pass http://node02;
        proxy_http_version 1.1;
        #proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

[root@node02 nginx]# cat /etc/nginx/conf.d/test.conf
server {
    listen 80;
    # server_name ip.test.com;
    location / {
        proxy_pass http://node03;
        proxy_http_version 1.1;
        #proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

[root@node03 nginx]# cat /etc/nginx/conf.d/test.conf
server {
    listen 80;
    # server_name ip.test.com;
    location / {
        proxy_pass http://node04;
        proxy_http_version 1.1;
        #proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

[root@node04 nginx]# cat /etc/nginx/nginx.conf|egrep -v '#|^$'
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;
        location / {
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
```

> 注：配置完成后保存所有配置文件，然后启动所有的nginx服务。

### 3.2 访问测试

- windows 客户端访问

  我们在windows也就是本地浏览器访问测试。

- 查看各个nginx日志

```
[root@node01 nginx]# tail -1  /var/log/nginx/access.log
192.168.71.1 - - [13/Mar/2020:10:28:24 +0800] "GET /favicon.ico HTTP/1.1" 404 3650 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.25 Safari/537.36 Core/1.70.3741.400 QQBrowser/10.5.3863.400" "-"


[root@node02 nginx]# tail -1  /var/log/nginx/access.log
192.168.71.133 - - [12/Mar/2020:12:40:45 +0800] "GET /favicon.ico HTTP/1.1" 404 3650 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.25 Safari/537.36 Core/1.70.3741.400 QQBrowser/10.5.3863.400" "-"

[root@node03 nginx]# tail -1  /var/log/nginx/access.log
192.168.71.134 - - [12/Mar/2020:12:40:37 +0800] "GET /favicon.ico HTTP/1.1" 404 3650 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.25 Safari/537.36 Core/1.70.3741.400 QQBrowser/10.5.3863.400" "-"

[root@node04 nginx]# tail -1  /var/log/nginx/access.log
192.168.71.135 - - [13/Mar/2020:10:28:24 +0800] "GET /favicon.ico HTTP/1.1" 404 3650 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.25 Safari/537.36 Core/1.70.3741.400 QQBrowser/10.5.3863.400" "-"
```

> X-Real-IP并不能获取到真实的客户端IP地址，如果时单级代理情况下，可以获取到正确的客户端IP，若存在多级代理就歇菜了。

## 4. X-Forwarded-For透传方式

  在每个HTTP请求头中加入X-Forwarded-For信息，若只存在1级的代理服务器，则该参数为客户端的真实IP,若是存在多级代理时，每经过一级代理服务器，则追加上级代理服务的IP，可以获取到真实的用户IP，但若是遇到伪造的X-Forwarded-For信息或第一级代理未启用X-Forwarded-For都不能获取到真实用户IP，此法是目前比较常用的方法，但更推荐使用nginx_http_realip_module模块添加可信代理的方法。

>**nginx配置**

```
[root@node01 conf.d]# cat test.conf
server {
    listen 80;
    #server_name ip.test.com;

    location / {
        proxy_pass http://node02;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Referer $http_referer;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Real-Port $remote_port;
        proxy_set_header X-Real-User $remote_user;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

    }
}

[root@node02 conf.d]# cat test.conf
server {
    listen 80;
    #server_name ip.test.com;

    location / {
        proxy_pass http://node03;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Referer $http_referer;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Real-Port $remote_port;
        proxy_set_header X-Real-User $remote_user;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

[root@node03 conf.d]# cat test.conf
server {
    listen 80;
    #server_name ip.test.com;

    location / {
        proxy_pass http://node04;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Referer $http_referer;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Real-Port $remote_port;
        proxy_set_header X-Real-User $remote_user;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

[root@node04 nginx]# egrep -v '#|^$' nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;
    proxy_set_header Host $host;
    proxy_set_header Referer $http_referer;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Real-Port $remote_port;
    proxy_set_header X-Real-User $remote_user;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;
        location / {
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
```

## 5. Realip_module透传方式

```
http://nginx.org/en/docs/http/ngx_http_realip_module.html
```

> 需要确认当前nginx的版本是否有该模块 `nginx -V 中查看--with-http_realip_module` 使用nginx Realip_module获取多级代理下的客户端真实IP地址,在真实Web节点上配置，配置信息如下

### 5.1 配置

>**server nginx配置**

```
[root@node04 nginx]# egrep -v '#|^$' nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;
    set_real_ip_from node01;
    set_real_ip_from node02;
    set_real_ip_from node03;
    real_ip_header   X-Forwarded-For;
    real_ip_recursive on;
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;
        location / {
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
```

> 添加

```
set_real_ip_from node01;
set_real_ip_from node02;
set_real_ip_from node03;
real_ip_header   X-Forwarded-For;
real_ip_recursive on;
```

### 5.2 测试

- web浏览器访问测试,查看各个日志

```
[root@node01 conf.d]# tail -f /var/log/nginx/access.log
192.168.71.1 - - [13/Mar/2020:10:57:25 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko Core/1.70.3741.400 QQBrowser/10.5.3863.400" "-"

[root@node02 conf.d]# tail -f /var/log/nginx/access.log
192.168.71.133 - - [12/Mar/2020:13:12:11 +0800] "GET / HTTP/1.0" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko Core/1.70.3741.400 QQBrowser/10.5.3863.400" "192.168.71.1"

[root@node03 conf.d]# tail -f /var/log/nginx/access.log
192.168.71.134 - - [12/Mar/2020:13:12:03 +0800] "GET / HTTP/1.0" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko Core/1.70.3741.400 QQBrowser/10.5.3863.400" "192.168.71.1, 192.168.71.133"

[root@node04 conf.d]# tail -f /var/log/nginx/access.log
192.168.71.1 - - [13/Mar/2020:10:57:25 +0800] "GET / HTTP/1.0" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko Core/1.70.3741.400 QQBrowser/10.5.3863.400" "192.168.71.1, 192.168.71.133, 192.168.71.134"
```

> 最后server端的地址获取到的是`192.168.71.1`真实访问地址

- 命令行测试



```
[root@node02 ~]# curl node01
<h1>hello caimengzhi</h1>
```

再次查看日志

```
[root@node01 conf.d]# tail -f /var/log/nginx/access.log
192.168.71.134 - - [13/Mar/2020:10:58:11 +0800] "GET / HTTP/1.1" 200 26 "-" "curl/7.29.0" "-"


[root@node02 conf.d]# tail -f /var/log/nginx/access.log
192.168.71.133 - - [12/Mar/2020:13:13:01 +0800] "GET / HTTP/1.0" 200 26 "-" "curl/7.29.0" "192.168.71.134"

[root@node03 conf.d]# tail -f /var/log/nginx/access.log
192.168.71.134 - - [12/Mar/2020:13:12:53 +0800] "GET / HTTP/1.0" 200 26 "-" "curl/7.29.0" "192.168.71.134, 192.168.71.133"

[root@node04 conf.d]# tail -f /var/log/nginx/access.log
192.168.71.134 - - [13/Mar/2020:10:58:11 +0800] "GET / HTTP/1.0" 200 26 "-" "curl/7.29.0" "192.168.71.134, 192.168.71.133, 192.168.71.134"
```



> 最后server端的地址获取到的是`192.168.71.134`真实访问地址
>
> nginx realip：程序无需改动，直接使用remote_addr变量即可获取真实IP地址，但需要知道所有沿途经过的IP地址或IP段

此时我在做测试，让node03的前一级代理也获取到客户端真实的IP，

```
[root@node03 conf.d]# cat test.conf
server {
    listen 80;
    #server_name ip.test.com;

    location / {
        proxy_pass http://node04;
        set_real_ip_from node01;
        set_real_ip_from node02;
        real_ip_header   X-Forwarded-For;
        real_ip_recursive on;
    }
}
```

重启nginx。我再次命令行测试

```
[root@node02 ~]# curl node01
<h1>hello caimengzhi</h1>
```

我们重点查看node03和node04

```
[root@node03 conf.d]# tail -1 /var/log/nginx/access.log
192.168.71.134 - - [12/Mar/2020:13:35:09 +0800] "GET / HTTP/1.0" 200 26 "-" "curl/7.29.0" "192.168.71.134, 192.168.71.133"

[root@node04 nginx]# tail -1 /var/log/nginx/access.log
192.168.71.134 - - [13/Mar/2020:11:18:45 +0800] "GET / HTTP/1.0" 200 26 "-" "curl/7.29.0" "192.168.71.134, 192.168.71.133"
```

> 此时都可以拿到最初客户端的访问IP地址了。 只想要拿到真实客户端地址的话，主要配置如同上面node03/node04那样set_real_ip_from等几个参数即可