## 1. 介绍目录访问权限

​    一般，我们对需要对目录是否有访问权限做限制。比如a目录下禁止访问

### 1.1 配置

```
location /a {
    allow 192.168.1.0/24;
    deny all;
    #return 404;
    return http://www.jd.com;
}
```

## 2. 认证登录

```
location /b {
    auth_basic "请输入账号和密码";
    auth_basic_user_file /etc/nginx/htpasswd;
}
```

## 3. 反向代理

```
location / {
    index index.php index.html index.htm; # 定义⾸⻚索引⽂件的名称
    proxy_pass http://mysvr;              # 请求转向mysvr 定义的服务器列表
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    client_max_body_size 10m;             # 允许客户端请求的最⼤单⽂件字节数
    client_body_buffer_size 128k;         # 缓冲区代理缓冲⽤户端请求的最⼤字节数，
    proxy_connect_timeout 90;             # nginx跟后端服务器连接超时时间(代理连接超时)
    proxy_send_timeout 90;                # 后端服务器数据回传时间(代理发送超时)
    proxy_read_timeout 90;                # 连接成功后，后端服务器响应时间(代理接收超时)
    proxy_buffer_size 4k;                 # 设置代理服务器（nginx）保存⽤户头信息的缓冲区⼤⼩
    proxy_buffers 4 32k;                  # proxy_buffers缓冲区，⽹⻚平均在32k以下的话，这样设置
    proxy_busy_buffers_size 64k;          # ⾼负荷下缓冲⼤⼩（proxy_buffers*2）
    proxy_temp_file_write_size 64k;       # 设定缓存⽂件夹⼤⼩，⼤于这个值，将从upstream服务器传
```

## 4. 限速

​    限速该特性可以限制某个⽤户在⼀个给定时间段内能够产⽣的HTTP请求数。请求可以简单到就是⼀个对于主⻚的GET请求或者⼀个登陆表格的POST请求。

​    限速也可以⽤于安全⽬的上，⽐如暴⼒密码破解攻击。通过限制进来的请求速率，并且（结合⽇志）标记出⽬标URLs来帮助防范DDoS攻击。⼀般地说，限流是⽤在 保护上游应⽤服务器不被在同⼀时刻的⼤量⽤户请求湮没。

应用常见

- DDOS防御
- 下载场景保护IO

```
Nginx官⽅版本限制IP的连接和并发分别有两个模块：
limit_req_zone ⽤来限制单位时间内的请求数，即速率限制
    Syntax: limit_req zone=name [burst=number] [nodelay];
    Default: —
    Context: http, server, locati
limit_req_conn ⽤来限制同⼀时间连接数，即并发限
```

### 4.1 案例1

基于IP对下载速率做限制 限制每秒处理1次请求，对突发超过5个以后的请求放⼊缓存区

```
http {
  limit_req_zone $binary_remote_addr zone=baism:10m rate=1r/s;
  server {
  location /search/ {
  limit_req zone=baism burst=5 nodelay;
}
```

> 参数设置

```
limit_req_zone $binary_remote_addr zone=baism:10m rate=1r/s;
第⼀个参数：$binary_remote_addr 表示通过remote_addr这个标识来做限制，“binary_”的⽬的是缩写内存占⽤量，是限制同⼀客户端ip地址。
第⼆个参数：zone=baism:10m表示⽣成⼀个⼤⼩为10M，名字为one的内存区域，⽤来存储访问的频次信息。
第三个参数：rate=1r/s表示允许相同标识的客户端的访问频次，这⾥限制的是每秒1次，还可以有⽐如30r/m的。limit_req zone=baism burst=5 nodelay;
第⼀个参数：zone=baism 设置使⽤哪个配置区域来做限制，与上⾯limit_req_zone ⾥的name对应。
第⼆个参数：burst=5，重点说明⼀下这个配置，burst爆发的意思，这个配置的意思是设置⼀个⼤⼩为5的缓冲区当有⼤量请求（爆发）过来时，超过了访问频次限制的请
求可以先放到这个缓冲区内。
第三个参数：nodelay，如果设置，超过访问频次⽽且缓冲区也满了的时候就会直接返回503，如果没有设置，则所有请求会等待排
```

### 4.2 案例2

基于IP做连接限制 限制同⼀IP并发为1 下载速度为100K

```
limit_conn_zone $binary_remote_addr zone=addr:10m;
server {
  listen 80;
  server_name localhost;
  location / {
      root html;
      index index.html index.htm;
  }

  location /abc {
      limit_conn addr 1;
      limit_rate 100k;
  }
}

http {
  include mime.types;
  default_type application/octet-stream;
  sendfile on;
  keepalive_timeout 65;
  # 基于IP做连接限制 限制同⼀IP并发为1 下载速度为100K
  limit_conn_zone $binary_remote_addr zone=addr:10m;
  # 基于IP对下载速率做限制 限制每秒处理1次请求，对突发超过5个以后的请求放⼊缓存区
  limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
  server {
      listen 80;
      server_name localhost;
  location / {
      root html;
      index index.html index.htm;
  }
  location /abc {
      limit_req zone=one burst=5 nodelay;
      limit_conn addr 1;
      limit_rate 100k;
  }
}
```

