## vsftpd搭建ftp文本服务器

对于使用互联网的用户来说，首要目的就是获取资料，能够获取文件资料的方式有很多，其中一种就是文件传输，如今的互联网机器有各种型号、品牌，类型，如dell、惠普、浪潮、IBM、也分为个人PC、工作站、服务器、大型机、超级计算机等，并且还分为Windows、Linux、Unix、Mac等不同的操作系统。

为了能够在这么多样的机器之间传输文件，FTP（文件传输协议、File Transfer protocol）诞生了。

FTP是一种在互联网中进行文件传输的协议，基于C/S模式，默认服务端口号是20、21

20端口用于数据传输、21端口用于接收客户端的FTP命令与参数。

FTP服务器按照FTP协议在互联网上提供文件存储于文件访问的服务

FTP客户端用于向服务器索要资源

FTP工作模式主要分为两种

- 主动模式（PORT）：FTP服务器主动向客户端发起连接请求
- 被动模式（PASV）：FTP服务器等待客户端发起连接请求。

## 安装vsftpd服务

FTP是一种传输协议，实现了这种协议的工具，有一款Linux平台上的程序，名为vsftpd（ver secure ftp daemon，非常安全的FTP守护进程）

```
基于centos平台，直接yum安装
[root@chaogelinux ~]# yum install vsftpd -y

#注意关闭防火墙规则
iptables -F
```

## vsftpd配置文件

> 注意配置文件，不得有空格。否则会重启失败

```
[root@docker01 ~]# egrep -v "^#|^$" /etc/vsftpd/vsftpd.conf 
anonymous_enable=YES  #是否开启匿名用户允许访问
local_enable=YES    #是否允许本地用户登录FTP
write_enable=YES    #write_enable=YES #全局设置，是否容许写入，开启允许上传的权限
local_umask=022     #本地用户上传文件的umask
dirmessage_enable=YES    #允许为目录配置显示信息,显示每个目录下面的message_file文件的内容
xferlog_enable=YES    #开启日志功能，以及存放路径
xferlog_file=/var/log/vsftpd.log  #日志路径
connect_from_port_20=YES    #使用20端口进行连接
xferlog_std_format=YES    #标准日志格式
listen=YES        #绑定到监听端口
listen_ipv6=YES    #开启ipv6
pam_service_name=vsftpd    #设置PAM的名称
userlist_enable=YES    #设置用户已列表，允许或是禁止
tcp_wrappers=YES    #控制主机访问，检查/etc/hosts.allow  hosts.deny的配置达到防火墙作用
```

## vsftpd服务程序

vsftpd允许用户三种认证的模式登录到FTP服务器。

- 本地用户模式，基于Linux本地账号密码进行认证，配置简单，但是一旦被破解，服务器信息就很危险
- 匿名用户模式，任何人无需密码直接登录
- 虚拟用户模式，单独为FTP创建用户数据库，基于口令验证账户信息，只适用于FTP，不会影响其他用户信息，最为安全。

## 安装ftp客户端

ftp客户端有多种形式，图形化、命令行

- ftp命令客户端

```
ftp> ascii  # 设定以ASCII方式传送文件(缺省值) 
ftp> bell   # 每完成一次文件传送,报警提示. 
ftp> binary # 设定以二进制方式传送文件. 
ftp> bye    # 终止主机FTP进程,并退出FTP管理方式. 
ftp> case   # 当为ON时,用MGET命令拷贝的文件名到本地机器中,全部转换为小写字母. 
ftp> cd     # 同UNIX的CD命令. 
ftp> cdup   # 返回上一级目录. 
ftp> chmod  # 改变远端主机的文件权限. 
ftp> close  # 终止远端的FTP进程,返回到FTP命令状态, 所有的宏定义都被删除. 
ftp> delete # 删除远端主机中的文件. 
ftp> dir [remote-directory] [local-file] # 列出当前远端主机目录中的文件.如果有本地文件,就将结果写至本地文件. 
ftp> get [remote-file] [local-file] # 从远端主机中传送至本地主机中. 
ftp> help [command] # 输出命令的解释. 
ftp> lcd # 改变当前本地主机的工作目录,如果缺省,就转到当前用户的HOME目录. 
ftp> ls [remote-directory] [local-file] # 同DIR. 
ftp> macdef                 # 定义宏命令. 
ftp> mdelete [remote-files] # 删除一批文件. 
ftp> mget [remote-files]    # 从远端主机接收一批文件至本地主机. 
ftp> mkdir directory-name   # 在远端主机中建立目录. 
ftp> mput local-files # 将本地主机中一批文件传送至远端主机. 
ftp> open host [port] # 重新建立一个新的连接. 
ftp> prompt           # 交互提示模式. 
ftp> put local-file [remote-file] # 将本地一个文件传送至远端主机中. 
ftp> pwd  # 列出当前远端主机目录. 
ftp> quit # 同BYE. 
ftp> recv remote-file [local-file] # 同GET. 
ftp> rename [from] [to]     # 改变远端主机中的文件名. 
ftp> rmdir directory-name   # 删除远端主机中的目录. 
ftp> send local-file [remote-file] # 同PUT. 
ftp> status   # 显示当前FTP的状态. 
ftp> system   # 显示远端主机系统类型. 
ftp> user user-name [password] [account] # 重新以别的用户名登录远端主机. 
ftp> ? [command] # 同HELP. [command]指定需要帮助的命令名称。如果没有指定 command，ftp 将显示全部命令的列表。
ftp> ! # 从 ftp 子系统退出到外壳。
```

