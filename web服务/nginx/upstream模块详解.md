## upstream模块介绍

Nginx的负载均衡功能来自于其模块`ngx_http_upstream_module`模块，该模块支持的代理方式有：

- uwsgi_pass
- Fastcgi_pass
- proxy_pass
- Memcached_pass
- ...

`ngx_http_upstream_module`模块允许Nginx定义一组或多组节点服务器，使用时可以通过`proxy_pass`代理方式，把用户请求发送到事先定于好的`upstream组`中。具体写法就是

```
upstream www_pools {
server x.x.x.x;
server x.x.x.x;
}

proxy_pass http://www_pools;
```

### 完整的upstream配置案例

```
upstream www_pools {
server 192.168.178.121;
server 192.168.178.122:80 weight=1 max_fails=1 fail_timeout=10s;
server 192.168.178.123:80 weight=10 max_fails=2 fail_timeout=20s backup;
server 192.168.178.124:80 wetight=10 max_fails=2 fail_timeout=20s backup;
}
```

### 使用域名及socket的upstream配置

```
upstream backend {
server backend1.example.com weight=5;
server backend2.example.com:8080;
server unix:/tmp/backend3;
server backend3.example.com:8080 backup;
}
```

### upstream模块参数

```
# 参数解释
server是固定关键字，后面跟着服务器ip或是域名，默认是80端口，也可以指定端口
weight表示节点的权重，数字越大，分配的请求越多，注意nginx结尾的分号
max_fails Nginx尝试连接后端节点失败的次数，根据企业情况调整，默认是1
backup 其它所有的非backup机器down或者忙的时候，请求backup机器，实现热备效果。
fail_timeout 在max_fails定义的次数失败后，距离下次检查的间隔时间，默认10s
down 表示当前主机暂停，不参与负载均衡

upstream模块的内容应放于nginx.conf配置中的 http{}标签内
其默认调度算法是wrr(权重轮询，weighted round-robin)
```

## upstream模块调度算法

调度算法一般分几类：

- 第一类是静态调度算法：负载均衡器根据自身设定的规则进行分配，不需要考虑后端节点的健康情况。例如轮询、加权轮询、哈希类型调度算法。
- 第二类是动态调度算法，负载均衡器会判断后端节点的当前状态，来决定是否分发请求。例如链接数最少的优先分发，响应时间短的优先分发，如least_conn、fail等都是动态调度。

### rr轮询（round-robin）

按照请求顺序逐一分配给不同的后端节点服务器，如果后端节点宕机，宕机的服务器会被自动从地址池中剔除，新的请求会发给正常的服务器。

### wrr（权重轮询）

给后端节点服务器增加权重，数值越大，优先获得客户端请求，可以以服务器配置来决定比例大小，从而解决新旧服务器的性能不均衡问题等。

```
upstream backend {
server 192.168.178.122 weight=1;
server 192.168.178.121 weight=2;
}
```

### ip_hash

每个请求按客户端IP的hash结果分配，当新的请求到达，将其客户端IP通过哈希算法得到一个唯一值，在随后的客户端请求中，如果客户端的IP哈希值相等，该请求就会固定发给一台服务器。

该调度算法可以解决动态网页中的session共享问题。

注意了使用ip_hash不得再使用weight、backup两个参数，造成冲突了，即使写了也不生效。

```
upstream chaoge_backend {
ip_hash;
server 192.168.178.121;
server 192.168.178.122;

}
```

### fail

该算法根据后端服务器节点的响应时间来分配，响应时间短的优先分配，该算法根据页面大小和加载时间长短进行负载均衡，nginx本身不支持fail形式，如果要支持该算法，必须下载nginx的`upstream_fail`模块

```
upstream chaoge_backend {
fair;
server 192.168.178.121;
server 192.168.178.122;
}
```

### least_conn

该算法根据后端节点的链接数决定分配请求，哪个机器链接数少，就发给谁。

### url_hash

和ip_hash类似，该算法根据客户端请求的URL信息进行hash得到唯一值，让每个URL固定的发给同一个后端服务器，后端服务器为`缓存服务器`效果最佳。

Nginx本身是不支持url_hash的，需要单独安装hash模块

url_hash(web缓存节点)和ip_hash(会话保持)功能类似。

```
upstream chaoge_backend {
server squid1:3128;
server squid:3128;
hash $request_uri;
hash_method crc32;
}
```

## proxy_pass指令

proxy_pass指令属于`ngx_http_proxy_module`模块，此模块可以把请求转发到另一台服务器，在实际的反向代理工作中，会通过location功能指定的URL，然后把接收到的符合URL的请求通过proxy_pass参数抛给定义好的`upstream`地址池。

### proxy_pass案例

案例1

```
在nginx.conf配置文件中定义

location /name/ {
proxy_pass http://127.0.0.1/remote/;
}

例如当请求URL是： http://192.168.178.121/name  ，会进入该locaiton的作用域，通过参数proxy_pass请求转发给了http://127.0.0.1/remote/
```

案例2

```
  location ~ .*\.php$ {
        proxy_pass http://www.example.cn$request_uri;
        proxy_set_header Host $proxy_host;
        proxy_set_header X-Forwarded-For $remote_addr;
    }

所有请求以.php结尾的URL，进行转发
```

### proxy_pass参数

| 参数                    | 作用解释                                                     |
| ----------------------- | ------------------------------------------------------------ |
| proxy_set_header        | 设置反向代理向后端发送的http请求头信息，如添加host主机头部字段，让后端服务器能够获取到真实客户端的IP信息等 |
| client_body_buffer_size | 指定客户端请求主体缓冲区大小                                 |
| proxy_connect_timeout   | 反向代理和后端节点连接的超时时间，也是建立握手后等待响应的时间 |
| proxy_send_timeout      | 表示代理后端服务器的数据回传时间，在规定时间内后端若数据未传完，nginx会断开连接 |
| proxy_read_timeout      | 设置Nginx从代理服务器获取数据的超时时间                      |
| proxy_buffer            | 设置缓冲区的数量大小                                         |