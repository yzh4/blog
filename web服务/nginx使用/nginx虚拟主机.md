## 1. 介绍

以下我会配置三个虚拟主机来讲解。

```
[root@master nginx]# ls
client_body_temp  conf  fastcgi_temp  html  logs  proxy_temp  sbin  scgi_temp  uwsgi_temp
[root@master nginx]# mkdir html/{www,blog,bbs}
[root@master nginx]# for n in www blog bbs;do echo "$n.yzh.org" >html/$n/index.html;done
[root@master nginx]# ll html/
total 8
-rw-r--r-- 1 root root 537 Mar 13 13:27 50x.html
drwxr-xr-x 2 root root  24 Mar 13 14:22 bbs
drwxr-xr-x 2 root root  24 Mar 13 14:22 blog
-rw-r--r-- 1 root root 612 Mar 13 13:27 index.html
drwxr-xr-x 2 root root  24 Mar 13 14:22 www
[root@master nginx]# for n in www blog bbs;do  cat html/$n/index.html;done
www.yzh.org
blog.yzh.org
bbs.yzh.org
[root@master nginx]# cd conf/
[root@master conf]# egrep -v "#|^$" nginx.conf >a.log
[root@master conf]# cat a.log
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
[root@master conf]# >nginx.conf
[root@master conf]# cat nginx.conf
cat nginx.conf
worker_processes  4;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.yzh.org;
        root html/www;
       index  index.html index.htmi index.php;
    }
    server {
        listen       80;
        server_name  bbs.yzh.org;
        root html/bbs;
       index  index.html index.htmi index.php;
    }
    server {
        listen       80;
        server_name  blog.yzh.org;
        root html/blog;
       index  index.html index.htmi index.php;
    }
}
```

以上配置了三台虚拟主机。检查配置文件。没有问题后重启nginx

```
[root@master conf]# /app/nginx/sbin/nginx -t
nginx: the configuration file /app/nginx-1.14.2/conf/nginx.conf syntax is ok
nginx: configuration file /app/nginx-1.14.2/conf/nginx.conf test is successful
```

> 配置虚拟主机的流程

```
1、复制一个完整的serber标签段，注意要放在http的结束大括号前面。
2、更改server_name及对应的网页root根目录
3、检查配置文件语法，平滑重启服务
4、创建sever_name对应的网页的更目录，并且创建测试文件，若是没有index首页会出现403.
5、在客户端的对sever_name的主机名做host解析或者DNS配置，并检查（ping域名看返回的IP对不对）
6、Win32浏览器访问，或者在linux的客户端的host解析，用wget或curl访问
```

测试

```
[root@master conf]# egrep yzh /etc/hosts
127.0.0.1 www.yzh.org
127.0.0.1 bbs.yzh.org
127.0.0.1 blog.yzh.org

[root@master conf]# curl  www.yzh.org
www.yzh.org
[root@master conf]# curl  bbs.yzh.org
bbs.yzh.org
[root@master conf]# curl  blog.yzh.org
blog.yzh.org
```

## 2. nginx状态信息虚拟主机

```
[root@master conf]# !e
egrep caimengzhi /etc/hosts
127.0.0.1 www.caimengzhi.org
127.0.0.1 bbs.caimengzhi.org
127.0.0.1 blog.caimengzhi.org
127.0.0.1 status.caimengzhi.org

[root@master conf]# cat nginx.conf
worker_processes  4;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.caimengzhi.org;
        root html/www;
       index  index.html index.htmi index.php;
    }
    server {
        listen       80;
        server_name  bbs.caimengzhi.org;
        root html/bbs;
       index  index.html index.htmi index.php;
    }
    server {
        listen       80;
        server_name  blog.caimengzhi.org;
        root html/blog;
       index  index.html index.htmi index.php;
    }
    server {
        listen       80;
        server_name  status.caimengzhi.org;
        location / {
           stub_status on;
           access_log off;
        }
    }
}

[root@master conf]# /app/nginx/sbin/nginx -t
nginx: the configuration file /app/nginx-1.14.2/conf/nginx.conf syntax is ok
nginx: configuration file /app/nginx-1.14.2/conf/nginx.conf test is successful
[root@master conf]# /app/nginx/sbin/nginx -s reload

[root@master conf]# curl status.caimengzhi.org
Active connections: 1
server accepts handled requests
 7 7 7
Reading: 0 Writing: 1 Waiting: 0
```

