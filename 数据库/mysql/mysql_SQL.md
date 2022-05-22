## MySQl核心SQL语句

SQL，英文全称为Structured Query Language，中文意思是结构化查询语言，它是一种对关系型数据库中的数据进行定义和操作的语言，是大多数关系型数据库管理系统所支持的工业标准语言。

## SQL的分类

### DQL数据查询语言

> DQL，全称为Data Query Language，其语句也称为“数据检索语句”，作用是`从表中获取数据，确定数据应怎样在应用程序中给出。`

关键字：

- select(sql最常用)
- where
- order by
- group by
- having

```sql
#登录数据库
[root@sql ~]# mysql -p1234  -S /data/3306/mysql.sock 

# 这句SQL翻译是从mysql库下的user表，查询user、host字段的数据、且对user排序
mysql> select user,host from mysql.user order  by user;
+------+-------------+
| user | host        |
+------+-------------+
| root | localhost   |
| root | 127.0.0.1   |
| root | %           |
| yzh  | 192.168.6.% |
+------+-------------+
4 rows in set (0.01 sec)


```

### DML数据操作语言

> DML，全称为Data Manipulation Language，中文为数据操作语言。

关键字：

- insert
- update
- delete

```sql
# 删除这些无用的用户
# 发现删除了2个空用户
mysql> delete from mysql.user where user=' ';
Query OK, 2 rows affected (0.00 sec)

mysql> delete from mysql.user where user='root' and host='mysql-server56';
Query OK, 1 row affected (0.00 sec)

mysql> delete from mysql.user where user='root' and host='::1';
Query OK, 1 row affected (0.00 sec)

# 剩下来的用户
mysql> select user,host from mysql.user order by user;
+------+-----------+
| user | host      |
+------+-----------+
| root | localhost |
| root | 127.0.0.1 |
| root | %         |
+------+-----------+
3 rows in set (0.00 sec)
```

### TPL事务

> 全称为Transaction Processing Language，TPL语句用于确保被DML语句影响的表的所有行能够及时得到更新。

关键字：

- begin
- transaction
- commit
- rollback

### DCL授权控制

> 全称为Data Control Language

关键字：

- grant
- revoke

来进行授权用户许可，确定单个用户和用户组对数据库对象的访问。

```sql
mysql> create user yzh@'%' identified by '123456';
Query OK, 0 rows affected (0.01 sec)

# 普通用户被创建的时候，默认有USAGE权限，只能用于登录数据库，无其他权限
mysql> show grants for yzh@'%';
+----------------------------------------------------------------------------------------------------+
| Grants for yzh@%                                                                                   |
+----------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'yzh'@'%' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
+----------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

# 只给yzh用户查看kings数据库的查询权限
mysql> grant select on kings.* to yzh@'%' with grant option;
Query OK, 0 rows affected (0.01 sec)

#查询该用户权限
mysql> show grants for yzh@'%'
    -> ;
+----------------------------------------------------------------------------------------------------+
| Grants for yzh@%                                                                                   |
+----------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'yzh'@'%' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
| GRANT SELECT ON `kings`.* TO 'yzh'@'%' WITH GRANT OPTION                                           |
+----------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)
```

### DDL数据定义

> 全称为Data Definition Language

关键字：

- create
- drop
- alter

可使用该语言在数据库中创建新库表或删除库表，或者为表添加字段、索引等。

```sql
# 这句SQL意为创建lol数据库，且设置数据库编码为utf8，且大小写不敏感，a和A一样处理
mysql> create database if not exists lol default charset utf8 collate utf8_general_ci; 
Query OK, 1 row affected (0.00 sec)
```

### CCL指针控制语言

> CCL，全称为CURSOR Control Language

关键字：

- declare
- cursor
- fetch into
- update
- where current

用于对一个或多个表的单独进行操作

## 运维和开发--SQL

> 运维主要关心数据库的架构维护、数据备份、基础数据的管理、但是一般不会修改数据表的结构

主要以DDL类别工作为主，也就是数据定义，运维和开发都得掌握

- create
- alter
- drop

查看DDL语句具体信息

```sql
mysql> ? Data Definition
```

