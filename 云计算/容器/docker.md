# 一.概念

## docker 是什么

- 使用最广泛的开源容器引擎
- 一种操作系统级的虚拟化技术
- 依赖于Linux内核特性：Namespace（资源隔离）和Cgroups（资源限制）
- 一个简单的应用程序打包工具

## docker优势/及我们为何要用docker?

### 1.资源利用更出色

主要体现在高效性方面。由于容器不需要进行硬件虚拟以及运行完整操作系统等额外开销，Docker对系统资源的利用率更高，无论是应用执行速度，内存消耗以及文件存储速度，都要比传统虚拟机技术更高效。因此，相比虚拟机技术，一个相同配置的主机，往往可以运行更多数量的应用。

### 2.秒级的启动速度

传统的虚拟机技术启动应用服务往往需要数分钟，而Docker容器应用，由于直接运行与宿主内核，无序启动完整的操作系统，因此可以做到秒级，甚至毫秒级的启动时间，大大的节约了开发，测试，部署的时间。Docker 容器的运行不需要额外的 hypervisor 支持，它是内核级的虚拟化，因此可以实现更高的性能和效率。

### 3.一致的运行环境

发布服务不用担心服务器的运行环境，所有的服务器都是自动分配docker，自动部署，自动安装，自动运行。因此遇到开发过程中一个常见的问题，即环境一致性问题，由于开发环境，测试环境，生产环境不一致，导致有些bug并未在开发过程中被发现，而Docker的镜像提供了除内核外完整的运行时环境，确保了应用运行环境一致性。官方就是Bulid 、ship、run any app/any where，编译、装载、运行、任何app/在任意地方都能运行。 就是实现了应用的封装、部署、运行的生命周期管理只要在glibc的环境下，都可以运行。

### 4.持续交付和部署

对于运维和开发人员来说，最希望的就是一次创建或配置，可以在任意地方正常运行。使用Docker可以通过定制应用镜像来实现持续集成，持续交付，部署。开发可以通过Dockerfile来进行镜像构建，并结合持续集成系统进行集成测试，而运维则可以在生产环境中快速部署该镜像，甚至结合持续部署系统进行自动部署。

### 5.便捷的自动迁移

自动迁移，可以制作镜像，迁移使用自定义的镜像即可迁移。由于Docker确保了执行环境的一致性，使得应用的迁移更加容易，Docker可以在很多平台上运行，无论是物理机，虚拟机，公有云，私有云，还是是笔记本，其运行结果是一致的，因此用户可以很轻易的将在一个平台上运行的应用，迁移到另一个平台上，而不用担心运行环境的变化导致应用无法正常运行的情况。

### 6.可以拓展和堆叠

Docker使用的分层存数以及镜像的技术，使得应用重复部分的复用更为容易，也使得应用的维护更新更加简单，基于基础镜像进一步扩展镜像也变得非常简单。因此，可扩展说的是可以增加并自动分发容器副本；而可以堆叠，说的是可以垂直和即时堆叠服务。

### 7.低成本

Docker团队同各个开源项目团队一起维护了大批高质量的官方镜像，既可以直接在生产环境使用，又可以作为基础进一步定制，大大的降低了应用服务的镜像制作成本。

### 8.自动化的管理

开发者可以使用一个标准的镜像来构建一套开发容器，开发完成之后，运维人员可以直接使用这个容器来部署代码。Docker 可以快速创建容器，快速迭代应用程序，并让整个过程全程可见，使团队中的其他成员更容易理解应用程序是如何创建和工作的。此外，使用 Docker只需要小小的修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式被分发和更新，从而实现自动化并且高效的管理。

## 对比于传统虚拟机

| 特性       | 容器               | 虚拟机     |
| ---------- | ------------------ | ---------- |
| 启动       | 秒级               | 分钟级     |
| 硬盘使用   | 一般为 MB          | 一般为 GB  |
| 性能       | 接近原生           | 弱         |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |

## docker基本组成

