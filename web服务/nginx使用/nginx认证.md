## 1. 介绍

 我们在使用`Nginx`的时候，希望需要密码验证，此时我们就可以使用`Nginx`的认证模块`ngx_http_auth_basic_module`，有的时候暴漏`Nginx`的web页面很不安全，需要添加一个认证，本文介绍使用`htpasswd`工具为`Nginx`添加认证的用户名和密码。

在 `Nginx`下，提供了 `ngx_http_auth_basic_module`模块实现让用户只有输入正确的用户名密码才允许访问web内容。默认情况下，`Nginx` 已经安装了该模块。所以整体的一个过程就是先用第三方工具设置用户名、密码（其中密码已经加过密），然后保存到文件中，接着在 `Nginx`配置文件中根据之前事先保存的文件开启访问验证。

```
https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html
```

 这个`ngx_http_auth_basic_module`模块允许通过使用“HTTP基本身份验证”协议验证用户名和密码来限制对资源的访问。

 访问也可能受到以下因素的限制：[地址](https://nginx.org/en/docs/http/ngx_http_access_module.html)，由[分请求结果](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html)，或通过[JWT](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html)。同时对地址和密码的访问限制由[满足感](https://nginx.org/en/docs/http/ngx_http_core_module.html#satisfy)指令。

demo配置

```
location / {
    auth_basic           "closed site";
    auth_basic_user_file conf/htpasswd;
}
```

## 2. 语法

语法指令**

| Syntax:  | `**auth_basic** *string* | off;`             |
| :------- | -------------------------------------------- |
| Default: | `auth_basic off;`                            |
| Context: | `http`, `server`, `location`, `limit_except` |

> 启用使用“HTTP基本身份验证”协议验证用户名和密码。指定的参数用作`*realm*`。参数值可以包含变量(1.3.10，1.2.7)。特殊价值`off`允许取消`auth_basic`指令继承自以前的配置级别。

| 语法： | `**auth_basic_user_file** *file*;`           |
| :----- | :------------------------------------------- |
| 默认： | —                                            |
| 背景： | `http`, `server`, `location`, `limit_except` |

> 指出了使用的范围。只能在`http`, `server`, `location`, `limit_except`

## 3. 密码

使用的时候要结和密码文件，以下是密码文件格式，指定以下列格式保存用户名和密码的文件：

```
# comment
name1:password1
name2:password2:comment
name3:password3
```

> 密码必须使用函数 crypt(3) 加密。你可以使用来自 Apache 的 `htpasswd`工具来创建密码文件

```
-c 创建 -b新建首次密码问。再次追加密码文件的是就去掉-b
htpasswd -cb htpasswd container container

htpasswd -cb htpasswd caimengzhi caimengzhi
```

> container container，左侧是账号，右侧是密码
>
> 没有`htpasswd`命令,可以安装apache软件。

## 4. 案例

```
设定密码
root@leco:/etc/nginx/sites-enabled# htpasswd -b ./htpasswd leco leco
Adding password for user leco

配置文件
root@leco:/etc/nginx/sites-enabled# cat book1.conf 
server {
    listen  10000;
    charset utf-8;
    location / {    
        auth_basic   "hello boy,we need username and password"; # 弹出页面提示
        auth_basic_user_file /etc/nginx/sites-enabled/htpasswd; # 密码文件
        root    /root/book/books/site;
        index   index.html;
    }
}

重启nginx
root@leco:/etc/nginx/sites-enabled# /etc/init.d/nginx restart
[ ok ] Restarting nginx (via systemctl): nginx.service.

测试没加账号和密码
root@leco:/etc/nginx/sites-enabled# curl 127.0.0.1:10000 -I
HTTP/1.1 401 Unauthorized
Server: nginx
Date: Wed, 15 May 2019 07:50:30 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 188
Connection: keep-alive
WWW-Authenticate: Basic realm="hello boy,we need username and password"

测试加账号和密码
root@leco:/etc/nginx/sites-enabled# curl -u leco:leco 127.0.0.1:10000 -I
HTTP/1.1 200 OK
Server: nginx
Date: Wed, 15 May 2019 07:50:52 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 119849
Last-Modified: Wed, 15 May 2019 06:30:23 GMT
Connection: keep-alive
ETag: "5cdbb1ff-1d429"
Accept-Ranges: bytes
```

> 测试OK。