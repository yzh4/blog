## 创建组和用户

```
[root@localhost ~]# groupadd oinstall
[root@localhost ~]# groupadd dba
[root@localhost ~]# groupadd asmdba
[root@localhost ~]# groupadd backupdba
[root@localhost ~]# groupadd dgdba
[root@localhost ~]# groupadd kmdba
[root@localhost ~]# groupadd racdba
[root@localhost ~]# groupadd oper

[root@localhost ~]# useradd -g oinstall -G dba,asmdba,backupdba,dgdba,kmdba,racdba,oper oracle19

[root@localhost ~]# passwd oracle19
Changing password for user oracle19.
New password:
BAD PASSWORD: The password contains the user name in some form
Retype new password:
passwd: all authentication tokens updated successfully.
[root@localhost ~]#

 

[root@localhost ~]# mkdir -p /u01/app/oracle19/product/19.2.0/db_1
[root@localhost ~]# chown -R oracle19:oinstall /u01/app/oracle19/

[root@localhost ~]# su - oracle19
[oracle19@localhost ~]$ vi .bash_profile

export ORACLE_HOME=/u01/app/oracle19/product/19.2.0/db_1
export PATH=$PATH:/u01/app/oracle19/product/19.2.0/db_1/bin
export ORACLE_SID=orcl

[oracle19@localhost ~]$ source .bash_profile

[oracle19@localhost db_1]$ pwd
/u01/app/oracle19/product/19.2.0/db_1
[oracle19@localhost db_1]$ ll -h
total 2.9G
-rw-rw-r--. 1 oracle19 oinstall 2.9G Jun 13 19:22 LINUX.X64_193000_db_home.zip
[oracle19@localhost db_1]$
```



```
[oracle19@localhost db_1]$ unzip LINUX.X64_193000_db_home.zip
```

![img](https://img-blog.csdnimg.cn/20210614114215295.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29rY19jaGFtcGlvbg==,size_16,color_FFFFFF,t_70)

 

## 禁用防火墙

systemctl stop firewalld.service

systemctl disable firewalld.service

systemctl status firewalld.service