## oracle19c安装部署

```perl
一、关闭防火墙
#关闭防火墙
systemctl stop firewalld.service
#关闭操作系统自启动
systemctl disable firewalld.service
#检查关闭情况
systemctl status firewalld.service  #dead表示未开启开机启动；inactive表示现在的状态是关闭

二、关闭SELINUX
vim /etc/selinux/config
SELINUX=disabled
SELINUXTYPE=targeted

三、关闭NUMA
#查看default 的grub 的entry
grubby --default-kernel
#查看default grub的具体信息
grubby --info `grubby --default-kernel`
#更新args   添加numa=off的参数
grubby --args=numa=off --update-kernel `grubby --default-kernel`
#重启服务器
reboot
# 确认numa=off加入default grub中
grubby --info `grubby --default-kernel`
grep -i numa /var/log/dmesg

四、创建用户
#创建oracle用户
groupadd oinstall
groupadd dba
useradd -g oinstall -G dba oracle   
passwd oracle ----修改oracle用户密码(oracle)(1234)
# oracle用户下获取组id

#用root用户执行， dba组的ID加入到配置文件中：
echo 1001 > /proc/sys/vm/hugetlb_shm_group

五、设置系统参数
#设置内核参数
vim /etc/sysctl.conf
kernel.shmall = physical RAM size / pagesize For most systems
kernel.shmmax = 1/2 of physical RAM
fs.file-max = 6815744
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
fs.aio-max-nr = 1048576
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
#执行sysctl -p 命令使以上设置生效 

#修改系统参数
#编辑/etc/pam.d/login 添加如下内容：
vim /etc/pam.d/login
session    required     pam_limits.so

#编辑/etc/profile添加如下内容：   #系统环境变量
vim /etc/profile
if [ /$USER = "oracle" ] ; then
    if [ /$SHELL = "/bin/ksh" ]; then
        ulimit -p 16384
        ulimit -n 65536
    else
        ulimit -u 16384 -n 65536
    fi
    umask 022
fi
#使修改生效
source /etc/profile


#编辑/etc/security/limits.conf 添加如下内容：
vim /etc/security/limits.conf 
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536

六、配置环境变量(内网配置)
#挂载本地镜像源
#创建目录
mkdir -p /oracle/app/product/19.3.0/db_1
chown -R oracle:oinstall /oracle
#挂载前，将原有的repo文件备份，新建挂载yum源的repo文件
cd /etc/yum.repos.d
mv CentOS-Base.repo epel.repo.bak
vim client.repo
#repo内容
[base]
name=CentOS-$releasever
enabled=1
failovermethod=priority
baseurl=file:///mnt/iso/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
#挂载yum源
#创建路径
mkdir /mnt/iso
#切换到iso镜像文件路径
cd /root/
#挂载
mount CentOS-7-x86_64-DVD-2009.iso /mnt/iso/

#挂载数据盘（/dev/vdb）
fdisk -l
fdisk /dev/vdb
n
p
回车
回车
回车
w
mkfs.ext4 /dev/vdb
mount /dev/vdb /opt/oradata/
df
#查看uuid
[root@iZsp7010qf4k0dwkuitbzpZ oracle]# blkid
/dev/vda1: UUID="795c9f31-f638-4308-8fab-4a7d5c606beb" TYPE="ext4" 
/dev/vdb: UUID="e677332c-1e87-4e06-9c39-070106d269c7" TYPE="ext4"
#填写uuid
vim /etc/fstab
UUID=e677332c-1e87-4e06-9c39-070106d269c7 /opt/oradata            ext4    defaults        1 1
mount -a
df -h
#挂载数据盘后授权
chown -R oracle:oinstall /opt/oradata/

#配置环境变量，创建目录
vim ~/.bash_profile        #当前用户环境变量
oracle用户：
export ORACLE_BASE=/oracle/app
export ORACLE_HOME=/oracle/app/product/19.3.0/db_1
export ORACLE_SID=oggdb
export PATH=/usr/sbin:$ORACLE_HOME/bin:$PATH
export JAVA_OPTS=-Djava.awt.headless=true
#使之生效source ~/.bash_profile

七、安装依赖包
yum install -y bc binutils compat-libcap1 compat-libstdc++33 elfutils-libelf elfutils-libelf-devel fontconfig-devel glibc glibc-devel ksh libaio libaio-devel libX11 libXau libXi libXtst libXrender libXrender-devel libgcc libstdc++ libstdc++-devel libxcb make smartmontools sysstat kmod* gcc-c* zip unzip telnet* xorg-x11-* xterm --skip-broken
#xterm 图形化安装界面
yum -y install compat-libstdc++-33-3.2.3-72.el7.x86_64.rpm
 
八、改变Swap交换空间大小
#检查当前的分区情况
free
free -g
#增加交换分区文件及大小，如果要增加2G大小的交换分区，则命令写法如下，其中的 count 等于想要的块大小。
dd if=/dev/zero of=/home/swap bs=1024 count=16384000
#设置交换文件
mkswap /home/swap
#立即启用交换分区文件
swapon /home/swap
#引导时自动启用，编辑 /etc/fstab 文件
/home/swap                                swap                    swap    defaults        0 0

九、安装Oracle
# 解压安装包
mkdir -p /oracle/app/product/19.3.0/db_1/
unzip LINUX.X64_193000_db_home.zip -d /oracle/app/product/19.3.0/db_1/
chown -R oracle:oinstall /oracle

#配置图形化
使用xstart新建连接，填写服务器信息（主机、协议ssh、设置端口22、用户名、命令）
#命令
/usr/bin/xterm -ls -display $DISPLAY
#保存运行
#切换至oracle安装路径，安装
cd /oracle/app/product/19.3.0/db_1&&./runInstaller
#安装oracle图形化界面出现方框乱码
#点击右下角先取消安装
#在xstart执行命令
export LANG=en_US.UTF-8
#重新./runInstaller安装
#图形化安装界面，完成安装前提示修改系统参数（kernel.shmall、kernel.shmmax值）
路径：/etc/sysctl.conf
kernel.shmall = 图形化指定值
kernel.shmmax = 图形化指定值
#保存后，使之生效sysctl -p
#图形化界面点击checkagain




十、修改监听端口
#停止监听
lsnrctl stop
#修改参数配置
#“host”即为命令前“oracle@”后的内容,例：iZsp7010qf4k0dwkuitbzpZ
#连接数据库
sqlplus /nolog
SQL> conn / as sysdba
SQL>alter system set local_listener = "(address=(protocol=tcp)(host=10.0.0.70)(port=11521))";
#修改ora文件
vim /oracle/app/product/19.3.0/db_1/network/admin/listener.ora
#端口1521修改为11521
vim /oracle/app/product/19.3.0/db_1/network/admin/tnsnames.ora
#端口1521修改为11521
#启动监听
lsnrctl start
#查看oracle进程
netstat -apn|grep 11521


十一、配置plsql远程连接
需要找到对应版本的oci.dll文件，我这个是19c


```