- Docker Client：客户端
- Ddocker Daemon：守护进程
- Docker Images：镜像
- Docker Container：容器
- Docker Registry：镜像仓库

#### 1.docker image 镜像

镜像是容器的基石，容器基于镜像启动和运行。镜像就好像容器的源代码，保存了容器各种启动的条件。镜像是一个层叠的只读文件系统。

#### 2.docker container 容器

容器通过镜像来启动，容器是docker的执行来源，可以执行一个或多个进程。镜像相当于构建和打包阶段，容器相当于启动和执行阶段。容器启动时，Docker容器可以运行、开始、停止、移动和删除。每一个Docker容器都是独立和安全的应用平台。

#### 3.docker registry 仓库

docker仓库用来保存镜像。docker仓库分为公有和私有。docker公司提供公有仓库docker hub,网址：https://hub.docker.com/。我们也可以创建自己私有的仓库。

## docker应用场景

- 应用程序打包和发布
- 应用程序隔离
- 持续集成
- 部署微服务
- 快速搭建测试环境
- 提供pass产品[平台即服务]



# 二.常用命令

### docker安装

```
机器环境初始化

1.防火墙 selinux关闭
systemctl stop firewalld
systemctl disable firewalld
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

2.yum源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

3.安装基础软件
yum install -y bash-completion vim lrzsz wget expect net-tools nc nmap tree dos2unix htop iftop iotop unzip telnet sl psmisc nethogs glances bc ntpdate  openldap-devel
```

```
docker机器准备,配置网卡转发

1.在centos平台运行docker可能会遇见些告警信息，修改内核配置参数，打开内核转发功能

# 写入
cat <<EOF >  /etc/sysctl.d/docker.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.all.rp_filter = 0
net.ipv4.ip_forward=1
EOF


2.重新加载内核参数
[root@docker01 ~]# sysctl -p /etc/sysctl.d/docker.conf

如果出现报错执行一下
modprobe br_netfilter

# 要保持本地软件源较新，可以用阿里云yum源更新软件

3.安装docker-ce社区版,下载阿里源repo文件
curl -o /etc/yum.repos.d/Centos-7.repo http://mirrors.aliyun.com/repo/Centos-7.repo

curl -o /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo


yum clean all && yum makecache


#yum安装
yum install docker-ce-20.10.6 -y
#查看源中可用版本
yum list docker-ce --showduplicates | sort -r
#如果需要安装旧版本
yum install -y docker-ce-18.09.9
```

```
配置镜像加速器

1.创建docker镜像配置文件
mkdir -p /etc/docker
vim /etc/docker/daemon.json
{"registry-mirrors" : ["https://8xpk5wnt.mirror.aliyuncs.com"]}

2.重启，设置开机自启
systemctl daemon-reload
systemctl restart docker
systemctl enable docker


# 查看docker-client信息
## docker-client
which docker
## docker daemon(守护进程)
ps aux |grep docker
## containerd
ps aux|grep containerd
systemctl status containerd
```

```
docker管理

1.检查服务状态
[root@docker01 ~]# docker version

2.检查系统内核
[root@docker01 ~]# uname -a

3.检查系统是否安装了存储驱动，看到结果即可
[root@docker01 ~]# ls -l /sys/class/misc/device-mapper/

# 若是没有，单独安装即可
[root@docker01 ~]# yum install device-mapper -y

# 加载存储驱动模块
[root@docker01 ~]# modprobe dm-mod
```

### docker常用命令

#### 镜像管理

查看帮助参数

```
docker --help
```

拉取/下载镜像

```
docker pull nginx
```

推送/上传镜像

```
docker login

docker tag diy/nginx:v1 yzh236/nginx:v1

docker push yzh236/nginx:v1
```

查看容器

```
docker container ls

docker ps -a
```

查看镜像

```
docker image ls

docker images
```

查看镜像详细信息

```
docker image inspect nginx
```

查看历史

```
docker history nginx
```

删除镜像

