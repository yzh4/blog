## docker_api

> docker api 网址：https://docs.docker.com/engine/api/v1.24/

API是程序开发的一个接口，docker api类型包括RESTful，基于HTTP协议和RPC协议。

所谓restful风格就是对HTTP协议的各种封装，如

- get
- post
- delete
- update
- put
- 等等

restfule服务器提供这些请求的接口(地址，路径，参数，也就是一个完整的url）

docker提供了三种API使用

- Registry，提供存储docker镜像的功能
- Docker Hub API：提供了docker hub集成的功能
- Docker Remote API：提供了和Docker守护进程集成的功能（也就是docker命令各种操作）

## Docker Remote API

核心介绍docker remote api，因为这是核心docker命令操作。

docker默认的守护进程会绑定在本地主机的套接字

```
[root@docker01 ~]# cat /usr/lib/systemd/system/docker.service |grep sock                       
Requires=docker.**sock**et containerd.service                                                      
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.**sock** 
```

docker的守护进程需要root身份去运行，否则会导致权限不足，管理资源。

```
[root@docker01 ~]# ps -ef|grep docker
root      66459      1  0 14:05 ?        00:00:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

如果只查询宿主机本地的docker API，那么没什么问题，如果想要远程访问，本地sock套接字文件就不行了，得把docker守护进程绑定到网络接口上。

## remote API操作

```
1.修改Remote API配置信息，改为网络接口形式
[root@docker01 ~]# cat /usr/lib/systemd/system/docker.service |grep dockerd
#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375

2.重启服务
[root@docker01 ~]# systemctl daemon-reload
[root@docker01 ~]# systemctl restart docker
[root@docker01 ~]# ps -ef|grep docker
root      64320      1  0 03:41 ?        00:00:00 /usr/bin/dockerd -H tcp://0.0.0.0:2375
root      64507  63607  0 03:42 pts/0    00:00:00 grep --color=auto docker

3.通过远程主机，查看是否能够连接docker守护进程。
# 远程查看镜像
[root@sql ~]# docker  -H 192.168.6.200:2375 images
REPOSITORY            TAG        IMAGE ID       CREATED         SIZE
yzh/flask             latest     b9a97e8c23d1   6 days ago      704MB
centos_curl           latest     5a08f53ab762   7 days ago      526MB

# 远程查看
[root@sql ~]# docker -H 192.168.6.200:2375 ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

## 实践docker remote api

> restful管理docker

```
1.在docker宿主机本地实践api，查看docker信息，已经可以通过http协议访问(通过json.tool格式化)
[root@docker01 ~]# curl -s http://192.168.6.200:2375/info|python -m json.tool
{
    "Architecture": "x86_64",
    "BridgeNfIp6tables": true,
    "BridgeNfIptables": true,
    "CPUSet": true,
    "CPUShares": true,
    "CgroupDriver": "cgroupfs",
    "CgroupVersion": "1",
    "ContainerdCommit": {
        "Expected": "7b11cfaabd73bb80907dd23182b9347b4245eb5d",
        "ID": "7b11cfaabd73bb80907dd23182b9347b4245eb5d"
    },


2.管理镜像信息，获取docker守护进程里的列表
[root@docker01 ~]# curl -s  http://127.0.0.1:2375/images/json |python -m json.tool

3.在docker hub搜索镜像，也就是之前我们使用的docker search ubuntu这样
[root@docker01 ~]# curl -s  http://127.0.0.1:2375/images/search?term="ubuntu" |python -m json.tool

4.通过API管理容器
[root@docker01 ~]# curl -s  http://127.0.0.1:2375/containers/json  |python -m json.tool

这条命令，就等同于
[root@docker01 ~]# docker  -H 127.0.0.1:2375 ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
63df24d4f52e        training/webapp     "python app.py"     9 minutes ago       Up 9 minutes        0.0.0.0:32768->5000/tcp   competent_ishizaka

5.创建容器   (提示，创建容器，该镜像本地必须存在)
[root@docker01 ~]# curl -X POST -H "Content-Type:application/json" http://127.0.0.1:2375/containers/create -d '{"image":"centos"}'
{"Id":"072c0a7ef96dae6d5bdd093bab4d03f349ec5ee5383cd5ed27c582c951b81d7f","Warnings":[]}

6.删除容器，后面跟着容器id
[root@docker01 ~]# curl  -X DELETE    http://127.0.0.1:2375/containers/072c0a7ef96d?v=1
```

