## MySQL8.0部署

1.mysql二进制安装包下载地址

 https://dev.mysql.com/downloads/mysql/

2.查看系统⾃带的 `mariadb`

```
rpm -qa|grep mariadb
```

3.卸载系统⾃带的 `MARIADB`（如果有）,remove后面加 `rpm -qa|grep mariadb` 查询出来的路径

```arduino
yum -y remove
```

将 `mysql-8.0.26-linux-glibc2.12-x86_64.tar.xz` 安装包上传到 `/usr/local/` 目录下

安装包解压,并重命名为mysql

```
[root@localhost local]# tar -Jxf mysql-8.0.16-linux-glibc2.12-x86_64.tar.xz 
[root@localhost local]# mv mysql-8.0.16-linux-glibc2.12-x86_64 mysql
```

创建MYSQL⽤户和⽤户组

```
[root@localhost local]# groupadd mysql
[root@localhost local]# useradd -g mysql mysql

```

修改MYSQL⽬录的归属⽤户

```
chown -R mysql:mysql ./mysql
```

准备MYSQL的配置⽂件,编辑 `vim /etc/my.conf`

```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
socket=/var/lib/mysql/mysql.sock
[mysqld]
skip-name-resolve
#设置3306端⼝
port = 3306
socket=/var/lib/mysql/mysql.sock
# 设置mysql的安装⽬录
basedir=/usr/local/mysql
# 设置mysql数据库的数据的存放⽬录
datadir=/usr/local/mysql/data
# 允许最⼤连接数
max_connections=200
# 服务端使⽤的字符集默认为8⽐特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使⽤的默认存储引擎
default-storage-engine=INNODB
lower_case_table_names=1
max_allowed_packet=16M
# binglog配置
server-id = 1
log-bin = mysql-bin
log_bin_index = binlog.index
binlog_format = ROW
# binlog过期清理时间；
expire_logs_days= 7
# binlog每个日志文件大小；
max_binlog_size = 1024m
# binlog缓存大小；
binlog_cache_size = 128m
# 最大binlog缓存大小。
max_binlog_cache_size = 1024m
```

并且修改权限

```
[root@localhost local]# mkdir /var/lib/mysql
[root@localhost local]# chmod 777 /var/lib/mysql/
```

进入`cd /usr/local/mysql` 目录 进行初始化

```awk
./bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```

执行初始化后,控制台会返回临时密码, `记录临时密码` ,后面会用到

`7r.D&tjoZIte`

```
2022-03-05T20:09:12.094963Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: 7r.D&tjoZIte
```

复制启动脚本到资源目录

```awk
cp ./support-files/mysql.server /etc/init.d/mysqld
```

编辑`vim /etc/init.d/mysqld `,修改 `basedir` 和 `datadir` ,为其实际对应的目录

```awk
basedir=/usr/local/mysql 
datadir=/usr/local/mysql/data
```

增加 mysqld 服务控制脚本执⾏权限

```awk
chmod +x /etc/init.d/mysqld
```

启动

```
/etc/init.d/mysqld start
```

配置环境变量

```
[root@localhost mysql]# echo 'export PATH=$PATH:/usr/local/mysql/bin' > /etc/profile

source /etc/profile
```

登录mysql并且修改密码

```
mysql -uroot -p
```

输入刚才记录的 `临时密码`

修改 `Root` 账户密码,并刷新权限

```pgsql
alter user user() identified by "root";
flush privileges;
```

设置远程主机登录

```pgsql
use mysql;
update user set user.Host='%' where user.User='root';
flush privileges;
```

### 忘记 `mysql` 密码, 找回密码

停止 `mysql` 服务

```arduino
 service mysqld stop
```

修改 `mysql` 的配置文件 `my.cnf`
`my.cnf` 配置文件的位置，一般在 `/etc/my.cnf` ，有些版本在 `/etc/mysql/my.cnf`
在配置文件中添加下面代码, `安全模式启动`

```sql
[mysqld]
skip-grant-tables
```

启动 `mysql` 服务

```crmsh
service mysqld start
```

进入 `mysql`, 这样可以不用输入密码进入 `mysql`

```ebnf
mysql -u root 
```

修改密码, 并刷新权限;

```pgsql
mysql> use mysql;
Database changed
mysql> update user set authentication_string="123456" where user="root";
Query OK, 0 rows affected (0.00 sec)
Rows matched: 1  Changed: 0  Warnings: 0
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

将刚才新添加的配置文件 `my.cnf` 删除这两行

```sql
[mysqld]
skip-grant-tables
```

重新启动 `mysql` 服务

```crmsh
service mysqld start
```

`MYSQL 8.0` 默认user表

```gherkin
mysql> select host, user, authentication_string, plugin from user;
+-----------+------------------+------------------------------------------------------------------------+-----------------------+
| host      | user             | authentication_string                                                  | plugin                |
+-----------+------------------+------------------------------------------------------------------------+-----------------------+
| %         | root             | $A$005$_N6
HbERn<n&lg8k*XmaW25XLFVs8MDOgM1K18egNm1p87ttKMQ9Awcf0/rUA  | caching_sha2_password |
| localhost | mysql.infoschema | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | caching_sha2_password |
| localhost | mysql.session    | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | caching_sha2_password |
| localhost | mysql.sys        | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | caching_sha2_password |
+-----------+------------------+------------------------------------------------------------------------+-----------------------+
```

### Access denied for user 'root'@'localhost' (using password: YES)

安全模式启动 `mysql服务`,先将密码设置为空

```routeros
use mysql;
update user set authentication_string='' where user='root';
```

在重新设置密码, 刷新权限

```pgsql
use mysql;
alter user user() identified by "root";
flush privileges;
```