解释

```
Active connections: 1 #nginx 整在处理的活动连接数是1个
server accepts handled requests 
 7 7 7 

第一个server  表示nginx启动到现在一共处理了7个连接数
第二个accept  表示nginx启动到现在一共创建了7个连接数
请求丢失数=（握手数 - 连接数），可以看出，本次状态显示没有丢失请求。
第三个accepts 表示一共处理了7次请求

一般server  accept  两个个要相等，否则请求必有丢失
Reading: 0 Writing: 1 Waiting: 9 
#Reading ：nginx 读取到客户端的Header的信息数
#Writing  ：nginx 返回给客户端的Heaer的信息数
#Waiting  ：已经处理完正在等候下一次请求指令的驻留连接，开启keep_alive的情况下，
           这个值等于active -(reading + writing)
```

## 3. 虚拟主机别名

```
server_name  www.caimengzhi.org caimengzhi.org;
```

就是在server_name后面的域名 空格后在加上一个域名，访问后面新增加的域名就会跳转到前面的域名

```
[root@master conf]# cat nginx.conf
worker_processes  4;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.caimengzhi.org caimengzhi.org;
        root html/www;
       index  index.html index.htmi index.php;
    }
    server {
        listen       80;
        server_name  bbs.caimengzhi.org;
        root html/bbs;
       index  index.html index.htmi index.php;
    }
    server {
        listen       80;
        server_name  blog.caimengzhi.org;
        root html/blog;
       index  index.html index.htmi index.php;
    }
    server {
        listen       80;
        server_name  status.caimengzhi.org;
        location / {
           stub_status on;
           access_log off;
        }
    }
}

[root@master conf]# /app/nginx/sbin/nginx -t
nginx: the configuration file /app/nginx-1.14.2/conf/nginx.conf syntax is ok
nginx: configuration file /app/nginx-1.14.2/conf/nginx.conf test is successful
[root@master conf]# /app/nginx/sbin/nginx -s reload
[root@master conf]# egrep caimengzhi /etc/hosts
127.0.0.1 www.caimengzhi.org caimengzhi.org
127.0.0.1 bbs.caimengzhi.org
127.0.0.1 blog.caimengzhi.org
127.0.0.1 status.caimengzhi.org
[root@master conf]# curl caimengzhi.org
www.caimengzhi.org
```

> apache

```
而apache指定别名的是：
Apache别名
SeverAlias 后面添加域名
```

## 4. 301 配置

```
[root@master conf]# vim nginx.conf
worker_processes  4;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.caimengzhi.org caimengzhi.org;
        root html/www;
        index  index.html index.htmi index.php;
        rewrite ^/(.*) http://bbs.caimengzhi.org/$1 permanent;
    }
    server {
        listen       80;
        server_name  bbs.caimengzhi.org;
        root html/bbs;
        index  index.html index.htmi index.php;
    }
    server {
        listen       80;
        server_name  blog.caimengzhi.org;
        root html/blog;
        index  index.html index.htmi index.php;
    }
    server {
        listen       80;
        server_name  status.caimengzhi.org;
        location / {
           stub_status on;
           access_log off;
        }
    }
"nginx.conf" 37L, 918C written
[root@master conf]# /app/nginx/sbin/nginx -s reload
[root@master conf]# /app/nginx/sbin/nginx -t
nginx: the configuration file /app/nginx-1.14.2/conf/nginx.conf syntax is ok
nginx: configuration file /app/nginx-1.14.2/conf/nginx.conf test is successful
[root@master conf]# curl caimengzhi.org
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.14.2</center>
</body>
</html>

[root@master conf]# elinks www.caimengzhi.org
                                           http://bbs.caimengzhi.org/
   bbs.caimengzhi.org
```