DCL数据授权控制，一般是运维操作的多些

- grant
- revoke
- commit
- Rollback

```sql
mysql> ? Account Management
```

运维要熟悉DML数据操作语言，需要熟悉查询数据操作。

DML语句

- select
- insert
- delete
- update

```sql
mysql> ? Data Manipulation
```

## 查看帮助

```
mysql> ? CREATE DATABASE
```

## SQL语法解析原理

输入mysql链接命令，连接成功后，输入SQL，这期间底层发生了什么？

大致是：

> 1.应用程序连接mysql，建立连接
>
> 2.SQL进行解析
>
> 3.进入存储引擎找数据，在磁盘、内存中读取数据，依次返回
>
> 4.数据返回给用户

### SQL解析流程

- 连接层

  - 应用程序（php，python，代码）连接mysql时，首先会进过连接池（创建数据库连接是很耗时且消耗资源的，数据库会提供连接池，保持连接，允许应用程序重复使用一个现有的连接，无须重复新建，省资源，减轻数据库压力），建立连接后，进入SQL解析层

- SQL层

  - 这一层是解析SQL语句，首先判断SQL正确性，是否符合DDL、DML、DCL语句规则

  - 根据不同的类型，命令分发模块转发给对应的模块处理

    - 例如接收的是select语句，就是DML查询，既然查询，就会去查找是否有缓存

      - 有缓存则直接返回给应用程序

      - 如果没有缓存，就进入了SQL的解析流程

        ```
        (Parsing queries)
        ```

        - 解析这件事由Parser解析器来对SQL进行词法分析（因为SQL可以是一个组合的，复杂、较长的语句），最终分析出一个或者多个SQL语句的执行计划

  - 得到执行计划后，还不会立即执行，因为解析器可能给出了SQL的多种执行方式，还要再进一步的判断，怎么执行才是最高效的。

    - mysql内部有一个查询优化器（Optimizer）根据自身的算法，找到一个最高效的方式执行SQL（例如有合理索引的那一条SQL）

  - 当优化器确定了执行计划，是不是就可以立即执行了?不是。。（大家可能回郁闷了，超哥你这么墨迹呢？）因为即使你确定了SQL可以执行了，但是你是不是有权限执行，对把。

  - 在SQL层内部对权限也检查后，SQL语句终于执行了，最后把执行后的结果，交给最底层的存储引擎接口（发动机），存储引擎和操作系统交互，读取到磁盘上的数据（发动机喝油开始干活了）。

  - 最终SQL执行完毕，本次获取的数据，常规下更新到缓存中，便于下次加速查询。

## DDL管理数据库

DDL的特点是对数据库内部的对象进行创建、修改、删除等操作，不涉及对表中内容的操作和更改。

> 创建数据库

```sql
mysql> create database  if not exists kings default charset utf8 collate utf8_general_ci;
Query OK, 1 row affected (0.00 sec)


mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |  # 系统自带库，存储数据库内置信息
| cnblog             |
| kings              |
| lol                |
| mysql              |           # 自带库，存储用户信息，授权等
| performance_schema |          # 存储和性能相关的数据
| student            |
| test               |
+--------------------+
8 rows in set (0.00 sec)


# 查看lol数据库信息，\G是格式化结果
mysql> show create database kings\G
*************************** 1. row ***************************
       Database: kings
Create Database: CREATE DATABASE `kings` /*!40100 DEFAULT CHARACTER SET utf8 */
1 row in set (0.00 sec)

#等同于下面这个结果
mysql> show create database kings;
+----------+----------------------------------------------------------------+
| Database | Create Database                                                |
+----------+----------------------------------------------------------------+
| kings    | CREATE DATABASE `kings` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+----------------------------------------------------------------+
1 row in set (0.00 sec)

```

## 字符集问题

如果想正确的读写mysql的中文数据，需要保证服务端、客户端的字符集一致

服务端是我们的mysql，客户端的形式有很多了，可能是xshell，navicat，程序等等

> 服务端mysql的编码设置与查看

```
# 查看mysql支持哪些字符集
mysql> show character set;
```