> FileZilla图形化工具

```
安装ftp命令行
yum install ftp -y
```

## 匿名用户模式

匿名用户模式是最不安全的方式，一般用在访问不重要的，允许公开的文件，且放在企业内网环境中，置于防火墙规则下，保证基本的安全性。

vsftpd默认开启了匿名用户模式，修改配置文件，定义匿名用户的权限，如下

```
[root@docker01 ~]# grep '^anon' /etc/vsftpd/vsftpd.conf 
anonymous_enable=YES   #允许匿名访问
anon_upload_enable=YES  #允许匿名用户上传
anon_mkdir_write_enable=YES   #允许匿名用户创建目录
anon_other_write_enable=YES   #允许匿名用户修改目录
```

重启服务，且加载开机自启

```
[root@chaogelinux ~]# systemctl restart vsftpd  #重启服务
[root@chaogelinux ~]# systemctl enable vsftpd        #开启自启
```

此时可以使用ftp命令连接到FTP服务器了。

连接了FTP服务端，其实连接的是目录`/var/ftp/`

```
[root@chaogelinux ftp]# pwd
/var/ftp
[root@chaogelinux ftp]# ls
pub
```

使用ftp客户端命令，连接ftp服务端

```
[root@chaogelinux ~]# ftp 123.206.16.61        #连接ftp服务器的ip地址
Connected to 10.141.32.137 (10.141.32.137).
220 (vsFTPd 3.0.2)
Name (10.141.32.137:root): anonymous        #填入默认的账号
331 Please specify the password.
Password:                                  #密码为空，直接回车
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>

ftp> ls    
227 Entering Passive Mode (10,141,32,137,98,195).
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Oct 30  2018 pub
226 Directory send OK.
ftp> cd pub        #切换工作目录
250 Directory successfully changed.
ftp> mkdir chaoge                #发现在这里创建文件夹报错了
550 Create directory operation failed.
```

由于我们使用匿名用户登录ftp，默认访问的是/var/ftp文件夹，我们来检查下文件夹权限

```
[root@chaogelinux ftp]# pwd
/var/ftp
[root@chaogelinux ftp]# ll
总用量 4
drwxr-xr-x 2 root root 4096 10月 31 2018 pub
```

由于pub文件夹，属于root用户，因此ftp无法向其写入内容，可以修改文件夹的user、group

```
#检查系统用户ftp
[root@chaogelinux ftp]# id ftp
uid=14(ftp) gid=50(ftp) 组=50(ftp)

#更改pub的属主
[root@chaogelinux ftp]# chown -Rf ftp /var/ftp/pub/        #递归处理所有的文件及子目录  去除错误信息
[root@chaogelinux ftp]# ll
总用量 4
drwxr-xr-x 2 ftp root 4096 10月 31 2018 pub
```

再次登录ftp，写入数据

```
yumac: ~ yuchao$ftp 123.206.16.61
Connected to 123.206.16.61.
220 (vsFTPd 3.0.2)
Name (123.206.16.61:yuchao): anonymous
331 Please specify the password.
Password:
230 Login successful.
ftp> cd pub                #切换目录
ftp> mkdir chaoge666        #创建文件夹
257 "/pub/chaoge666" created
ftp> rename chaoge666 chaoge888    #重命名文件夹
350 Ready for RNTO.
250 Rename successful.
ftp> rmdir chaoge888        #删除文件夹
250 Remove directory operation successful.
```

此时成功写入了ftp文件夹数据，服务器上检查文件信息

