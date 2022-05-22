## 1. 介绍

​    实现Nginx负载均衡的组件主要由两个，见下表

| Nginx http功能模块       | 模块说明                                                     |
| :----------------------- | :----------------------------------------------------------- |
| ngx_http_proxy_module    | proxy代理模块，用于把请求后抛给服务器节点或upstream服务器池。 |
| ngx_http_upstream_module | 负载均衡模块，可以实现网站的负载均衡功能及节点的健康检查。   |

官方介绍连接：http://nginx.org/en/docs/http/ngx_http_upstream_module.html

## 2. 环境准备

### 2.1 硬件准备

准备4台VM虚拟机（有物理服务器更佳），两台做负载均衡，两台做RS，如下表。

| HOSTNAME | IP        | 说明              |
| :------- | :-------- | :---------------- |
| lb01     | 10.0.0.7  | Nginx主负载均衡器 |
| lb02     | 10.0.0.8  | Nginx辅负载均衡器 |
| web01    | 10.0.0.9  | web01服务器       |
| web02    | 10.0.0.10 | web02服务器       |

### 2.2 安装Nginx软件

下面将在以上4台服务器上安装Nginx。完整的安装过程见上面的说明，这里只写出安装的命令部分。

- 安装依赖软件包命令集合。

```
yum install openssl-devel pcre-devel -y
```

- 安装Nginx软件包的命令集合

```
wget http://nginx.org/download/nginx-1.6.3.tar.gz
useradd nginx -M -s /sbin/nologin
tar xf nginx-1.6.3.tar.gz 
cd nginx-1.6.3
./configure --prefix=/app/nginx-1.6.3 --user=nginx --group=nginx  --with-http_ssl_module --with-http_stub_status_module  --with-pcre
make && make install
ln -s /app/nginx-1.6.3/ /app/nginx
```

### 2.3 Nginx upstream

lb01和lb02的nginx配置文件如下：

```
worker_processes  1;
pid        logs/nginx.pid;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
upstream server_pools {         #配置一个upstream标签，名称为server_pools
       server 172.16.1.7:80 weight=1;    #后端两台web服务器的ip地址和端口，weight=1表示权重都是1
       server 172.16.1.8:80 weight=1;
    }
 server {
        listen       80;
        server_name  www.caimengzhi.org;
        location / {
            proxy_pass http://server_pools;         #访问www.etiantian.org 就抛给上面的upstream server_pools
            proxy_set_header Host $host;            #把客户端的请求头也一起抛过去
proxy_set_header X-Forwarded-For $remote_addr;  #把客户端的IP地址也抛过去，web服务器端记录客户端真实IP地址
                    }
}


server {
        listen       80;
        server_name  blog.caimengzhi.org;
        location / {
            proxy_pass http://server_pools;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $remote_addr;
                  }
     }
}
```

通过上面的配置，我们就实现了负载均衡了。

## 3. keepalived

在lb01和lb02负载均衡服务器上安装keepalived，安装命令如下：

```
yum install keepalived -y
```

> keepalived的默认配置文件路径在/etc/keepalived/keepalived.conf

### 3.1 主负载均衡配置

```
! Configuration File for keepalived

global_defs {
   notification_email {
     610658552@qq.com           #管理员邮箱
   }
   notification_email_from 610658552@qq.com   #管理员邮箱
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL          #这里是keepalived的ID，不同服务器的id不能相同
}

vrrp_instance VI_1 {       #实例
    state MASTER           # MASTER主的意思，高可用里有一主一备
    interface eth0         #绑定IP的网卡
    virtual_router_id 51   #两端的route_id 要一致
    priority 150           #优先级数值越高越优先，主的一定要被备的大
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.3/24 dev eth0 label eth0:1    #要配置的虚拟IP地址
    }
}
```

### 3.2 备负载均衡配置

```
! Configuration File for keepalived

global_defs {
   notification_email {
     610658552@qq.com     #管理员邮箱
   }
   notification_email_from 610658552@qq.com  
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL1       #这里是keepalived的ID，不同服务器的id不能相同
}

vrrp_instance VI_1 {       #实例
    state BACKUP           # 状态是备用，高可用里有一主一备
    interface eth0         #绑定IP的网卡
    virtual_router_id 51   #两端的route_id 要一致
    priority 100           #优先级数值越高越优先，主的一定要被备的大
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.3/24 dev eth0 label eth0:1    #要配置的虚拟IP地址
    }
}
```

启动keepalived，然后就能正常工作了，启动命令：

```
/etc/init.d/keepalived start
```

> 做IP解析的时候，只要解析到10.0.0.3这个IP即可