> 查看当前mysql的默认字符集，用的是二进制安装包的mysql，默认字符集是latin1

```
mysql> show variables like 'char%';
+--------------------------+--------------------------------------------------------------+
| Variable_name            | Value                                                        |
+--------------------------+--------------------------------------------------------------+
| character_set_client     | utf8                                                         |
| character_set_connection | utf8                                                         |
| character_set_database   | latin1 
```

## 查看数据库

> show语句

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| cnblog             |
| kings              |
| lol                |
| mysql              |
| performance_schema |
| student            |
| test               |
+--------------------+
8 rows in set (0.01 sec)

# 与show命令有关的帮助
mysql> help show

# % 是通配符，匹配以k开头，符合的数据库
mysql> show databases like 'k%';
+---------------+
| Database (k%) |
+---------------+
| kings         |
+---------------+
1 row in set (0.01 sec)

```

## 切换数据库

```sql
# 查看当前在哪一层数据库，类似pwd命令概念
# 默认为NULL空
mysql> select database();
+------------+
| database() |
+------------+
| NULL       |
+------------+
1 row in set (0.00 sec)

mysql> use kings;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

# 这就是已经在库里了
Database changed
mysql> select database();
+------------+
| database() |
+------------+
| kings      |
+------------+
1 row in set (0.00 sec)

```

## 查看库中信息

1.进入数据库查看库中信息

```sql
mysql> select database();
+------------+
| database() |
+------------+
| kings      |
+------------+
1 row in set (0.00 sec)

mysql> show tables;
+-----------------+
| Tables_in_kings |
+-----------------+
| heros           |
| tanks           |
+-----------------+

```

2.库外查询

```sql
mysql> show tables from kings
    -> ;
+-----------------+
| Tables_in_kings |
+-----------------+
| heros           |
| tanks           |
+-----------------+
2 rows in set (0.00 sec)
```

3.基于通配符查询数据表

准备一些数据

```
Tanks               坦克
Warrior             战士
Assassin       刺客
Mage           法师
Archer         射手
Assist         辅助
```

查询sql

```sql
# 进入数据库，创建表
use kings;

# 缩写
create table table_name(column_name column_type);


# 完整
create table if not exists `Assist`(
    id int unsigned  auto_increment,
    name varchar(100) not null,
    skills varchar(255) not null,
    price int not null,
    PRIMARY KEY (id)
)engine=innodb default charset=utf8;

create table if not exists `Archer`(
    id int unsigned auto_increment,
    name varchar(100) not null,
    skills varchar(255) not null,
    price int not null,
    PRIMARY KEY (id)
)engine=innodb default charset=utf8;

# 查询table，以大写A开头的表
mysql> show tables from kings like 'A%';
+----------------------+
| Tables_in_kings (A%) |
+----------------------+
| Archer               |
| Assist               |
+----------------------+
2 rows in set (0.00 sec)

```

## Drop语句

```sql
# 删除数据库
mysql> drop database lol;
Query OK, 0 rows affected (0.00 sec)

# 删除table
# 注意是区分大小写的
mysql> use kings;
Database changed
mysql> show tables;
+-----------------+
| Tables_in_kings |
+-----------------+
| Archer          |
| Assist          |
| heros           |
| tanks           |
+-----------------+
4 rows in set (0.00 sec)

mysql> drop table Archer;
Query OK, 0 rows affected (0.01 sec)

```

## DCL用户管理

> 对于数据库的维护，不能只用一个root去维护,会有其他用户，进行权限控制来加大数据库的安全性

```sql
mysql> select user,host from mysql.user;
+------+-------------+
| user | host        |
+------+-------------+
| root | %           |       # 这个用于远程登录
| yzh  | %           |
| root | 127.0.0.1   |    # 别删
| yzh  | 192.168.6.% |
| root | localhost   |     # 别删
+------+-------------+
5 rows in set (0.00 sec)

