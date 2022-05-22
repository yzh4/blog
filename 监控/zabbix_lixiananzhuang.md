## 背景

由于生产环境与互联网隔离，无法在线安装部署，需要在无互联网环境下离线安装。采用创建离线yum源的方式进行安装部署。 服务器操作系统为 ：Centos 7.5 ZABBIX组件选择为 zabbix 5.0LTS

## 离线获取zabbix软件包

准备一个可以上网的linux机器

> 准备zabbix镜像源

```perl
rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm

sed -i 's#http://repo.zabbix.com#https://mirrors.aliyun.com/zabbix#' /etc/yum.repos.d/zabbix.repo

yum clean all
```

> 通过yum下载zabbix，但是不安装，只下载所有的rpm包

```perl
yum install  --downloadonly   --downloaddir=/zabbix_repo  zabbix-server-mysql zabbix-agent  zabbix-agent2  zabbix-java-gateway zabbix-js zabbix-get zabbix-sender  net-snmp centos-release-scl  createrepo
```

> 编辑配置文件 /etc/yum.repos.d/zabbix.repo 并开启 zabbix-frontend 源

```perl
[zabbix-frontend]
...
enabled=1
...
```

> 通过yum下载并保存zabbix前端 及其依赖的rpm包

```perl
yum install --downloadonly   --downloaddir=/zabbix_repo  zabbix-web-mysql-scl zabbix-apache-conf-scl zabbix-nginx-conf-scl
```

> 获取数据库软件包

```perl
yum install --downloadonly --downloaddir=/zabbix_repo   mariadb-server
```

## 安装离线包

> 将上述离线安装好的软件包，也就是/zabbix_repo 的内容，压缩打包
>
> 以后再没有网络的机器上，就可以通过该软件包，解压缩，获取安装zabbix所需的所有rpm包，一键安装即可

```perl
# 解压缩到根目录
tar zxf zabbix_repo.tar.gz  -C /
```

> 删除且备份现有的yum源文件

```perl
find  /etc/yum.repos.d/ -name *.repo  -exec mv {} {}.bak \;
```

> 新增repo文件，用于安装我们本地的rpm包

```perl
tee  /etc/yum.repos.d/zabbix.repo <<EOL
[zabbix]
name=zabbix
baseurl=file:///zabbix_repo
gpgcheck=0
enabled=1
EOL
```

> 此时可以yum安装zabbix了，通过本地源，无须网络

```perl
yum install zabbix-server-mysql zabbix-agent2 centos-release-scl -y
```

> 删除scl源文件

```perl
find  /etc/yum.repos.d/ -name CentOS-SCLo*.repo  -exec mv {} {}.bak \;
```

> 安装zabbix客户端

```perl
yum install zabbix-web-mysql-scl zabbix-apache-conf-scl zabbix-nginx-conf-scl -y
```