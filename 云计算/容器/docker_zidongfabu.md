

## 工作流程

1.开发人员提交代码(git commit)代码至gitlab代码仓库

2.jenkins手动或自动触发项目构建

3.jenkins拉取代码，(maven)代码编译，打包镜像，推送镜像至harbor镜像仓库

4.jenkins在docker主机上创建容器并应用

## 主机规划

| **序号** | **主机名** | 主机IP        | **主机功能**                 | **软件**             |
| -------- | ---------- | ------------- | ---------------------------- | -------------------- |
| 1        | dev        | 192.168.6.22  | 开发者 项目代码 solo         | git                  |
| 2        | gitlab     | 192.168.6.20  | 代码仓库                     | gitlab               |
| 3        | jenkins    | 192.168.6.21  | 编译代码、打包镜像、项目发布 | jenkins、docker、git |
| 4        | harbor     | 192.168.6.201 | 存储容器镜像                 | harbor、docker       |
| 5        | web-server | 192.168.6.200 | 运行容器，项目上线           | docker               |



# 一、软件部署

## 部署jenkins

（jenkins服务器192.168.6.21）安装 jenkins,docker,manven

```
#安装jdk
[root@jenkins jdk]# rpm -ivh jdk-8u121-linux-x64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:jdk1.8.0_121-2000:1.8.0_121-fcs  ################################# [100%]
Unpacking JAR files...
	tools.jar...
	plugin.jar...
	javaws.jar...
	deploy.jar...
	rt.jar...
	jsse.jar...
	charsets.jar...
	localedata.jar...
[root@jenkins jdk]# java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)



#安装jenkins
[root@jenkins jenkins]# rpm -ivh jenkins-2.99-1.1.noarch.rpm 
warning: jenkins-2.99-1.1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID d50582e6: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:jenkins-2.99-1.1                 ################################# [100%]
[root@jenkins jenkins]# rpm -qa | grep jenkins
jenkins-2.99-1.1.noarch
[root@jenkins jenkins]# java -jar /usr/lib/jenkins/jenkins.war --version
2.99

#检查是否开机自启动 
 chkconfig --list
#如果没有开机自启动 
chkconfig jenkins on

#启动jenkins
[root@jenkins jenkins]# systemctl start jenkins
[root@jenkins jenkins]# systemctl status jenkins


#jenkins配置文件
[root@jenkins jenkins]# rpm -ql jenkins
/etc/init.d/jenkins
/etc/logrotate.d/jenkins
/etc/sysconfig/jenkins
/usr/lib/jenkins
/usr/lib/jenkins/jenkins.war
/usr/sbin/rcjenkins
/var/cache/jenkins
/var/lib/jenkins
/var/log/jenkins

```

进入jenkins

![image-20220316221220923](docker_zidongfabu.assets/image-20220316221220923.png)

```
#默认密码
[root@jenkins jenkins]# cat /var/lib/jenkins/secrets/initialAdminPassword


后修改密码
admin
admin
```

### 出现一直加载情况

![image-20220316232027786](docker_zidongfabu.assets/image-20220316232027786.png)

```
修改如下
[root@template jenkins]# cat /var/lib/jenkins/hudson.model.UpdateCenter.xml
<?xml version='1.0' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>http://mirror.xmission.com/jenkins/updates/update-center.json</url>
  </site>
</sites>


[root@template jenkins]# systemctl reload jenkins
[root@template jenkins]# systemctl restart jenkins
```

### 安装maven

下载地址：https://maven.apache.org/download.cgi

```
[root@jenkins jenkins]# ll
total 81332
-rw-r--r-- 1 root root  9136463 Mar 17 07:17 apache-maven-3.6.1-bin.tar.gz
-rw-r--r-- 1 root root 74141787 Mar 16 12:28 jenkins-2.99-1.1.noarch.rpm

[root@jenkins jenkins]# tar xfv apache-maven-3.6.1-bin.tar.gz 

```

### 安装git

```
yum install -y git
```

### 安装docker

配置网卡转发

