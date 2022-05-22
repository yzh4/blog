## Nginx部署HTTPS

 利用证书实现HTTPS访问Nginx服务，需要nginx使用ssl模块配置HTTPS支持，默认情况下ssl模块并未被安装，如果要使用该模块则需要在编译时指定–with-http_ssl_module参数，安装模块依赖于OpenSSL库和一些引用文件，这些文件并不在同一个软件包中，通常这个文件名类似libssl-dev。

 nginx的https协议需要ssl模块的支持，我们在编译nginx时使用--with-http_ssl_module参数加入SSL模块。还需要服务器私钥，服务器证书，如果是公司对外环境，这个证书需要购买第三方的权威证书，否则用户体验得不到保障

 这里仅仅在本地的虚拟机环境上使用并测试。

### 部署https实践

```
1.创建Nginx需要的证书文件
确保机器安装了openssl和openssl-devel，创建证书
yum install openssl openssl-devel -y 

2.确保nginx支持了ssl模块，查看nginx编译信息即可
nginx -V

3.模拟证书颁发机构CA创建证书
进入nginx安装目录，便于管理证书
[root@chaogelinux ~]# cd /opt/ngx112/
[root@chaogelinux ngx112]# mkdir key
[root@chaogelinux ngx112]# cd key/

# 生成私钥文件，利用字shell降低文件权限
[root@chaogelinux key]# (umask 077;openssl genrsa -out server1024.key 1024)
Generating RSA private key, 1024 bit long modulus
.++++++
...++++++
e is 65537 (0x10001)


# 自己签发证书，crt证书扩展名
[root@chaogelinux key]# openssl req -new -x509 -key server1024.key -out server.crt -days 365
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:BJ
Locality Name (eg, city) [Default City]:BJ
Organization Name (eg, company) [Default Company Ltd]:chaoge
Organizational Unit Name (eg, section) []:it
Common Name (eg, your name or your server's hostname) []:pythonav.cn
Email Address []:yc_uuu@163.com

# 向机构申请证书，我们这里生成的是证书请求文件，而不是直接生成证书了，运维发送该文件给机构，请求合法证书
[root@chaogelinux key]# openssl req -new -key server1024.key -out server.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:BJ
Locality Name (eg, city) [Default City]:BJ
Organization Name (eg, company) [Default Company Ltd]:chaoge
Organizational Unit Name (eg, section) []:it
Common Name (eg, your name or your server's hostname) []:pythonav.cn
Email Address []:yc_uuu@163.com

# 针对这个请求文件，做一个加密处理，告知办法机构，可以用这个密码解密，了解公司信息，也可以直接回车不写密码
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:


# 一般这个证书颁发，需要等到一周内的时间，因此我们直接使用本地的自己签发的证书即可，进行练习

4.配置Nginx，加载私钥，证书
修改nginx.conf，添加
include extra/443.conf;

# 创建https配置文件
vim 443.conf 写入

[root@web01 extra]# cat 443.conf

server {
        server_name _;
        listen 443 ssl;
        ssl_certificate /opt/nginx/key/server.crt;
        ssl_certificate_key /opt/nginx/key/server1024.key;
        charset utf-8;
    location / {

         root html;
         index index.html index.htm;
}
}
~


# 修改80端口虚拟主机，进行请求转发给443
}
server {
    listen 80;
    server_name www.chaoge.com;
    charset utf-8;
     rewrite ^(.*)$ https://$host$1 permanent;
    location / {
            root html;
            index index.html index.htm;
    }
    }
include extra/443.conf;
}


# 检测语法，重启nginx
[root@web01 extra]# nginx -t
nginx: the configuration file /opt/nginx-1.16.0//conf/nginx.conf syntax is ok
nginx: configuration file /opt/nginx-1.16.0//conf/nginx.conf test is successful
```

## 使用腾讯云获取证书

> https://pythonav.com/wiki/detail/3/40/

