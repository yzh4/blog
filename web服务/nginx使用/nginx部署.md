## 1. 介绍

​    Nginx(Engine x),俄罗斯人开发阿德，开源的WWW服务软件。一共才780K，而apache大概7M左右。

​    Nginx本身是一款静态（html，css，js，jpg等）www软件 静态小文件高并发量，同时占用的资源很少，3W并发量 10个线程150w。

​    Nginx使用平台：unix linux,windows都可以。

- 使用排名 http://w3techs.com/technologies/overview/web_server/all
- 使用趋势 http://news.netcraft.com/

国内网站使用nginx更多一些

### 1.1 Nginx功能

- www web 服务 http协议 端口80
- 负载均衡（反向代理proxy）
- Web cache（web缓存）

### 1.2 Nginx优点

- 高并发（静态小文件），静态1-2W
- 占用资源少，2W并发 开10个线程服务，内存消耗几百M。
- 功能种类比较多（web，cache，proxy），但是每一个功能都不是很强。
- 支持epoll模型。使得nginx可以支持高并发。
- ginx配合动态服务和apache有区别。
- 利用nginx可以对ip限速，可以限制连接数（有这样的模块，apache也有只是第三方的）。
- 配置更简单，更灵活。

### 1.3 使用场景

- 静态服务器（图片，视频服务）html，js，css，flv等，国内使用的静态服务器就两个，一个是nginx一个是lightttpd。
- 静态的访问高。并发1-3W。
- 动态服务。Nginx+fastcgi的方式运行php，jsp。并发500-1500. 相对于apache+php,lighttp+fcgi php
- 反向代理，负载均衡。日PV2000W以下，都可以直接使用nginx代理.haproxy,F5,a10
- 缓存服务.Squid,varnish

### 1.4 对比

#### 1.4.1 Apache

- 2.2版本非常强大稳定，据官方说2.4本版本性能超强
- Preforx模式取消了进程创建开销，性能很高
- 处理动态业务数据时，因关联到后端的引擎和数据库，瓶颈不在Apache本身
- 高并发时候消耗的系统资源相对多一些
- 基于传统的select的模型
- 扩展库，DSO方法，apxs。
- 功能多更稳定，更安全，插件也多

#### 1.4.2 nginx

- 基于异步IO模型(epoll.kqueue)，性能强，支持上万并发量
- 对小文件支持很好，性能很高（限小文件）
- 代码优美，扩展库必须编译主进程程序
- 消耗系统资源相对比较低

#### 1.4.3 Lighttpd

- 基于异步IO模型，性能和nginx相近
- 扩展库是SO模式，比nginx更灵活
- 全球使用率比较低，安全性能没有上面两个好
- 通过插件（mod_secdowload）可实现文件URL地址加密。

### 1.5 web服务建议

- 静态业务：高并发，采用nginx后者lighttpd根据自己的掌握程度或公司要求
- 动态业务：采用nignx或者apache
- 既有静态业务也有动态业务：nginx或者apache，不要多选，要单选。
- 动态业务可以由前端的代理（haproxy）,根据页面元素类型，向后转发相应的服务器进行处理 若是并发不是很大，又对apache很熟，采用apache也是可以的，apache2.4版本也很强大，并发连接数也有所增加，
- 提示：nginx做web（apache，lighttpd），反向代理（haproxy，lvs nat）以及缓存服务（squid）

### 1.6 nginx虚拟主机

- 基于域名，通过域名来区分虚拟主机。 ==>应用：外部网站
- 基于端口，通过端口来区分虚拟主机。 ==>应用：公司内部网站。
- 基于IP（不完善），基本不用。

## 2. nginx部署

### 2.1 安装pcre

​    PCRE 作用是让 Nginx 支持 Rewrite 功能。

首先要检查机器是否已经安装pcre

```
[root@docker01 ~]# pcre-config --version
8.32
```

若是没有安装就安装一下步骤安装，缺少什么依赖就安装依赖。

```
cd /usr/local/src/
wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
tar zxvf pcre-8.35.tar.gz
cd pcre-8.35
./configure && make && make install
pcre-config --version
yum -y install pcre-devel openssl openssl-devel
```

### 2.2 下载安装nginx

以下我以centos7基础环境。

#### 2.2.1 在线安装



```
centos 
yum install nginx

ubuntu
apt-get install nginx
```

由于在线安装nginx，可能不是最新的[依赖于系统的nginx镜像远版本]。



#### 2.2.2 离线安装

现在目前最新最稳定的版本的1.14.2

```
useradd nginx -s /sbin/nologin -M
wget http://nginx.org/download/nginx-1.14.2.tar.gz
unzip nginx-1.14.2.zip
cd nginx-1.14.2
./configure \
--user=nginx \
--group=nginx \
--prefix=/app/nginx-1.14.2 \
--with-http_stub_status_module \
--with-http_ssl_module
make && make install
ln -s /app/nginx1.6.2/  /app/nginx 
```

操作过程

```
[root@master ~]# cd /usr/local/src/
[root@master src]# wget http://nginx.org/download/nginx-1.14.2.tar.gz
[root@master src]# useradd nginx -s /sbin/nologin -M
[root@master src]# ls
nginx-1.14.2.zip
[root@master src]# tar xf nginx-1.14.2.tar.gz
[root@master src]# cd nginx-1.14.2
[root@master nginx-1.14.2]# ./configure \
--user=nginx \
--group=nginx \
--prefix=/app/nginx-1.14.2 \
--with-http_stub_status_module \
--with-http_ssl_module
[root@master nginx-1.14.2]# ln -sf /app/nginx-1.14.2/ /app/nginx
[root@master nginx-1.14.2]# ls /app/
nginx  nginx-1.14.2
```



#### 2.2.3 启动并检查

```
[root@master nginx-1.14.2]# /app/nginx/sbin/nginx -t
nginx: the configuration file /app/nginx-1.14.2/conf/nginx.conf syntax is ok
nginx: configuration file /app/nginx-1.14.2/conf/nginx.conf test is successful
[root@master nginx-1.14.2]# /app/nginx/sbin/nginx
[root@master nginx-1.14.2]# ps axf|grep -v grep |grep nginx
 25298 ?        Ss     0:00 nginx: master process /app/nginx/sbin/nginx
 25299 ?        S      0:00  \_ nginx: worker process
[root@master nginx-1.14.2]# netstat -lnp|grep -w 80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      25298/nginx: master

[root@master nginx-1.14.2]# curl 127.0.0.1 -I
HTTP/1.1 200 OK
Server: nginx/1.14.2
Date: Wed, 13 Mar 2019 05:30:09 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Wed, 13 Mar 2019 05:27:38 GMT
Connection: keep-alive
ETag: "5c8894ca-264"
Accept-Ranges: bytes
```

> 注意

```
检查nginx配置文件: /app/nginx/sbin/nginx -t
启动nginx:         /app/nginx/sbin/nginx
重新加载配置文件:  /app/nginx/sbin/nginx -s reload

nginx不通的排错
1、ping ip  物理通不通
2、telnet ip 浏览器到web服务器通不通
3、服务器本地 curl  127.0.0.1 web服务开没开 
```