```

### 创建mysql用户

语法

```perl
create user 'user'@'host' identified by '用户登录密码';
创建     用户   用户名@允许从哪登录     设定密码  "1234";
```

创建用户

````sql
mysql> create user yuezenghui@'127.0.0.1' identified by '1234';
Query OK, 0 rows affected (0.00 sec)

mysql> select user,host from mysql.user;
+------------+-------------+
| user       | host        |
+------------+-------------+
| root       | %           |
| yzh        | %           |
| yuezenghui | 1127.0.0.1  |
| root       | 127.0.0.1   |
| yzh        | 192.168.6.% |
| root       | localhost   |
+------------+-------------+
6 rows in set (0.00 sec)

````

> 工作里创建用户，一般是只允许一个内网登录，可以设置不同的网段

SQL最常用的通配符就是`%`了，它表示任意字符的匹配，且不计字符的多少

```
#创建cc用户,只允许在192.168.6.0/24网段内访问机器
mysql> create user cc@'192.168.6.%' identified by 'cc888';


#登录测试cc用户
#本地登录报错
[root@sql ~]# mysql -ucc -h127.0.0.1 -P 3306 -p
Enter password: 
ERROR 1045 (28000): Access denied for user 'cc'@'localhost' (using password: YES)

#需要指定服务器地址,如果不在这个网段内，就无法登录了
[root@sql ~]# mysql -ucc -h192.168.6.80  -P 3306 -p
Enter password: 
```

## grant语句

默认创建的用户是没有权限的，只有usage登录

grant也可以用于创建用户

```sql
mysql> grant usage on kings.* to aa@'127.0.0.1' identified by '1234';
Query OK, 0 rows affected (0.01 sec)

# 查看该用户的权限信息
mysql> show grants for yzh@'%';
+----------------------------------------------------------------------------------------------------+
| Grants for yzh@%                                                                                   |
+----------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'yzh'@'%' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
| GRANT SELECT ON `kings`.* TO 'yzh'@'%' WITH GRANT OPTION                                           |
+----------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

```

## 删除用户

语法

```
drop user 'user'@'主机域';  # 单引号
```

```sql
mysql> select user,host from mysql.user;
+------------+-------------+
| user       | host        |
+------------+-------------+
| root       | %           |
| yzh        | %           |
| yuezenghui | 1127.0.0.1  |
| root       | 127.0.0.1   |
| cc         | 192.168.6.% |
| yzh        | 192.168.6.% |
| root       | localhost   |
+------------+-------------+
7 rows in set (0.00 sec)

mysql> drop user cc@'192.168.6.%';
Query OK, 0 rows affected (0.00 sec)


#mysql 新设置用户或更改密码后需用flush privileges刷新MySQL的系统权限相关表
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

```

也可以用如下SQL删除用户，delete

```sql
mysql> delete from mysql.user where user='yuezenghui' and host='%';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

## 授权用户

> 默认create创建的用户是没有权限的，因此得grant给它点权限。

```sql
语法

mysql> help grant

CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY 'password';
GRANT ALL ON db1.* TO 'jeffrey'@'localhost';
GRANT SELECT ON db2.invoice TO 'jeffrey'@'localhost';
GRANT USAGE ON *.* TO 'jeffrey'@'localhost' WITH MAX_QUERIES_PER_HOUR 90;
```

### 实践grant

只允许yzh用户在kings中，执行select命令

```
mysql> grant select on kings.* to qaq@'%' identified by '123456';
Query OK, 0 rows affected (0.00 sec)

```

登录验证qaq用户有没有除select外其他权限

```sql
[root@sql ~]# mysql -h127.0.0.1 -P3306 -uqaq -p123456


mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| kings              |
| test               |
+--------------------+
3 rows in set (0.00 sec)

mysql> use kings;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-----------------+
| Tables_in_kings |
+-----------------+
| Assist          |
| heros           |
| tanks           |
+-----------------+
3 rows in set (0.00 sec)

mysql> create table t1(id int,name varchar(50));
ERROR 1142 (42000): CREATE command denied to user 'qaq'@'localhost' for table 't1'
```

### 给与最大权限

创建huige用户，可以对kings数据库有最大管理权限，只允许从localhost登录(无法远程登录了)

注意，只能用root用户有权限创建用户

```
mysql> grant all privileges on kings.* to huige@'localhost' identified by '1234';
Query OK, 0 rows affected (0.00 sec)

```

