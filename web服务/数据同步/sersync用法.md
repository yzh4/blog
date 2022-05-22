## sersync的简单原理

采用inotify来对文件进行监控，当监控到文件有文件发生改变的时候，就会调用rsync实现触发式实时同步！

> 下载：https://code.google.com/archive/p/sersync/downloads

Sersync项目利用Inotify和Rsync工具技术实现对服务器数据实时复制。

当事件发生变化后，利用rsync命令把变化的数据复制到远端服务器上。

Sersync特点

- 使用C++编写，支持对监控事件的过滤
- Sersync采用xml配置文件，由守护进程启动，配置起来比起简易的`inotify+rsync更简单`
- 使用多线程复制，可以并发复制多个不同文件，效率更高
- Sersync自带异常检测机制，可以通过`失败队列`对出错的文件重新复制
- 自带crontab功能，实现对`失败队列`中的文件定时整体复制
- 自带socket和HTTP协议扩展，定制特殊需求，二次开发

### 安装sersync

> （注意sersync是工作在rsync的源服务器上，也就是客户端上）

```
下载好之后分配成这种目录结构
[root@rsync01 sersync]# tree /opt/sersync
/opt/sersync
├── bin
│   └── sersync
├── conf
│   └── confxml.xml
└── log
```

## Sersync配置文件

```bash
1.修改配置文件，修改如下部分
[root@nfs01 conf]# vim /opt/sersync/conf/confxml.xml

***********************************30行开始******************************
   <commonParams params="-artuz"/>  #-artuz为rsync同步时的参数
    <authstart="true" users="rsync的虚拟用户名(rsync_backup)" passwordfile="rsync的密码文件"/>
   <userDefinedPort start="true"port="873"/><!-- port=874 -->
   <timeout start="false" time="100"/><!--timeout=100 -->
    <sshstart="false"/>
       ************************************第36行***********************************
       <failLogpath="自己定义的log文件夹(/usr/local/sersync/log)rsync_fail_log.sh"
       timeToExecute="60"/><!--defaultevery 60mins execute once-->
       *******************************************************************************
       *注：若有多个目录备份可以穿件多个配置文件在启动时的-o参数中添加即可

[root@salt-client01 conf]# diff confxml.xml confxml.xml.bak
24,25c24,25
<   <localpath watch="/data/">                          #data就是本地需要同步的文件夹到服务器端的目录
<       <remote ip="192.168.6.22" name="backup"/>       #data (server的模块名)是rsync 服务端的文件夹，也就是推送到服务器端的目标文件夹，可以配置多个，
---
>   <localpath watch="/opt/tongbu">
>       <remote ip="127.0.0.1" name="tongbu1"/>
31c31
<       <auth start="true" users="rsync_backup" passwordfile="/etc/rsync.password"/>   #true 才能生效，rsync_body同步时候虚拟账号，后面是密码文件
---
>       <auth start="false" users="root" passwordfile="/etc/rsync.pas"/>
33c33
<       <timeout start="true" time="100"/><!-- timeout=100 -->                    #true 才能生效
---
>       <timeout start="false" time="100"/><!-- timeout=100 -->
36c36
<   <failLog path="/usr/local/sersync/log/rsync_fail_log.sh" timeToExecute="60"/><!--default every 60mins execute once-->  #检测rsync进程判断，没有自动启
---
>   <failLog path="/tmp/rsync_fail_log.sh" timeToExecute="60"/><!--default every 60mins execute once-->
```

启动sersync

```
#声明环境变量
echo 'export PATH=$PATH:/opt/sersync/bin'>>/etc/profile 
source /etc/profile

[root@web02 log]# sersync -r -d -o /opt/sersync/conf/confxml.xml 
set the system param
execute：echo 50000000 > /proc/sys/fs/inotify/max_user_watches
execute：echo 327679 > /proc/sys/fs/inotify/max_queued_events
parse the command param
option: -r      rsync all the local files to the remote servers before the sersync work
option: -d      run as a daemon
option: -o      config xml name：  /opt/sersync/conf/confxml.xml
daemon thread num: 10
parse xml config file
host ip : localhost     host port: 8008
daemon start，sersync run behind the console 
use rsync password-file :
user is rsync_backup
passwordfile is         /etc/rsync.password
config xml parse success
please set /etc/rsyncd.conf max connections=0 Manually
sersync working thread 12  = 1(primary thread) + 1(fail retry thread) + 10(daemon sub threads) 
Max threads numbers is: 22 = 12(Thread pool nums) + 10(Sub threads)
please according your cpu ，use -n param to adjust the cpu rate
------------------------------------------
rsync the directory recursivly to the remote servers once
working please wait...
execute command: cd /backup && rsync -artuz -R --delete ./  --port=874 rsync_backup@192.168.6.22::backup --password-file=/etc/rsync.password >/dev/null 2>&1 
run the sersync: 
watch path is: /backup #此时可以看出sersync已经启动成功了
```

检测脚本

```shell
[root@web02 log]# cat rsync_log.sh 
#!/bin/bash
#Author:yzh
#xxx
#Time:2022-03-03 23:32:37
#Name:rsync_log.sh
#Version:V1.0
#Description:This is a production script.
sersync="/opt/sersync/bin/sersync"
conf_file="/opt/sersync/conf/confxml.xml"
status=$(ps aux|grep 'sersync'|grep -v 'grep'|wc -l)
if [ $status -eq 0  ]then;
    $sersync -d -r -o $conf_file &
else
    exit 0;
fi
```

```
#脚本写好以后，添加到计划任务中去
*/1 * * * * /bin/bash /opt/sersync/log/rsync_log.sh  > /dev/null 2>&1
```

测试同步

```

```

