## 1. 介绍目录访问权限

​    标准情况下，软件默认的参数都是对安装软件的硬件标准来设置的，⽬前我们服务器的硬件资源远远⼤于要求的标准，所以为了让服务器性能更加出众，充分利⽤服 务器的硬件资源，我们⼀般需要优化APP的并发数来提升服务器的性能。

## 2. cpu

```
Nginx是主进程+⼯作进程模型
• worker_processes 1； ⼯作进程数量 按CPU的总核⼼调整
• worker_cpu_affinity 0010 0100 1000; CPU的亲和⼒
• worker_connections 1024； ⼀个⼯作进程的并发数
```

## 3. 长连接

http协议属于TCP协议，优化⽬标:减少三次握⼿和四次断开的次数

```
keepalive_timeout 5;     ⻓连接时间
keepalive_requests 8192; 每个⻓连接接受最⼤请求数
```

## 4. 数据压缩

```
gzip on; （启⽤ gzip 压缩功能）

gzip_proxied any; （nginx 做前端代理时启⽤该选项，表示⽆论后端服务器的headers头返回什么信息，都⽆条件启⽤压缩）

gzip_min_length 1024; （最⼩压缩的⻚⾯，如果⻚⾯过于⼩，可能会越压越⼤，这⾥规定⼤于1K的⻚⾯才启⽤压缩）

gzip_buffers 4 8k; （设置系统获取⼏个单位的缓存⽤于存储gzip的压缩结果数据流 按照原始数据⼤⼩以8K为单位申请4倍内存空间）

gzip_comp_level 3; （压缩级别，1压缩⽐最⼩处理速度最快，9压缩⽐最⼤但处理最慢，同时也最消耗CPU,⼀般设置为3就可以了）

gzip_types text/plain text/css application/x-javascript application/javascript application/xml; （什么类型的⻚⾯或⽂档启⽤压缩）
```

## 5. 客户端缓存

```
语法： expires [time|epoch|max|off]
默认值： expires off
作⽤域： http, server, location
location ~.*\.(js|css)?$
{
    expires 1h;
}
```

## 6. 参考

http://www.nginx.cn/doc/