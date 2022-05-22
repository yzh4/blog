## 介绍

###  NTP 简介

  Linux服务器运行久时，系统时间就会存在一定的误差，一般情况下可以使用date命令进行时间设置，但在做数据库集群分片等操作时对多台机器的时间差是有要求的，此时就需要使用ntpdate进行时间同步

###  ntp和ntpdate区别

- 两个服务都是centos自带的（centos7中不自带ntp）。ntp的安装包名是ntp,ntpdate的安装包是ntpdate。他们并非由一个安装包提供。

- ntp守护进程为ntpd，配置文件是/etc/ntp.conf

- ntpdate用于客户端的时间矫正，非NTP服务器可以不启动NTP。

## 安装

> ntpdate是客户端，直接crontab定时去同步时间即可

```
yum install -y ntpdate
```

## 配置

```
[root@docker scripts]# echo "*/5 * * * * /usr/sbin/ntpdate ntp.api.bz &> /dev/null" >> /var/spool/cron/root
[root@docker scripts]# crontab -l
*/5 * * * * /usr/sbin/ntpdate ntp.api.bz &> /dev/null
```

## ntp常用服务器

NTP服务器(上海) ：ntp.api.bz

阿里云 ：ntpdate -u ntp.aliyun.com

中国国家授时中心：210.72.145.44



> 若是同步有问题可以使用  `ntpdate -u ntp.api.bz`

## 修改时区

有时候我们设置了ntp时间服务，时间还是有问题，这时候我们还要修改时区

1.查看时间状态

```
timedatectl
```

2、设置系统时区为上海：

```
timedatectl set-timezone Asia/Shanghai 
```

3.列出所有时区

```
timedatectl list-timezones
```

4、将硬件时钟调整为与本地时钟一致, 0 为设置为 UTC 时间：

```
timedatectl set-local-rtc 1 
```