> 访问跟后的内容也就是^/(.*)，$1 接受所有,以上说明已经跳转成功。

!>注意

```
1、别名 地址还是caimengzhi.org  ==> bbs.caimengzhi.org的内容
2、caimengzhi.org跳转地址栏bbs.caimengzhi.org(rewrite)
3、Ip 访问，访问的是第一个标签内容
4、解决恶意域名绑定
```

## 5. 禁止IP访问

```
[root@master conf]# cat nginx.conf
worker_processes  4;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen 80;
        location / {
            deny all;
        }
    }
    server {
        listen       80;
        server_name  www.caimengzhi.org caimengzhi.org;
        root html/www;
        index  index.html index.htmi index.php;
        rewrite ^/(.*) http://bbs.caimengzhi.org/$1 permanent;
    }
    server {
        listen       80;
        server_name  bbs.caimengzhi.org;
        root html/bbs;
        index  index.html index.htmi index.php;
    }
    server {
        listen       80;
        server_name  blog.caimengzhi.org;
        root html/blog;
        index  index.html index.htmi index.php;
    }
    server {
        listen       80;
        server_name  status.caimengzhi.org;
        location / {
           stub_status on;
           access_log off;
        }
    }
}

[root@master conf]# /app/nginx/sbin/nginx -t
nginx: the configuration file /app/nginx-1.14.2/conf/nginx.conf syntax is ok
nginx: configuration file /app/nginx-1.14.2/conf/nginx.conf test is successful
[root@master conf]# /app/nginx/sbin/nginx -s reload
[root@master conf]# curl www.caimengzhi.org
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.14.2</center>
</body>
</html>
[root@master conf]# curl bbs.caimengzhi.org
bbs.caimengzhi.org
[root@master conf]# curl blog.caimengzhi.org
blog.caimengzhi.org
[root@master conf]# curl 127.0.0.1
<html>
<head><title>403 Forbidden</title></head>
<body bgcolor="white">
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.14.2</center>
</body>
</html>
```

> nginx的IP访问，默认是走第一个server。第一个server是deny的。所以就禁止了ip访问，只能通过域名了。

## 6. 虚拟主机分类

多个虚拟主机写在一个nginx.conf文件中，不方便维护也比较麻烦。类似apache的也可以分类虚拟主机。

```
[root@master conf]# cp nginx.conf nginx.conf.ori
[root@master conf]# mkdir extra
[root@master conf]# cat nginx.conf
error_log  logs/error.log error;
worker_processes  4;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    include extra/www.conf;
    include extra/bbs.conf;
    include extra/blog.conf;
    #或者直接包含所有
    # include extra/*.conf; #表示包含extra所有的.conf文件
}

[root@master conf]# ls extra/
bbs.conf  blog.conf  www.conf
[root@master conf]# cat extra/www.conf
server {
    listen       80;
    server_name  www.caimengzhi.org;
    root         html/www;
    index        index.html index.htmi index.php;
    access_log   logs/www_access.log;
}
[root@master conf]# cat extra/bbs.conf
server {
    listen       80;
    server_name  bbs.caimengzhi.org;
    root         html/bbs;
    index        index.html index.htmi index.php;
    access_log   logs/bbs_access.log;
}
[root@master conf]# cat extra/blog.conf
server {
    listen       80;
    server_name  blog.caimengzhi.org;
    root         html/blog;
    index        index.html index.htmi index.php;
    access_log   logs/blog_access.log;
}
[root@master conf]# /app/nginx/sbin/nginx -t
nginx: the configuration file /app/nginx-1.14.2/conf/nginx.conf syntax is ok
nginx: configuration file /app/nginx-1.14.2/conf/nginx.conf test is successful
[root@master conf]# /app/nginx/sbin/nginx -s reload
```

测试

```
[root@master conf]# curl www.caimengzhi.org
www.caimengzhi.org
[root@master conf]# curl bbs.caimengzhi.org
bbs.caimengzhi.org
[root@master conf]# curl blog.caimengzhi.org
blog.caimengzhi.org
```