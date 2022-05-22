## docker镜像

> docker镜像使用学习

在进行开发时，需要安装很多第三方工具,例如etcd，kafka等等

开发机器，mac，windows,不想搞乱当前机器环境时

1.下载安装docker工具

2.获取该软件docker镜像，例如nginx镜像。`docker pull nginx`

3.运行该镜像,然后启动了一个容器，这个nginx服务就运行在容器中

4.停止容器，删除该镜像，宿主机就没有使用过nginx一样

## 获取docker镜像

1.从dockerhub获取

2.本地导出

3.私有docker仓库

```
#获取镜像，镜像托管仓库，好比yum源一样
#默认的docker仓库是，dockerhub,有大量优质镜像，以及用户自己上传的镜像.centos容器
vim nginx ... 提交为镜像，上传到dockerhub
```

查看当前是否有容器在运行中

```
[root@docker01 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

查看本地镜像文件有哪些

```
[root@docker01 ~]# docker image ls
[root@docker01 ~]# docker images
```

下载docker镜像

```
[root@docker01 ~]# docker pull centos #默认是 centos:latest
[root@docker01 ~]# docker pull centos:7.8.2003 #指定版本
```

查看docker镜像存储路径

```
[root@docker01 ~]# docker info | grep Root
Docker Root Dir: /var/lib/docker
 
 
[root@docker01 ~]# ls /var/lib/docker/image/overlay2/imagedb/content/sha256/ -l
total 24
-rw------- 1 root root 2142 Feb 12 08:31 5d0da3dc976460b72c77d94c8a1ad043720b0416bfc16c52c45d4847e53fadb6
```

开启交互式终端

```
# -it 开启交互式终端
# --rm 容器退出时删除该容器
[root@docker01 ~]# docker run -it --rm centos bash
[root@ccb2f785b7b9 /]# cat /etc/redhat-release 
CentOS Linux release 8.4.2105
```

## 查看docker镜像

docker images

```
[root@docker01 ~]# docker images
```

查看具体镜像

```
[root@docker01 ~]# docker images centos
```

只列出镜像id

```
# -q 只列出id
[root@docker01 ~]# docker images -q
```

格式化显示镜像

```
[root@docker01 ~]# docker images --format "{{.ID}}--{{.Repository}}"
f6987c8d6ed5--nginx
```

查看正在运行的容器镜像

```
[root@docker01 ~]# docker ps
```

哪些镜像运行过

```
[root@docker01 ~]# docker ps -a
```

## 删除docker镜像

镜像名删除镜像

```
#被删除镜像不得有依赖记录
[root@docker01 ~]# docker rmi hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:2498fce14358aa50ead0cc6c19990fc6ff866ce72aeb5546e1d59caac3d0d60f
Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
Deleted: sha256:e07ee1baac5fae6a26f30cabfe54a36d3402f96afda318fe0a96cec4ca393359
```

删除容器记录

```
[root@docker01 ~]# docker rm e58f31f51ff4
e58f31f51ff4
```

id删除镜像

```
#指定前三位id即可
docker rmi feb
```

## 镜像管理

批量删除镜像

!> (危险命令，慎用)

```
docker rmi `docker images -aq`
```

批量删除容器

```
docker rm `docker ps -aq`
```

导出镜像

> 比如默认运行的centos镜像，不提供vim功能，运行该容器后，在容器内安装vim
> 然后提交该镜像在导出该镜像为压缩文件，可以发给其他人用

```
docker image save centos:7.2.1511 > /opt/centos7.8.2003.tgz
```

导入镜像

```
[root@docker01 ~]# docker image load -i /opt/centos7.8.2003.tgz 
a11c91bfd866: Loading layer    202MB/202MB
Loaded image: centos:7.2.1511
```

查看docker服务信息

```
docker info
```

查看镜像详细信息

```
docker image inspacet 镜像id(前3位)
```

