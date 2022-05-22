## 1. 介绍

​    一般，我们做好防盗链之后其他网站盗链的本站图片就会全部失效无法显示，但是您如果通过浏览器直接输入图片地址，仍然会显示图片，仍然可以右键图片另存为下载文件！依然可以下载？这样就不是彻底的防盗了！那么，nginx应该怎么样彻底地实现真正意义上的防盗链呢？

## 2. 配置

```
[root@master extra]# cat www.conf
server {
    listen       80;
    server_name  www.caimengzhi.org;
    root         html/www;
    index        index.html index.htmi index.php;
    access_log   logs/www_access.log;
    location ~* \.(gif|jpg|png|jpeg)$ {
        expires     30d;
        valid_referers  *.caimengzhi.org www.caimengzhi.org m.caimengzhi.com *.baidu.com *.google.com;
        if ($invalid_referer) {
        rewrite ^/ http://ww4.sinaimg.cn/bmiddle/051bbed1gw1egjc4xl7srj20cm08aaa6.jpg;
        #return 404;
        }
    }
}
```

解释

```
第一行： location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    其中“gif|jpg|jpeg|png|bmp|swf”设置防盗链文件类型，自行修改，每个后缀用“|”符号分开！

第三行：valid_referers  *.caimengzhi.org www.caimengzhi.org m.caimengzhi.com *.baidu.com *.google.com;
    就是白名单，允许文件链出的域名白名单，自行修改成您的域名！*.caimengzhi.com这个指的是子域名，域名与域名之间使用空格隔开！

第五行：rewrite ^/ http://www.caimengzhi.com/static/images/404.jpg;
    这个图片是盗链返回的图片，也就是替换盗链网站所有盗链的图片。这个图片要放在没有设置防盗链的网站上，因为防盗链的作用，这个图片如果也放在防盗链网站上就会被当作防盗链显示不出来了，盗链者的网站所盗链图片会显示X符号。
```

这样您在浏览器直接输入图片地址就不会再显示图片出来了，也不可能会再右键另存什么的。

第五行：`rewrite ^/ http://www.it300.com/static/images/404.jpg`;这个是给图片防盗链设置的防盗链返回图片，如果我们是文件需要防盗链下载，把第五行：

```
rewrite ^/ http://www.caimengzhi.com/static/images/404.jpg;
```

改成一个链接，可以是您主站的链接，比如把第五行改成：

```
rewrite ^/ http://www.caimengzhi.com;
```

这样，当别人输入文件下载地址，由于防盗链下载的作用就会跳转到您设置的这个链接！最后，配置文件设置完成别忘记重启nginx生效！