```
1.docker必须安装在centos7平台，内核版本不低于3.10
在centos平台运行docker可能会遇见些告警信息，修改内核配置参数，打开内核转发功能

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
如果出现报错执行一下这个 modprobe br_netfilter

# 要保持本地软件源较新，可以用阿里云yum源更新软件

3.安装docker-ce社区版
## 下载阿里源repo文件
$ curl -o /etc/yum.repos.d/Centos-7.repo http://mirrors.aliyun.com/repo/Centos-7.repo
$ curl -o /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

$ yum clean all && yum makecache


## yum安装
$ yum install docker-ce-20.10.6 -y
## 查看源中可用版本
$ yum list docker-ce --showduplicates | sort -r
## 如果需要安装旧版本
##yum install -y docker-ce-18.09.9复制Error已复制
```

镜像加速器

使用docker首要操作就是获取镜像文件，默认下载是从Docker Hub下载，网速较慢，国内很多云服务商都提供了加速器服务，阿里云加速器，Daocloud加速器，灵雀云加速器。

```
1.修改docker配置文件，我们选用七牛云镜像站


[root@docker01 ~]# cat /etc/docker/daemon.json
{"registry-mirrors": ["https://reg-mirror.qiniu.com"]}

# 也可以
## 配置源加速
## https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
mkdir -p /etc/docker
vim /etc/docker/daemon.json
{"registry-mirrors" : ["https://8xpk5wnt.mirror.aliyuncs.com"]}


2.重启，设置开机自启
## 设置开机自启
systemctl enable docker  
systemctl daemon-reload


[root@docker01 ~]# systemctl restart docker

# 查看docker-client信息
## docker-client
which docker
## docker daemon
ps aux |grep docker
## containerd
ps aux|grep containerd
systemctl status containerd
```

## dev服务器部署git

（dev192.168.6.22）服务器安装git

- 开发人员主机安装git
- 下载项目及上传代码至代码仓库

```
[root@dev ~]# yum install -y git
```

## gitlab服务器安装gitlab