测试huige用户

```
[root@sql ~]# mysql -uhuige -p1234 -P3306 -hlocalhost


mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| kings              |
| test               |
+--------------------+
3 rows in set (0.00 sec)

mysql> use kings;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-----------------+
| Tables_in_kings |
+-----------------+
| Assist          |
| heros           |
| tanks           |
+-----------------+
3 rows in set (0.00 sec)

mysql> create table t1(id int,name varchar(50));
Query OK, 0 rows affected (0.01 sec)

mysql> show tables;
+-----------------+
| Tables_in_kings |
+-----------------+
| Assist          |
| heros           |
| t1              |
| tanks           |
+-----------------+
4 rows in set (0.00 sec)

```

### with grant option

> 创建一个和root一样大权力的system用户

这个语句的作用是，可以将自己的权限，传递给其他用户

system的权限是`all privileges`，表示所有权限，并且通过`with grant option`功能，表示system用户也可以给与其他用户最大权限，也就是root的作用。

> 查看当前root权限

```
mysql> show grants for root@localhost;
+----------------------------------------------------------------------------------------------------------------------------------------+
| Grants for root@localhost                                                                                                              |
+----------------------------------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY PASSWORD '*A4B6157319038724E3560894F7F932C8886EBFCF' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION                                                                           |
+----------------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)
```

> 创建个和root一样权限的system用户

```
mysql> grant all privileges on *.* to system@'localhost' identified by '1234' with grant option;
Query OK, 0 rows affected (0.00 sec)


mysql> show grants for system@localhost
    -> ;
+------------------------------------------------------------------------------------------------------------------------------------------+
| Grants for system@localhost                                                                                                              |
+------------------------------------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'system'@'localhost' IDENTIFIED BY PASSWORD '*A4B6157319038724E3560894F7F932C8886EBFCF' WITH GRANT OPTION |
+------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### grant proxy

MySQL的用户权限管理一般都是通过User+Host的形式来区分不同的用户权限，当用户数一多，逐一去修改权限就变得较为繁琐。

有时很多用户需要的权限极为相似，此时，利用MySQL官方提供的Proxy User功能来实现“用户组权限”进行组内用户权限批量管理。

```
mysql> select version();
+------------+
| version()  |
+------------+
| 5.6.34-log |
+------------+
1 row in set (0.00 sec)


# 创建多个用户
# 且创建代理用户 group1
# 这样group1的权限，会映射给user1 user2 省的有大量用户的时候，权限难以管理
mysql> create user 'group1';
Query OK, 0 rows affected (0.00 sec)

mysql> create user 'user1';
Query OK, 0 rows affected (0.00 sec)

mysql> create user 'user2';
Query OK, 0 rows affected (0.00 sec)

mysql> grant proxy on 'group1' to 'user1';
Query OK, 0 rows affected (0.00 sec)

mysql> grant proxy on 'group1' to 'user2';
Query OK, 0 rows affected (0.00 sec)

```

检查所建用户的权限

```
mysql> show grants for group1;
+------------------------------------+
| Grants for group1@%                |
+------------------------------------+
| GRANT USAGE ON *.* TO 'group1'@'%' |
+------------------------------------+
1 row in set (0.00 sec)

mysql> show grants for user1;
+--------------------------------------------+
| Grants for user1@%                         |
+--------------------------------------------+
| GRANT USAGE ON *.* TO 'user1'@'%'          |
| GRANT PROXY ON 'group1'@'%' TO 'user1'@'%' |
+--------------------------------------------+
2 rows in set (0.00 sec)

mysql> show grants for user2;
+--------------------------------------------+
| Grants for user2@%                         |
+--------------------------------------------+
| GRANT USAGE ON *.* TO 'user2'@'%'          |
| GRANT PROXY ON 'group1'@'%' TO 'user2'@'%' |
+--------------------------------------------+
2 rows in set (0.00 sec)

