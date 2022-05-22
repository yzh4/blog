## 1. 作用

​    location指令的作用是可以根据用户请求的URI来执行不同的应用，其实就是根据用户请求的 网站地址URL匹配，匹配成功即进行相关的操作。

## 2. 语法

location使用的语法例子：

```
location [=|~|~*|^~] uri{
            …
}
```

> 解释

```
location    [=|~|~*|^~|@]   uri                {…}
指令      匹配标识       匹配的网站网址  匹配URI后要执行的配置段
```

​    上述语法中的URI部分是关键，这个URI可以是普通的字符串地址路径或者是正则表达式，当匹配成功则执行后面大括号里面的相关指令。正则表达式的签名还可以有^或~*等特殊的字符。 这两种特殊字符或*匹配的区别为：

- `=` 开头表示精确匹配
- `^~` 开头表示`uri`以某个常规字符串开头，理解为匹配 url路径即可。nginx不对url做编码，因此请求为`/static/20%/aa`，可以被规则`^~ /static/ /aa`匹配到（注意是空格）。
- `~` 开头表示区分大小写的正则匹配
- `~*` 开头表示不区分大小写的正则匹配
- `!~`和`!~*`分别为区分大小写不匹配及不区分大小写不匹配 的正则
- `/` 通用匹配，任何请求都会匹配到。

## 3. 匹配实例

下面是一组典型的location匹配，是官方的例子

```
location = / {
        [configuration A]
}

location / {
        [configuration B]
}

location /documents/ {
        [configuration C]
}

location ^~ /images/ {
        [configuration D]
}

location ~*\.(gif|jpg|jpeg)${
        [configuration E]
}
```

在上述location配置中，用户请求对应匹配如下表：

| 用户请求的URI         | 完整的URL地址                               | 匹配的配置      |
| :-------------------- | :------------------------------------------ | :-------------- |
| /                     | http://www.linux.ac.cn/                     | configuration A |
| /index.html           | http://www.linux.ac.cn/                     | configuration B |
| /documents/index.html | http://www.linux.ac.cn/documents/index.html | configuration C |
| /images/1.gif         | http://www.linux.ac.cn/images/1.gif         | configuration D |
| /documents/1.jpg      | http://www.linux.ac.cn/documents/1.jpg      | configuration E |

## 4. location实战

Nginx配置文件内容如下：

```
server {
    listen       90;
    server_name  www.linux.cmz.cn;

    location / {
        return 401;
    }

    location =/ {
          return 402;
    }

    location /documents/ {
          return 403;
    }

    location ^~/images/ {
        return 404;
    }

    location ~*\.(gif|jpg|jpeg)$ {
        return 500;
    }
    access_log  logs/www_access.log main;
}
```

然后以linux客户端为例对上述location匹配进行真实测试，配置hosts文件如下。

```
root@file:~# tail -1 /etc/hosts
127.0.0.1 www.linux.cmz.cn
```

实验结果如下

```
root@file:~# curl www.linux.cmz.cn:90/ -I
HTTP/1.1 402 Payment Required
Server: caimengzhi
Date: Tue, 28 May 2019 03:22:05 GMT
Content-Type: text/html
Content-Length: 165
Connection: keep-alive

root@file:~# curl www.linux.cmz.cn:90/index -I
HTTP/1.1 401 Unauthorized
Server: caimengzhi
Date: Tue, 28 May 2019 03:22:08 GMT
Content-Type: text/html
Content-Length: 177
Connection: keep-alive

root@file:~# curl www.linux.cmz.cn:90/documents/1.png -I
HTTP/1.1 403 Forbidden
Server: caimengzhi
Date: Tue, 28 May 2019 03:22:25 GMT
Content-Type: text/html
Content-Length: 151
Connection: keep-alive
root@file:~# curl www.linux.cmz.cn:90/images/11.gif -I
HTTP/1.1 404 Not Found
Server: caimengzhi
Date: Tue, 28 May 2019 03:22:40 GMT
Content-Type: text/html
Content-Length: 151
Connection: keep-alive

root@file:~# curl www.linux.cmz.cn:90/11.gif -I
HTTP/1.1 500 Internal Server Error
Server: caimengzhi
Date: Tue, 28 May 2019 03:22:49 GMT
Content-Type: text/html
Content-Length: 175
Connection: close
```

用户请求说明：

![](nginx_location.assets/p2.png)

上述不用URI及特殊字符组合匹配的顺序说明

| 不用URI及特殊字符组合的匹配顺序 | 匹配说明                                 |
| :------------------------------ | :--------------------------------------- |
| 第一名：location =/ {           | 精确匹配/                                |
| 第二名：location ^~/images/ {   | 匹配常规字符串，不做正则匹配检查。       |
| 第三名：location ~*.(gif        | jpg                                      |
| 第四名：location /documents/ {  | 匹配常规字符串，如果有正则则优先匹配正则 |
| 第五名：location / { 所         | 有location都不能匹配的时候默认匹配它     |