```
docker image rm -f nginx

#删除所有镜像
docker image rm -f `docker images -q`

#筛选删除
docker image rm -f `docker images|grep hello-world|awk '{print $3}'`
```

镜像导出

```
docker image save centos > centos.tar.gz

docker save -o nginx.tar.gz nginx 
```

镜像导入

```
docker image load -i centos.tar.gz

docker load <  nginx.tar.gz
```

#### 容器管理

参数

| 选项              | 描述                                        |
| :---------------- | :------------------------------------------ |
| -i, –interactive  | 交互式                                      |
| -t, –tty          | 分配一个伪终端                              |
| -d, –detach       | 运行容器到后台                              |
| -e, –env          | 设置环境变量                                |
| -p, –publish list | 发布容器端口到主机                          |
| -P, –publish-all  | 发布容器所有EXPOSE的端口到宿主机随机端口    |
| –name string      | 指定容器名称                                |
| -h, –hostname     | 设置容器主机名                              |
| –ip string        | 指定容器IP，只能用于自定义网络              |
| –network          | 连接容器到一个网络                          |
| –mount mount      | 将文件系统附加到容器                        |
| -v, –volume list  | 绑定挂载一个卷                              |
| –restart string   | 容器退出时重启策略，默认no，可选值：[always |

创建容器

```
docker container run -d -p 80:80 nginx

docker run -d -e JAVA_HOME=/usr/local/jdk --name test1 -h java8 java:8
```

容器资源访问限制

| 选项                       | 描述                                                        |
| :------------------------- | :---------------------------------------------------------- |
| -m，–memory                | 容器可以使用的最大内存量                                    |
| –memory-swap               | 允许交换到磁盘的内存量                                      |
| –memory-swappiness=<0-100> | 容器使用SWAP分区交换的百分比（0-100，默认为-1）             |
| –oom-kill-disable          | 禁用OOM Killer （内存不足时，选择一个占用较大的进程kill掉） |
| –cpus                      | 可以使用的CPU数量                                           |
| –cpuset-cpus               | 限制容器使用特定的CPU核心，如(0-3, 0,1)                     |
| –cpu-shares                | CPU共享（相对权重）                                         |

内存限制，允许容器最多使用500M内存和100M的Swap，并禁用 OOM Killer

```
docker run -d --name nginx03 --memory="500m" --memory-swap=“600m" --oom-kill-disable nginx

docker stats 容器id
```

cpu限制，允许容器最多使用50%的cpu

```
docker run -d --name nginx05 --cpus=".5" nginx
```

#### 管理容器常用命令

| 选项       | 描述                       |
| :--------- | :------------------------- |
| ls         | 列出容器                   |
| inspect    | 查看一个或多个容器详细信息 |
| exec       | 在运行容器中执行命令       |
| commit     | 创建一个新镜像来自一个容器 |
| cp         | 拷贝文件/文件夹到一个容器  |
| logs       | 获取一个容器日志           |
| port       | 列出或指定容器端口映射     |
| top        | 显示一个容器运行的进程     |
| stats      | 显示容器资源使用统计       |
| stop/start | 停止/启动一个或多个容器    |
| rm         | 删除一个或多个容器         |

列出容器

```
docker container ls
docker ps
docker ps -l
```

容器详情

```
docker inspect 容器id
```

进入容器执行命令

```
docker exec -it 容器id bash
```

不进入容器执行命令

```
docker exec 容器id 命令
```

拷贝

```
docker cp 宿主机本地文件 容器ID:容器内目录
docker cp 容器ID:容器内目录 宿主机本地文件 
```

日志

```
docker logs 容器id
```

端口映射

```
docker run -d -p80:80 nginx

docker run -d -P nginx
```

容器快照导入和导出

```
docker export 容器id > 容器名.tar  #导出快照

cat 容器名.tar | docker import - 容器名    #导入快照

区别：容器的快照信息将丢弃历史记录和元数据信息（只保留容器当时的快照状态），而镜像的存储文件将保存完整记录
```

容器进程

```
docker top 容器id
```

查看容器资源信息

```
docker stats 容器id
```

启停容器

```
docker start|stop|restart 容器id
```

删除容器

```
# 删除停止的容器
docker rm 容器id

# 强制删除容器
docker rm -f 容器id

# 强制删除所有容器
docker rm -f `docker ps -a |awk '{print $1}'`
```

# 三.dockerfile案例

### dockerfile

​    Dockerfile 是一个文本格式的配置文件，用户可以使用 Dockerfile 快速创建自定义的镜像。我们会先介绍 Dockerfile 的基本结构及其支持的众多指令，并具体讲解通过执行指令来编写定制镜像的 Dockerfile。



​    Dockerfile 由一行行命令语句组成，并且支持已 # 开头的注释行。一般而言，Dockerfile 的内容分为四个部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

```bash
# This dockerfile uses the Ubuntu image
# VERSION 2
# Author: yuezenghui
# Command format: Instruction [arguments / command] …

# 第一行必须指定基于的容器镜像
FROM ubuntu

# 维护者信息
MAINTAINER yuezenghui 2361046420@qq.com

# 镜像的操作指令
RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo “\ndaemon off;” >> /etc/nginx/nginx.conf

# 容器启动时执行指令
CMD /usr/sbin/nginx
```

### dockerfile常用指令

| 指令             | 描述                                                   |
| :--------------- | :----------------------------------------------------- |
| FROM             | 构建新镜像是基于哪个镜像                               |
| MAINTAINER LABEL | 镜像维护者姓名或邮箱地址                               |
| RUN              | 构建镜像时运行的Shell命令                              |
| COPY             | 拷贝文件或目录到镜像中                                 |
| ADD              | 拷贝文件，会自动解压                                   |
| ENV              | 设置环境变量                                           |
| USER             | 为RUN、CMD和ENTRYPOINT执行命令指定运行用户             |
| EXPOSE           | 声明容器运行的服务端口                                 |
| HEALTHCHECK      | 容器中服务健康检查                                     |
| WORKDIR          | 为RUN、CMD、ENTRYPOINT、COPY和ADD设置工作目录          |
| ENTRYPOINT       | 运行容器时执行，如果有多个ENTRYPOINT指令，最后一个生效 |
| CMD              | 运行容器时执行，如果有多个CMD指令，最后一个生效        |

### 自定义构建nginx镜像

```
[root@docker docker]# cat Dockerfile_nginx 

FROM centos:7
MAINTAINER yuezenghui
WORKDIR "/tmp"
ADD nginx-1.16.0.tar.gz /tmp
RUN useradd -M -s /sbin/nologin nginx
RUN  yum -y install gcc*  make pcre-devel zlib-devel openssl openssl-devel libxslt-devel gd gd-devel GeoIP GeoIP-devel pcre pcre-devel \
    && cd nginx-1.16.0 \
    && ./configure --user=nginx --group=nginx --prefix=/usr/local/nginx --with-file-aio --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module --with-http_image_filter_module --with-http_geoip_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_auth_request_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_stub_status_module && make && make install \
    && echo "hello nginx" > /usr/local/nginx/html/index.html

EXPOSE 80

# 启动nginx 将nginx主进程 pid为1 nginx一旦挂掉那么docker容器就会直接退出
CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]

-g daemon off解释：
CMD在执行的shell脚本["sh", "test_api.sh"]，实际上是启动shell进程来执行，脚本执行完，进程就会退出(此时nginx还是一摊死的物理文件)，所以我们要在脚本内再添加nginx -g "daemon off;" 将整个shell进程转为前台能持续运行的进程。
```

构建

```
docker build -f Dockerfile_nginx -t diy/nginx:v1 .

docker run  -d -p 80:80 yzh236/nginx:v1
```

上传到docker hub仓库

```
docker tag diy/nginx:v1 yzh236/nginx:v1

docker push yzh236/nginx:v1
```