```

> 使用新用户登录，但是没有赋予任何权限，看不到其他库，只有一个information_schema

```
[root@sql ~]# mysql -uuser1 -P3306 -h127.0.01

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| test               |
+--------------------+
2 rows in set (0.00 sec)
```

> 此时给我们的组用户group1授权，再看权限

```
mysql> grant select on *.* to group1;
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> show grants for group1;
+-------------------------------------+
| Grants for group1@%                 |
+-------------------------------------+
| GRANT SELECT ON *.* TO 'group1'@'%' |
+-------------------------------------+
1 row in set (0.00 sec)

```

此时此刻，group1组用户多了select权限，user1、user2是看不出变化的。

> 再用user1、组内用户登录，看看权限

```
[root@mysql-server56 ~]# mysql -uuser1 -h127.0.0.1 -P3307


mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
+--------------------+
1 row in set (0.00 sec)
```

此时发现，并没有发生权限变化。

这是因为这个功能只在mysql5.7之后可用，我们这里的5.6无法使用，虽然说mysql有显示grant proxy

> 解释

基于mysql_native_password的认证插件自带了代理用户的功能。代理用户相当于“代理”其他用户的权限，这样很方便的把一个账号的权限授予其他账号，而不需要每个账号都需要执行授权操作。开启代理用户的功能需要开启参数：`check_proxy_users` 和 `mysql_native_password_proxy_users`

```
# mysql 5.7的显示
mysql> show variables like "%proxy%";
+-----------------------------------+-------+
| Variable_name                     | Value |
+-----------------------------------+-------+
| check_proxy_users                 | OFF   |
| mysql_native_password_proxy_users | OFF   |
| proxy_user                        |       |
| sha256_password_proxy_users       | OFF   |
+-----------------------------------+-------+
4 rows in set (0.00 sec)
```

## 创建用户且设置密码

注意localhost是被dns解析后出的127.0.0.1，也只能本地访问

```perl
mysql> create user 'abc'@'localhost' identified by '1234';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

希望局域网内访问就是

```perl
mysql> create user 'cc1'@'10.211.55.%' identified by 'cc1888';
Query OK, 0 rows affected (0.00 sec)
```

若是希望外网用户也可以访问，例如服务器上的mysql

```perl
create user 'cc2'@'%' identified by 'cc1888';
```

此时检查用户

```perl
mysql> select user,host from mysql.user;
+--------+-------------+
| user   | host        |
+--------+-------------+
| cc2    | %           |
| group1 | %           |
| user1  | %           |
| user2  | %           |
| cc1    | 10.211.55.% |
| root   | 127.0.0.1   |
| yuyu   | 127.0.0.1   |
| cc     | localhost   |
| pyyu1  | localhost   |
| root   | localhost   |
| system | localhost   |
+--------+-------------+
11 rows in set (0.00 sec)
```

## 权限列表

```perl
mysql> show grants for 'cc2'@'%';
+----------------------------------------------------------------------------------------------------+
| Grants for cc2@%                                                                                   |
+----------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'cc2'@'%' IDENTIFIED BY PASSWORD '*16932EF14224F810F27C01DE21AB5522C07BD695' |
+----------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

# 授予最高权限
mysql> grant all privileges on *.* to 'cc2'@'%';
Query OK, 0 rows affected (0.00 sec)


mysql> show grants for 'cc2'@'%';
+-------------------------------------------------------------------------------------------------------------+
| Grants for cc2@%                                                                                            |
+-------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'cc2'@'%' IDENTIFIED BY PASSWORD '*16932EF14224F810F27C01DE21AB5522C07BD695' |
+-------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

> cc2用户有所有的权限，我们单独去掉一个select权限，再来看

## 移除权限

revoke语句

```
mysql> revoke select on *.* from huige@localhost;
Query OK, 0 rows affected (0.00 sec)


mysql> show grants for huige@localhost;
+--------------------------------------------------------------------------------------------------------------+
| Grants for huige@localhost                                                                                   |
+--------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'huige'@'localhost' IDENTIFIED BY PASSWORD '*A4B6157319038724E3560894F7F932C8886EBFCF' |
| GRANT ALL PRIVILEGES ON `kings`.* TO 'huige'@'localhost'                                                     |
+--------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

```

`all privileges`

## all privileges命令表

![image-20220222195410854](mysql_SQL.assets/image-20220222195410854.png)