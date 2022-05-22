## 介绍

我们所了解过的FTP文件传输，的确可以让不同主机之间进行文件传输，此方式特点是`传输文件`，用户想要在客户端直接修改服务器的数据，还是较为麻烦。

既然如此，Linux上有一款应用叫做Samba，是一个能让Linux系统应用微软网络通讯协议的软件。

微软为了解决局域网的文件共享，制定了SMB协议，也就是（Server Messages Block，服务器消息块），后来SMB通信协议应用到了Linux系统上，就形成了现在的`Samba软件`。

- Samba最大的功能就是可以用于Linux与windows系统直接的文件共享和打印共享
- Samba既可以用于windows与Linux之间的文件共享
- 也可以用于Linux与Linux之间的资源共享
- 由于NFS(网络文件系统）可以很好的完成Linux与Linux之间的数据共享
- 因而 Samba较多的用在了Linux与windows之间的数据共享上面。

## 安装Samba服务

```
#安装samba
[root@chaogelinux ftpdir]# yum install samba -y

#默认主配置文件
[root@chaogelinux ftpdir]# cat /etc/samba/smb.conf -n
```

## Samba配置文件核心参数

smb的配置文件如下

```
[root@chaogelinux ftpdir]# ls /etc/samba/
lmhosts  smb.conf  smb.conf.example
```

`smb.sonf.example`配置样例文件，里面有关于配置Samba服务器样例

smb的配置文件，主要分为`全局配置`和`共享配置`

- [global] 全局
- 共享
  - [home]
  - [printers]

【全局配置】

```
workgroup = MYGROUP
Samba服务器加入的工作组名，一个局域网内，必须有相同的工作组名。

server string = Samba Server Version %v
Samba服务器注释，可以不选，%v代表显示Samba版本号

netbios name = samba
主机NetBIOS名

netbios name = samba
主机NetBIOS名

interfaces = lo eth0
设置Samba服务器端监听网卡，可以写网卡名称或者IP地址

hosts allow/deny = 10.10.10.1 
允许连接到Samba server客户端IP，多个参数用空格分开。可以用一个IP表示，也可以用一个网段表示。

max connections = 0
用来指定连接Samba server服务器最大连接数如果操作则连接请求被拒绝。0表示不限制。

deadtime = 0
来设置断掉一个没有任何文件的链接时间。单位十分钟，0代表Samba server不自动断开任何连接

time server = yes/no
用来设置让nmdb成为Windows客户端的时间服务器

log file = /var/log/samba/%m.log
设置Samba server日志文件存储位置和日志名称。文件后面加一个%m（主机名），每个主机都会有一个主机名.log日志文件

max log size = 50
限制每个日志文件的最大容量为50KB，0代表不限制

Security = user
设置客户端访问Samba服务器的验证方式，Samba4版本已经不使用share和server方式，这里不介绍
1) user:Samba用户名和密码登录
2) domain：添加Samba服务器到N域，由NT与控制起来进行身份验证。域安全级别，使用主域控制器（PDC）来完成认证

passdb backend = tdbsam
后台管理用户密码方式
1）smbpasswd：该方式是使用smb自己的工具smbpasswd来给系统用户
2）tdbsam：该方式则是使用一个数据库文件来建立用户数据库。
3）ldapsam：该方式则是基于LDAP的账户管理方式来验证用户。

smb passwd file = /etc/samba/smbpasswd
用来定义samba用户的密码文件。smbpasswd文件如果没有那就要手工新建。

username map = /etc/samba/smbusers
用来定义用户名映射，比如可以将root换administrator、admin等。

guest account = nobody
用来设置guest用户名。

socket options = TCP_NODELAY SO_RCVBUF=8192 SO_SNDBUF=8192
用来设置服务器和客户端之间会话的Socket选项，可以优化传输速度

load printers = yes/no
设置是否在启动Samba时就共享打印机。
```

【共享参数】

```
comment = 任意字符串
comment是对该共享的描述，可以是任意字符串。

browseable = yes/no
browseable用来指定该共享是否可以浏览。

path = 共享目录路径
path用来指定共享目录的路径。

writable = yes/no
用来指定该共享路径是否可写

invalid users = 禁止访问该共享的用户

invalid users用来指定不允许访问该共享资源的用户。
例如：invalid users = root，@bob（多个用户或者组中间用逗号隔开。）

public = yes/no
用来指定该共享是否允许guest账户访问。

guest ok = yes/no
用来指定该共享是否允许guest账户访问。
```

## 配置共享资源

samba的配置文件，全局配置参数是针对整体的资源共享设置，生效于每一个独立的共享资源。

区域配置参数可以设置单独的共享资源，仅仅对该资源有效。

```
#修改smb.conf如下，添加以下参数
[chaoge]
comment = This is test configure
path = /home/chaoge
public = no
writable = yes
guest ok = yes
```

### pdbedit命令

pdbedit是samba的用户管理命令

语法

```
pdbedit -a username：新建Samba账户。

pdbedit -r username：修改Samba账户。

pdbedit -x username：删除Samba账户。

pdbedit -u, --user=USER      use username

pdbedit -L：列出Samba用户列表，读取passdb.tdb数据库文件。

pdbedit -Lv：列出Samba用户列表详细信息。

pdbedit -c “[D]” -u username：暂停该Samba用户账号。

pdbedit -c “[]” -u username：恢复该Samba用户账号。
```

创建用于访问samba共享资源的账户信息

**注意samba创建的用户数据库必须在当前系统中存在**

```
[root@chaogelinux samba]# id chaoge
uid=2002(chaoge) gid=2002(chaoge) 组=2002(chaoge)

[root@chaogelinux samba]# pdbedit -a -u chaoge
new password:
retype new password:
Unix username:        chaoge
NT username:
Account Flags:        [U          ]
User SID:             S-1-5-21-4265721185-3061822781-1370749960-1000
Primary Group SID:    S-1-5-21-4265721185-3061822781-1370749960-513
Full Name:
Home Directory:       \\chaogelinux\chaoge
HomeDir Drive:
Logon Script:
Profile Path:         \\chaogelinux\chaoge\profile
Domain:               CHAOGELINUX
Account desc:
Workstations:
Munged dial:
Logon time:           0
Logoff time:          三, 06 2月 2036 23:06:39 CST
Kickoff time:         三, 06 2月 2036 23:06:39 CST
Password last set:    三, 08 1月 2020 14:35:13 CST
Password can change:  三, 08 1月 2020 14:35:13 CST
Password must change: never
Last bad password   : 0
Bad password count  : 0
Logon hours         : FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
[root@chaogelinux samba]#
```

此时检查用于共享的资源目录，注意权限

```
[root@chaogelinux samba]# ll /home/chaoge/
总用量 8
-rw-rw-r-- 1 chaoge chaoge   31 11月 21 09:57 fine.txt
drwxr-xr-x 2 chaoge chaoge 4096 1月   8 09:24 超哥到此一游
```

重启smb服务，注意防火墙，是否允许smb的端口

```
systemctl restart smb

#检查端口运行情况
[root@chaogelinux ~]# netstat -tunlp|grep smb
tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN      2545/smbd
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN      2545/smbd
tcp6       0      0 :::445                  :::*                    LISTEN      2545/smbd
tcp6       0      0 :::139                  :::*                    LISTEN      2545/smbd
```

### 【此时进行远程连接】

win中选择【运行】填入smb服务器地址（\\ip地址）