(gitlab192.168.6.22）服务器安装git

Gitlab安装配置

环境准备

```
操作系统centos7
内存：2G及以上
磁盘：50G
安全：关闭防火墙，selinux
```

1.安装GItlab所需的依赖包

```
yum install curl policycoreutils-python openssh-server postfix wget -y
```

2.安装gitlab，可以在线装，也可以安装本地准备好的rpm包，如果没有找超哥要

```
[root@lb02 opt]# yum localinstall gitlab-
gitlab-12-0-stable-zh.tar.gz          gitlab-ce-12.0.3-ce.0.el7.x86_64.rpm
[root@lb02 opt]# yum localinstall gitlab-ce-12.0.3-ce.0.el7.x86_64.rpm -y
```

3.配置gitlab服务，修改域名和邮箱

```
# 安装好后，默认提示Please configure a URL for your GitLab instance by setting `external_url`
# configuration in /etc/gitlab/gitlab.rb file.
需要修改配置文件，换成你自己的服务器地址就好
vim /etc/gitlab/gitlab.rb
external_url 'http://172.20.0.51'

配置邮箱，打开注释，修改配置，注意别改错了，否则将无法收到邮件
### Email Settings
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'yc_uuu@163.com'
gitlab_rails['gitlab_email_display_name'] = 'Onlyu-Gitlab'

# email server settings
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.163.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "yc_uuu@163.com"
gitlab_rails['smtp_password'] = ""
gitlab_rails['smtp_domain'] = "smtp.163.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
```

可以进入gitlab命令行，测试邮件收发是否正常

```
[root@lb02 ~]# gitlab-rails console
-------------------------------------------------------------------------------------
 GitLab:       12.0.3 (08a51a9db93)
 GitLab Shell: 9.3.0
 PostgreSQL:   10.7
-------------------------------------------------------------------------------------
Loading production environment (Rails 5.1.7)
irb(main):005:0> Notify.test_email('877348180@qq.com','hello','hello yuchao').deliver_now
Notify#test_email: processed outbound mail in 0.9ms
Sent mail to 877348180@qq.com (374.3ms)
Date: Wed, 08 Jul 2020 18:11:00 +0800
From: Onlyu-Gitlab <yc_uuu@163.com>
Reply-To: Onlyu-Gitlab <noreply@gitlab.onlyu.com>
To: 877348180@qq.com
Message-ID: <5f059bb4e7e31_e5253fb2a3fcf98c83859@lb02.mail>
Subject: hello
Mime-Version: 1.0
Content-Type: text/html;
 charset=UTF-8
Content-Transfer-Encoding: 7bit
Auto-Submitted: auto-generated
X-Auto-Response-Suppress: All

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "http://www.w3.org/TR/REC-html40/loose.dtd">
<html><body><p>hello yuchao</p></body></html>

=> #<Mail::Message:70036107130080, Multipart: false, Headers: <Date: Wed, 08 Jul 2020 18:11:00 +0800>, <From: Onlyu-Gitlab <yc_uuu@163.com>>, <Reply-To: Onlyu-Gitlab <noreply@gitlab.onlyu.com>>, <To: 877348180@qq.com>, <Message-ID: <5f059bb4e7e31_e5253fb2a3fcf98c83859@lb02.mail>>, <Subject: hello>, <Mime-Version: 1.0>, <Content-Type: text/html; charset=UTF-8>, <Content-Transfer-Encoding: 7bit>, <Auto-Submitted: auto-generated>, <X-Auto-Response-Suppress: All>>
```

4.初始化gitlab

```
# 注意，当修改了gitlab配置文件，就得reconfigure
[root@lb02 opt]# gitlab-ctl reconfigure

# 启停
gitlab-ctl start |restart |status |stop

# 初始化之后，gitlab组件都已经启动了
[root@lb02 opt]# gitlab-ctl status
```

**Gitlab服务构成**

```
GitLab 由主要由以下服务构成，他们共同承担了 Gitlab 的运作需要
Nginx:静态 web 服务器。
gitlab-shell:用于处理 Git 命令和修改 authorized keys 列表。 
gitlab-workhorse: 轻量级的反向代理服务器。 logrotate:日志文件管理工具。
postgresql:数据库。
redis:缓存数据库。
sidekiq:用于在后台执行队列任务(异步执行)。
unicorn:An HTTP server for Rack applications，GitLab Rails 应用是托管在这个
```

6.访问gitlab

```
http://192.168.6.20/

首次需要输入2次密码
admin
admin

然后可以登录，默认账号密码
root
admin
```

7.gitlab管理命令

```
gitlab-ctl start
gitlab-ctl stop
gitlab-ctl stop postgresql
gitlab-ctl restart 
gitlab-ctl status
gitlab-ctl reconfigure
gitlab-ctl tail
gitlab-ctl tail redis
```

gitlab主要目录

```
/var/opt/gitlab/git-data/repositories/ ：库默认存储目录
/opt/gitlab  ：应用代码和相应的依赖程序
/var/opt/gitlab/ ： gitlab-ctl reconfigure生成的数据和配置
/etc/gitlab ：配置文件目录
/var/log/gitlab：此目录下存放了gitlab各个组件产生的日志
/var/opt/gitlab/backups ：备份文件生成的目录
```

## 安装harbor,docker

> 首先安装docker和docker compose编排工具才可以安装harbor仓库

docker：如上

docker compose:

```
yum install -y docker-compose
```

在harbor-server安装

下载地址：https://github.com/goharbor/harbor/releases?page=1

```
[root@harbor local]# tar -xf harbor-offline-installer-v1.7.5.tgz

[root@harbor harbor]# ls
common                          docker-compose.yml    LICENSE
docker-compose.chartmuseum.yml  harbor.cfg            open_source_license
docker-compose.clair.yml        harbor.v1.7.5.tar.gz  prepare
docker-compose.notary.yml       install.sh


#修改配置文件
[root@harbor harbor]# diff harbor.cfg harbor.cfg.bak 
8c8
< hostname = 192.168.6.201
---
> hostname = reg.mydomain.com



安装harbor
[root@harbor harbor]# sh install.sh

✔ ----Harbor has been installed and started successfully.----

Now you should be able to visit the admin portal at http://192.168.6.201. 
For more details, please visit https://github.com/goharbor/harbor .


#查看harbor镜像
[root@harbor harbor]# docker images
REPOSITORY                      TAG             IMAGE ID       CREATED       SIZE
goharbor/chartmuseum-photon     v0.8.1-v1.7.5   ca910ae935e8   2 years ago   113MB
goharbor/harbor-migrator        v1.7.5          326685e45678   2 years ago   679MB
goharbor/redis-photon           v1.7.5          6886d1b5b009   2 years ago   101MB
goharbor/clair-photon           v2.0.8-v1.7.5   970bdadc8995   2 years ago   164MB
goharbor/notary-server-photon   v0.6.1-v1.7.5   667b2f0fb68d   2 years ago   135MB
goharbor/notary-signer-photon   v0.6.1-v1.7.5   9ea20ae211bc   2 years ago   132MB
goharbor/harbor-registryctl     v1.7.5          b5781fe85ba3   2 years ago   102MB
goharbor/registry-photon        v2.6.2-v1.7.5   789d7a921303   2 years ago   86.7MB
goharbor/nginx-photon           v1.7.5          172196970055   2 years ago   35.9MB
goharbor/harbor-log             v1.7.5          26784a32e151   2 years ago   81.4MB
goharbor/harbor-jobservice      v1.7.5          0755114a9c68   2 years ago   84.1MB
goharbor/harbor-core            v1.7.5          86925c0ee5d5   2 years ago   95.6MB
goharbor/harbor-portal          v1.7.5          9a9082e62030   2 years ago   40.6MB
goharbor/harbor-adminserver     v1.7.5          b2afe190593f   2 years ago   72.3MB
goharbor/harbor-db              v1.7.5          c1226370ffaa   2 years ago   138MB



#添加daemon.json文件
[root@harbor harbor]# cat /etc/docker/daemon.json
{
"insecure-registries": ["http://192.168.6.201"]
}


[root@docker ~]# systemctl daemon-reload 
[root@docker ~]# systemctl daemon-reload 
[root@docker ~]# systemctl enable docker
[root@docker ~]# systemctl restart docker

```

web验证harbor

![image-20220317220609597](docker_zidongfabu.assets/image-20220317220609597.png)

账号：admin

密码：Harbor12345

停止harbor：
 docker-compose down -v：停止并移除整个project的所有services

```
[root@localhost harbor]# docker-compose down -v
Stopping harbor-jobservice  ... done
Stopping nginx              ... done
Stopping harbor-ui          ... done
Stopping redis              ... done
```

重启服务：

```
[root@localhost harbor]# systemctl daemon-reload

[root@localhost harbor]# systemctl restart docker
```

启动harbor：
 docker-compose up -d ：构建（容器）并启动（容器）整个project的所有service

```
[root@localhost harbor]# docker-compose up -d
```

## docker服务器

安装docker

```
和上面的docker一样安装

配置可以登录harbor的命令，添加daemon.json文件
[root@harbor harbor]# cat /etc/docker/daemon.json
{
"insecure-registries": ["http://192.168.6.201"]
}
```

# 配置

### 配置开发人员主机密钥至gitlab

```
#生成密钥
[root@dev ~]# ssh-keygen -t rsa -f /root/.ssh/id_rsa -P ''
Generating public/private rsa key pair.
/root/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:p0MTRbe1zHd4KkrB/M0YoXX9ZUdOGpJAvBtZNXI008Q root@dev
The key's randomart image is:
+---[RSA 2048]----+
|         +=.*+@*+|
|         +.+o@ @E|
|        . =+o *.O|
|         .+o = +o|
|        S oo+ +  |
|       . =.. .   |
|        o .      |
|         .       |
|                 |
+----[SHA256]-----+

[root@dev ~]# ls /root/.ssh/
authorized_keys  id_rsa  id_rsa.pub  known_hosts

#生成便于标识的开发者密钥
[root@dev ~]# ssh-keygen -t rsa -f /root/.ssh/id_rsa -C "dev@aiops.net.cn" -P ''
Generating public/private rsa key pair.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:HfPBmDmQfdJUvHUqCRR5WElPsYAPQzi0UnwOqDd4h/c dev@aiops.net.cn
The key's randomart image is:
+---[RSA 2048]----+
|       ++*+X=++. |
|      ..*o@==+..o|
|     o...=BOo.o+.|
|    o =.o..=+.o  |
|     o +S.. ..   |
|          E      |
|                 |
|                 |
|                 |
+----[SHA256]-----+

[root@dev ~]# ls /root/.ssh/
id_rsa  id_rsa.pub

[root@dev ~]# cat /root/.ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDKRFyLy4U1EyDwIaZu82v2EByXEE8nlaU67iHASmNhHUV2Pt4fLQhsKmOdnFk8zDt8BpHt0TOw9/vpbrYeCpP8S0B6u737P9e5zOMcoxq8e04hSHkt+HGj+kMAHjcKncm49a3rKzNrr/23LMnFDjVVcF3NGbW32qEXdPpRozmPciTIJpiEG2pPtnqJ6fJiDCCUZa9WT1sbwHeHCpchLaK194zUfG2tTz8+t3diRbJfpyj4rKTegH7jCEJBdaXhWGs4D0WdDXnEgw+vhMRlvpn0WEXiPf4PQ8IEU1smJViMejHfd5j35Gv/IVhsmaZjuO/sCI3Tz+BB2kdlZYMrtt4b dev@aiops.net.cn

```

### gitlab添加密钥

![image-20220318124737106](docker_zidongfabu.assets/image-20220318124737106.png)



### 配置jenkins主机密钥至gitlab

**生成密钥对**

```
[root@jenkins ~]# ssh-keygen -t rsa -f /root/.ssh/id_rsa -C 'jenkins@aiops.net.cn' -P ''
Generating public/private rsa key pair.
Created directory '/root/.ssh'.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:QAhOFwhPMsWdbLTzM36N5qvKvUywD9EX1vzItJuLt14 jenkins@aiops.net.cn
The key's randomart image is:
+---[RSA 2048]----+
|++=*++.          |
| Bo.*o   o       |
|  o.o . o +      |
|     + o + +     |
|    o = S + .    |
|     = + o o     |
|    o o + + E    |
|   . * + ..o     |
|    o.*o+++.     |
+----[SHA256]-----+

[root@jenkins ~]# ls /root/.ssh/
id_rsa  id_rsa.pub
[root@jenkins ~]# cat /root/.ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDU1hICpAyiBNlV7IirzXfRLSDCbBImtLRggwdfYUlq55e5VZ84rZLkqjMHcD69NX65qJJJM2GF+X5YSK40n/0Q7DmNuDorpXRdlx8tPjDunDIzMdFxSHO4zgXteI+NRbzAWNmQwPSzkZ8GWcQ0d474xeQNh8dYa1TWWaNrsuZ5BflvL6X7at9ISnlfg4ZOXLj90g8ulAKjsBt9NZmnhxwGxp7ual0IdNm2udgljXP1LPEZgzoTkaiV/J8RYWcSgLQN8ZJxqAxhrMfn9IjHUWJCMJVMbkO6U4peKYXSRFdr29IDCe3pUeHAN1T4DkpOVYTRoDXDjSRiKiu/xaGoXvrJ jenkins@aiops.net.cn

```

![image-20220318153022868](docker_zidongfabu.assets/image-20220318153022868.png)

### 在jenkins添加私钥

![image-20220319123509420](docker_zidongfabu.assets/image-20220319123509420.png)



![image-20220319124247049](docker_zidongfabu.assets/image-20220319124247049.png)

```
[root@jenkins var]# cat /root/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpgIBAAKCAQEA2BHjs6oIdbCdtyCwmH6kdiWVI/s/oskWmMdaOzn3Qko9ATlq
WOeTUgDbvaXcqwsBWdFISaoD3HNVVhOM+2Bdj6kUz465ObeqZMy2qklKL36R8aoK
yMcqvML82xC+hqXhBg1sVAQ21ECygE7CSDM+JTayeIwwwus1RVAjTRId75qB0rEC
tqR2Vl6SJ9NOhqlWW+2tg03na9Chue4i9CaXdybVJIFCMKYSD8wOq66LawTTLzc/
FfVgyDazdg0kse7yK97AAjNJv0I24yxKdBRY0XcSjYVgzJmYsu8j3wpzcfJBc56u
CTHnQpnHJRzzY4hvORFW1uGC0gY6gs9Dz6SteQIDAQABAoIBAQCNCRBSQ9iFyaUI
1eOvhmqfW+S/uc5N2WhQjT+OjUWiFYdAd1O9eFsnGbxVTCB/3Q0irYK74qfmi7nW
F6jcpIgtr7YPzR+GVyZ36VAKUxCDc+Cbl9y2NZrl33jhHUrf5QK+yqA9LwsC/MAH
FGGocxpLfCp+mTNy58i715VTR1IXBNWfscNyQ4biDlcReRHHtQSkS48JukWlCjWY
zZivU7VCgfctM5qcho2f4cWXLFMenXMa2yfboduV4mwyCNjpMqgM5Jz54/IxeqKJ
wkcsAaCltGg/uYe17mNp/cG5ihAVQi80YPucG3QIQ8l1JFvlLJQxp8pNwgXLCkpA
h+RHhhgBAoGBAPuphjNtsndTL5Ku3JDkGspkilEOKk5/xuZFn9P9BOGA81OLbx7W
ST6RG/fqmGix7VIUfjUoZEQgY6gCXQJDkCUNAZx61yGaO2YetM1lAqnAcasRcNnz
FJd6O6gW5aNNCpOp4iyYVJ/8GZOeu0VrdWJmrZgyk6dZ4noC2WRt3XiBAoGBANvL
T9KBHvGoZKWvFfW0Otvo7bxwfMGn5pN/G0bVQfmesLCCLpCmqBKp1sBMVOXqgeiJ
Ym2x53u2i8Id1G4cAPuJw9BLf6fa1tg1rKkBqMYP/1krVAZCrTMnJdvY7N+QYghm
dExRJAl1WP5FM2PfiR0iRO/0skcobVomZbQzvnj5AoGBAIJnEmH9upc4JJOGRkZR
3Ev5ku5zP5rTfRX/B9WJ6wiKkAs/WgIJmYRxHqP1DKpbYBTDB6610la2zczWBKnS
ppN8TNRvlqAT0imVSayQZbPeQ0gYTnixC39+WAE+dBmNJyjiw47GFDueHwauHm3X
Q8kMWXFe68GwwrjkVw9CGpuBAoGBAJcWCQbQz26arKTf2B6QB63HTjSstnWAbxWQ
2DU2vyVSGm8a7dttVDeQHBkP34Q0UZyBY/G3y0HVV/jV/IK8KUi/iKUUYOoHmowB
SXDbvSJ5OLQY/qdXCiwRiwq28Q00+M6BbwxO8pC147jV5N+Qd3ZZaj3wXORhO9ou
btcSgqoxAoGBAPBGlpykRWVF/QAEt/DLKdLPbS8fD1zZeVBUpg6/0WNykZeZ8BZP
zAJjqAUNf29MKCY2UJi+APLzjMoTSGu6+0QLOJ9z3Kbeg/4jrJs5gkJ34mqMW7Xy
rvHGSFHAKmsD6cJqbN2csr5CzxDpeoTa5yk1odOgYu7oWqkEwYvAuq3c
-----END RSA PRIVATE KEY-----
```

### 配置jenkins使用docker

验证系统中是否有jenkins用户

```
[root@jenkins var]# grep "jenkins" /etc/passwd
jenkins:x:996:995:Jenkins Automation Server:/var/lib/jenkins:/bin/false
```

验证系统中是否有docker用户及用户组

```
[root@jenkins var]# grep "docker" /etc/group
docker:x:994:

```

添加jenkins用户到docker用户组

```
[root@jenkins var]# usermod -G docker jenkins
[root@jenkins var]# grep docker /etc/group
docker:x:994:jenkins
```

### jenkins-server添加插件

安装的插件有：

- ssh 用于jenkins-server对web-server进行操作

- git parameter 用于git版本提交进行参数构建

- gitlab 用于jenkins-server拉取项目

- gitlab hook 用于项目自动构建

- maven integration 用于编译