## 5. URL 重写

rewrite模块（ngx_http_rewrite_module）

​    Rewrite功功能是Nginx服务器提供的⼀个重要功能。⼏乎是所有的web产品必备技能，⽤于实现URL重写。URL重写是⾮常有⽤的功能，⽐如它可以在 我们在改变⽹站结构后，不需要客户端修改原来的书签，也不需要其他⽹站修改对我们⽹站的友情链接，还可以在⼀定程度上提⾼⽹站的安全性，能够 让我们的⽹站显得更专业。

​    Nginx服务器Rewrite功能的实现是依赖于PCRE（Perl Compatible Regular Expression。Perl兼容的正则表达式）的⽀持，所以在编译安装Nginx之前， 需要安装PCRE

应用场景

- 域名变更 （京东）
- ⽤户跳转 （从某个连接跳到另⼀个连接）
- 伪静态场景 （便于CDN缓存动态⻚⾯数

URL rewrite 语法

```
rewrite  <regex> <replacement> [flag];
关键字   正则    替代内容      flag标记

flag:
    last      # 本条规则匹配完成后，继续向下匹配新的location URI规则
    break     # 本条规则匹配完成即终⽌，不再匹配后⾯的任何规则
    redirect  # 返回302临时重定向，浏览器地址会显示跳转后的URL地址
    permanent # 返回301永久重定向，浏览器地址栏会显示跳转后的URL地址
set指令 ⾃定义变量
Syntax:
set $variable value;
Default:
—
Context:
server, location, 

1) set 设置变量
2) if 负责语句中的判断
3) return 返回返回值或URL
4) break 终⽌后续的rewrite规则
5) rewrite 重定向U
```

### 5.1 案例1

将`http://www.caimengzhi.com `重写为 `http://www.caimengzhi.com/blog`

```
location / {
  set $name blog;
  rewrite ^(.*)$ http://www.caimengzhi.com/$name;
}
```

### 5.2 案例2

```
if 指令 负责判断
Syntax:
if (condition) { ... }
Default:
—Context:
server, location
 location / {
  root html;
  index index.html index.htm;
  if ($http_user_agent ~* 'Chrome') {
  return 403;
  #return http://www.jd.com;
  }
```

### 5.3 案例3

```
return 指令 定义返回数据
Syntax: return code [text];
return code URL;
return URL;
Default: —Context: server, location, if
location / {
  root html;
  index index.html index.htm;
  if ($http_user_agent ~* 'Chrome') {
  return 403;
  #return http://www.jd.com;
}
```

### 5.4 案例4

```
break 指令 停⽌执⾏当前虚拟主机的后续rewrite指令集
Syntax: break;
Default:—
Context:server, location, if
location / {
  root html;
  index index.html index.htm;
  if ($http_user_agent ~* 'Chrome') {
      break;
      return 403;
}
```

### 5.5 案例5

域名跳转`www.caimengzhi.com` 重写为` www.jd.com`

```
server {
  listen 80;
  server_name www.caimengzhi.com;
  location / {
    rewrite ^/$ http://www.jd.com permanent;
  }
```

!>注意

```
注意:
重定向就是将⽹⻚⾃动转向重定向
301永久性重定向：新⽹址完全继承旧⽹址，旧⽹址的排名等完全清零
  301重定向是⽹⻚更改地址后对搜索引擎友好的最好⽅法，只要不是暂时搬移的情况，都建议使⽤301来做转址。

302临时性重定向：对旧⽹址没有影响，但新⽹址不会有排名
  搜索引擎会抓取新的内容⽽保留旧的
```

break,本条规则匹配完成即终⽌，不再匹配后⾯的任何规则,类似临时重定向，返回客户端3

### 5.6 案例6

根据⽤户浏览器重写访问⽬录，如果是chrome浏览器 就将 `http://192.168.1.30/`\(URI 重写为` http://192.168.1.30/chrome/\)UR`

```
location / {
    .....
    if ($http_user_agent ~* 'chrome') {
    rewrite ^(.*)$ /chrome/$1 last;
}
location /chrome {
    root html ;
    index index.html;
}
```

> 解释

```
^ 以什么开头 ^a
$ 以什么结尾 c$
. 除了回⻋以外的任意⼀个字符
* 前⾯的字符可以出现多次或者不出现
更多内容看正则表达式 re
```