```
[root@chaogelinux pub]# pwd
/var/ftp/pub
[root@chaogelinux pub]# ls
chaoge666
[root@chaogelinux pub]# ls
chaoge888
```

## 本地用户模式

使用Linux本地用户模式，比匿名用户来的安全，修改配置文件，关闭匿名模式，开启本地用户模式

相关配置文件

```
[root@chaogelinux pub]# ls /etc/vsftpd/
ftpusers  user_list  vsftpd.conf  vsftpd_conf_migrate.sh
```

修改配置文件如下/etc/vsftpd/vsftpd.conf

```
anonymous_enable=NO    #关闭匿名用户模式
local_enable=YES    #开启本地用户模式
write_enable=YES    #设置可写权限
local_umask=022    #文件umask值
userlist_enable=YES    #启用【禁止登录的用户名单】文件名是，ftpusers，user_list
userlist_deny=YES    #开启禁止登录名单
```

重启vsftpd服务，加载配置

```
systemctl restart vsftpd
systemctl enable vsftpd
```

【此时可以使用ftp客户端连接了，使用linux本地用户信息】

假设当前服务器上用户管理中存在一个用户`chaoge`

```
[root@chaogelinux ~]# grep 'chaoge' /etc/passwd
chaoge:x:2002:2002::/home/chaoge:/bin/bash
```

此时客户端可以使用此用户连接FTP，默认访问的是该用户的家目录，也就是`/home/chaoge文件夹内容`

```
yumac: ~ yuchao$ftp 123.206.16.61
Connected to 123.206.16.61.
220 (vsFTPd 3.0.2)
Name (123.206.16.61:yuchao): chaoge
331 Please specify the password.
Password:
230 Login successful.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 2002     2002           31 Nov 21 01:57 fine.txt
226 Directory send OK.    
ftp> mkdir 超哥到此一游            #新建文件夹
257 "/home/chaoge/超哥到此一游" created
```

有些用户是无法登录ftp，是因为vsftp有一个名单，写上了禁止谁登录

```
ftpusers ， user_list
[root@chaogelinux vsftpd]# pwd
/etc/vsftpd
[root@chaogelinux vsftpd]# ls
ftpusers  user_list  vsftpd.conf  vsftpd_conf_migrate.sh

#文件中写入的名字，都是禁止登录的
[root@chaogelinux vsftpd]# cat ftpusers
# Users that are not allowed to login via ftp
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
news
uucp
operator
games
nobody

####如此，禁止登录的用户
yumac: ~ yuchao$ftp 123.206.16.61
Connected to 123.206.16.61.
220 (vsFTPd 3.0.2)
Name (123.206.16.61:yuchao): root
530 Permission denied.
ftp: Login failed.
```

## 虚拟用户模式

了解了有匿名用户、本地用户、再来了解下虚拟用户模式，顾名思义是虚拟创建出的用户，也是最为安全的一种。

首先安装Berkeley DB工具

```
yum install db4 db4-utils -y
```

创建用于进行FTP认证的用户数据库，奇数行账户名、偶数行是密码

```
[root@chaogelinux vsftpd]# cat ftp_user.txt
chaoge
666
pyyu
888
```

由于这样的明文信息很不安全，vsftpd也无法加载该格式的数据，因此还得用db_load命令对`ftp_user.txt`文件数据加密，且设置权限使得普通用户无权限查阅。

```
#创建加密文件  -T 和-t参数必须加上，用于转化普通文本为vsftpd识别的数据库文件
[root@chaogelinux vsftpd]# db_load -T -t hash -f /etc/vsftpd/ftp_user.txt /etc/vsftpd/ftp_user.db

#检查文件属性
[root@chaogelinux vsftpd]# file ftp_user.db
ftp_user.db: Berkeley DB (Hash, version 9, native byte-order)

#降低文件权限
[root@chaogelinux vsftpd]# ll ftp_user.db
-rw-r--r-- 1 root root 12288 1月   8 09:36 ftp_user.db
[root@chaogelinux vsftpd]# chmod 600 ftp_user.db
[root@chaogelinux vsftpd]# ll ftp_user.db
-rw------- 1 root root 12288 1月   8 09:36 ftp_user.db

#删除旧的数据文件
[root@chaogelinux vsftpd]# rm -rf ftp_user.txt
```

创建当虚拟用户登录后默认访问的文件夹路径，且和linux中的一个本地用户做一个映射关系，防止`匿名用户登录后，创建了文件夹，但是系统没有此用户报错权限问题`

```
#新建一个用户，指定用户家目录，且禁止登录
useradd -d /var/ftpdir -s /sbin/nologin virtual_chao

#检查此ftpdir的属性
[root@chaogelinux vsftpd]# ls -ld /var/ftpdir/
drwx------ 2 virtual_chao virtual_chao 4096 1月   8 09:46 /var/ftpdir/

#更改文件夹权限
[root@chaogelinux vsftpd]# chmod -Rf 755 /var/ftpdir/

#添加virtual_chao用户至ftpusers禁止用户登录文件，增大系统安全，但是不会影响虚拟用户登录
[root@chaogelinux vsftpd]# echo 'virtual_chao' >> ftpusers
```

此时需要创建支持虚拟用户的PAM文件，PAM是一组安全机制的模块，编辑认证文件`/etc/pam.d/vsftpd`

```
#注释掉文中所有语句，添加以下2句，注意db参数指定的是db_load生成的文件路径，无需添加后缀

[root@chaogelinux pam.d]# cat /etc/pam.d/vsftpd
auth required pam_userdb.so db=/etc/vsftpd/ftp_user
account required pam_userdb.so db=/etc/vsftpd/ftp_user
```

最后修改vsftpd配置文件

```
[root@chaogelinux pam.d]# grep -Ev '^$|^#' /etc/vsftpd/vsftpd.conf
anonymous_enable=NO     #禁止匿名模式
local_enable=YES        #允许本地用户
write_enable=YES    
local_umask=022
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd        #指定PAM认证文件
userlist_enable=YES
userlist_deny=YES
tcp_wrappers=YES
guest_enable=YES        #开启虚拟用户
guest_username=virtual_chao        #指定虚拟用户账号
allow_writeable_chroot=YES      #如果用户被限制只能在其家目录，允许用户可以对家目录写入数据
```

【针对不同的虚拟用户设置不同的权限】

用户ftp认证的虚拟账号，ftp_user.txt

- chaoge，允许上传，新建，修改，查看，删除权限
- pyyu，只读权限

这样的需求可以通过vsftpd配置实现，定义`user_config_dir`参数实现不同的虚拟用户，不同的权限配置

```
#新建管理虚拟用户权限的文件夹
mkdir /etc/vsftpd/virtual_user_dir

#进入文件夹，创建权限的配置文件
[root@chaogelinux vsftpd]# cd /etc/vsftpd/virtual_user_dir/
[root@chaogelinux virtual_user_dir]# cat chaoge
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES

#再创建一个pyyu文件，禁止写入
[root@chaogelinux virtual_user_dir]# pwd
/etc/vsftpd/virtual_user_dir
[root@chaogelinux virtual_user_dir]# ls
chaoge  pyyu
[root@chaogelinux virtual_user_dir]# cat pyyu
anon_upload_enable=NO
anon_mkdir_write_enable=NO
anon_other_write_enable=NO
```

编辑vsftpd主配置文件，添加如下一行

```
cat /etc/vsftpd/vsftpd.conf
user_config_dir=/etc/vsftpd/virtual_user_dir
```

重启vsftpd服务，注意配置文件，不得有任何莫名其妙的空格！否则会重启失败

```
systemctl restart vsftpd
```

【此时使用客户端登录用户chaoge，进行读写】

```
yumac: ~ yuchao$ftp 123.206.16.61
Connected to 123.206.16.61.
220 (vsFTPd 3.0.2)
Name (123.206.16.61:yuchao): chaoge    #用匿名用户登录chaoge
331 Please specify the password.
Password:
230 Login successful.
ftp> ls            #此时看到的是匿名用户映射的家目录内容
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0               0 Jan 08 02:49 haha
226 Directory send OK.
ftp> mkdir 超哥到此一游        #chaoge有权限增加文件
257 "/超哥到此一游" created

#####检查服务器上的FTP目录
[root@chaogelinux ftpdir]# pwd
/var/ftpdir
[root@chaogelinux ftpdir]# ls
haha  超哥到此一游
```

【使用pyyu用户登录ftp服务端，查看是否能够读写】

```
yumac: ~ yuchao$ftp 123.206.16.61
Connected to 123.206.16.61.
220 (vsFTPd 3.0.2)
Name (123.206.16.61:yuchao): pyyu
331 Please specify the password.
Password:
230 Login successful.
ftp> mkdir pyyu也想到此一游                #无法写入数据，只能读取
550 Permission denied.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0               0 Jan 08 02:49 haha
drwx------    2 2003     2003         4096 Jan 08 02:50 超哥到此一游
226 Directory send OK.
```