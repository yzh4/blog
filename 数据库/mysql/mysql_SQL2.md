## grant授权实践

> 创建用于测试环境使用的账号，不能给与all privileges权限，只能给与单个，用于DML场景的语句，比如select,insert,update,delete适合web

因为一般开发工程师、测试工程师，会需要测试读写数据库，例如对网站登录功能，注册功能测试，那就得写入数据，读取数据。

通过各种语言，开发数据库数据读写操作。

```
# 清空之前创建的测试用户
mysql> select user,host from mysql.user;
+------+-----------+
| user | host      |
+------+-----------+
| root | %         |
| root | 127.0.0.1 |
| root | localhost |
+------+-----------+
3 rows in set (0.00 sec)

# 创建用户，授权
mysql> grant select,insert,update,delete on kings.* to yzh@'192.168.6.%' identified by '11234';
Query OK, 0 rows affected (0.00 sec)

#刷新列表
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

```

> 授权几大特性

- 权限不要轻易用all，安全性太低，应该使用select、insert、update、delete等具体权限
- 主机范围尽量不用%，应该设置具体的网段
- 库的选择也不用*，应该指定具体的库、表

> python读取数据库，注意，只有查询、插入、更新、删除四个权限

```
# 创建英雄表
mysql> create table heros(id int,name varchar(50));
Query OK, 0 rows affected (0.01 sec)

mysql> desc heros;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | YES  |     | NULL    |       |
| name  | varchar(50) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

# 默认是没数据的
mysql> select * from kings.heros;
Empty set (0.00 sec)
```

> 安装pymysql模块

```perl
[root@mysql-server56 ~]# pip3 install pymysql
WARNING: Running pip install with root privileges is generally not a good idea. Try `pip3 install --user` instead.
Collecting pymysql
  Downloading https://files.pythonhosted.org/packages/4f/52/a115fe175028b058df353c5a3d5290b71514a83f67078a6482cff24d6137/PyMySQL-1.0.2-py3-none-any.whl (43kB)
    100% |████████████████████████████████| 51kB 238kB/s
Installing collected packages: pymysql
Successfully installed pymysql-1.0.2
```

> 代码

```python
import pymysql  
# 连接
# 使用该普通用户，连接数据库写入数据
conn = pymysql.Connect(host="10.211.55.12", port=3306, user="1234",
                       passwd="1234",db="kings")
# 创建游标
cursor = conn.cursor()

# 插入数据
sql="insert into heros(id,name) values('1','孙悟空')"

#执行sql，并返回受影响行数
rows = cursor.execute(sql)
print(f"插入了{rows}行数据")

# 执行查询SQL
cursor.execute("select * from heros;")
print(f"查出的结果是：{cursor.fetchall()}")

# 提交事务
conn.commit()
# 关闭游标
cursor.close()
# 关闭连接
conn.close()
```

执行结果

```
[root@sql 3306]# python3 python_mysql.py 
插入了1行数据
查出的结果是：((1, '孙悟空'),)

```

再来mysql中查询数据

```perl
mysql> select * from kings.heros;
+------+-----------+
| id   | name      |
+------+-----------+
|    1 | 孙悟空    |
+------+-----------+
1 row in set (0.00 sec)
```

数据已插入表中

## 博客产品授权

比如有很多开源软件的博客产品，在安装期间会需要初始化安装数据库，也可以有删除数据库。

因此需要增加create,drop权限，如

```sql
# 例如创建kings数据库，允许用户在库中建表
mysql> grant select,insert,update,delete,create,drop on kings.* to kings_user@'192.168..6.%' identified by '1234';
Query OK, 0 rows affected (0.00 sec)

```

## 只读权限

在很多场景下，我们只会给一些账号，只读的权限，这很重要，例如后面的读写分离，例如给开发人员的测试账号，尽量不给select以外的权限

```sql
mysql> grant select on student.* to stu@'192.168.6.%' identified by '1234';
Query OK, 0 rows affected (0.00 sec)

```

## 创建数据表

创建数据表，语法

```perl
create table <表名> (
    字典名 类型,
    字段名2 类型,
)
```

### 创建英雄表

```
Tanks               坦克
Warrior             战士
Assassin       刺客
Mage           法师
Archer         射手
Assist         辅助
```

> 创建数据表，坦克表

```sql
use kings;

create table if not exists `tanks`(
	id int unsigned auto_increment,
	name varchar(100) not null,
	skills varchar(255) not null,
	price int not null,
	PRIMARY KEY(id)
)engine=innodb default charset=utf8;



# `create table`是固定关键字，创建table，`tanks`是表名，`if not exists`是更专业的写法，判断该table是否存在，不存在则创建。
create table if not exists `tanks`(
  # 英雄序号，数字类型，无符号， 既为非负数，自动递增+1
    id int unsigned  auto_increment,
    # 英雄名字字段，100长度的变长字符类型varchar，还有一种char(定长)
    name varchar(100) not null,
    # 英雄介绍 ，不得为空
    introduction varchar(255) not null,
    # 价格 
    price int not null,
    # 设置一个主键
    PRIMARY KEY (id)
    # 设置mysql数据库的引擎，默认字符集是utf8
)engine=innodb default charset=utf8;
```

### 查看table表结构

```sql
mysql> desc tanks;
+--------+--------------+------+-----+---------+-------+
| Field  | Type         | Null | Key | Default | Extra |
+--------+--------------+------+-----+---------+-------+
| id     | int(11)      | NO   | PRI | 0       |       |
| name   | varchar(100) | NO   |     | NULL    |       |
| skills | varchar(255) | NO   |     | NULL    |       |
| price  | int(11)      | NO   |     | NULL    |       |
+--------+--------------+------+-----+---------+-------+
4 rows in set (0.00 sec)

```

> 无需切换数据库内的方法

```sql
mysql> show columns from kings.tanks;
+--------+--------------+------+-----+---------+-------+
| Field  | Type         | Null | Key | Default | Extra |
+--------+--------------+------+-----+---------+-------+
| id     | int(11)      | NO   | PRI | 0       |       |
| name   | varchar(100) | NO   |     | NULL    |       |
| skills | varchar(255) | NO   |     | NULL    |       |
| price  | int(11)      | NO   |     | NULL    |       |
+--------+--------------+------+-----+---------+-------+
4 rows in set (0.00 sec)

```

### 查看建表信息

```sql
mysql> show create table tanks;
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                        |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tanks | CREATE TABLE `tanks` (
  `id` int(11) NOT NULL DEFAULT '0',
  `name` varchar(100) NOT NULL,
  `skills` varchar(255) NOT NULL,
  `price` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```

### 一个完全的table语句

```sql
use sns;
set names gbk;
CREATE TABLE `subject_comment_manager` (
  `subject_comment_manager_id` bigint(12) NOT NULL auto_increment COMMENT '主键',
  `subject_type` tinyint(2) NOT NULL COMMENT '素材类型',
  `subject_primary_key` varchar(255) NOT NULL COMMENT '素材的主键',
  `subject_title` varchar(255) NOT NULL COMMENT '素材的名称',
  `edit_user_nick` varchar(64) default NULL COMMENT '修改人',
  `edit_user_time` timestamp NULL default NULL COMMENT '修改时间',
  `edit_comment` varchar(255) default NULL COMMENT '修改的理由',
  `state` tinyint(1) NOT NULL default '1' COMMENT '0代表关闭，1代表正常',
  PRIMARY KEY  (`subject_comment_manager_id`),
  KEY `IDX_PRIMARYKEY` (`subject_primary_key`(32)), #<==括号内的32表示对前
                                                  32个字符做前缀索引。
  KEY `IDX_SUBJECT_TITLE` (`subject_title`(32))
  KEY `index_nick_type` (`edit_user_nick`(32),`subject_type`)
  #<==联合索引，此行为是新加的，实际表语句内没有此行。
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

## 修改数据表

> rename table

```sql
mysql> show tables;
+-----------------+
| Tables_in_kings |
+-----------------+
| Tanks           |
| heros           |
| t1              |
+-----------------+
3 rows in set (0.00 sec)

mysql> rename table Tanks to tanks;
Query OK, 0 rows affected (0.01 sec)

mysql> show tables;
+-----------------+
| Tables_in_kings |
+-----------------+
| heros           |
| t1              |
| tanks           |
+-----------------+
3 rows in set (0.00 sec)
```

> Alter table

```sql
mysql> alter table heros rename to Heros;
Query OK, 0 rows affected (0.00 sec)

mysql> show tables;
+-----------------+
| Tables_in_kings |
+-----------------+
| Heros           |
| t1              |
| tanks           |
+-----------------+
3 rows in set (0.00 sec)
```

### 修改表字段记录

> 表中字段
>
> id、名字、技能、价格

```perl
mysql> desc tanks;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name   | varchar(100)     | NO   |     | NULL    |                |
| skills | varchar(255)     | NO   |     | NULL    |                |
| price  | int(11)          | NO   |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.01 sec)
```

### 添加字段

> Alter table 表名 add 字段 类型 其他;

> 添加一个介绍字段introduction varchar(255) not null,

```
mysql> alter table tanks add introduction varchar(255) not null;


mysql> desc tanks;
+--------------+--------------+------+-----+---------+-------+
| Field        | Type         | Null | Key | Default | Extra |
+--------------+--------------+------+-----+---------+-------+
| id           | int(11)      | NO   | PRI | 0       |       |
| name         | varchar(100) | NO   |     | NULL    |       |
| skills       | varchar(255) | NO   |     | NULL    |       |
| price        | int(11)      | NO   |     | NULL    |       |
| introduction | varchar(255) | NO   |     | NULL    |       |
+--------------+--------------+------+-----+---------+-------+
5 rows in set (0.00 sec)

```

> 添加字段到指定位置，加到skills后面
>
> 再添加一个"召唤师技能"字段
>
> ENUM字段类型
>
> summoner_skills ENUM('flush','ghost') not null default 'flush' # flush 闪现 ghost 幽灵疾步

```sql
mysql> alter table tanks add summoner_skills enum('flush','ghost') not null default 'flush' after skills;


mysql> desc tanks;
+-----------------+-----------------------+------+-----+---------+-------+
| Field           | Type                  | Null | Key | Default | Extra |
+-----------------+-----------------------+------+-----+---------+-------+
| id              | int(11)               | NO   | PRI | 0       |       |
| name            | varchar(100)          | NO   |     | NULL    |       |
| skills          | varchar(255)          | NO   |     | NULL    |       |
| summoner_skills | enum('flush','ghost') | NO   |     | flush   |       |
| price           | int(11)               | NO   |     | NULL    |       |
| introduction    | varchar(255)          | NO   |     | NULL    |       |
+-----------------+-----------------------+------+-----+---------+-------+
6 rows in set (0.00 sec)

```

> 再添加一个性别字段，在name后面

```sql
mysql> alter table tanks add gender enum('male','female') not null after name;


mysql> desc tanks;
+-----------------+-----------------------+------+-----+---------+-------+
| Field           | Type                  | Null | Key | Default | Extra |
+-----------------+-----------------------+------+-----+---------+-------+
| id              | int(11)               | NO   | PRI | 0       |       |
| name            | varchar(100)          | NO   |     | NULL    |       |
| gender          | enum('male','female') | NO   |     | NULL    |       |
| skills          | varchar(255)          | NO   |     | NULL    |       |
| summoner_skills | enum('flush','ghost') | NO   |     | flush   |       |
| price           | int(11)               | NO   |     | NULL    |       |
| introduction    | varchar(255)          | NO   |     | NULL    |       |
+-----------------+-----------------------+------+-----+---------+-------+
7 rows in set (0.00 sec)

```

> 一次添加2个字段
>
> camp varchar(50) 阵营
>
> pic varchar(255) 头像，存储图片url

```sql
mysql> alter table tanks add camp varchar(50),add pic varchar(255);


mysql> desc tanks;
+-----------------+-----------------------+------+-----+---------+-------+
| Field           | Type                  | Null | Key | Default | Extra |
+-----------------+-----------------------+------+-----+---------+-------+
| id              | int(11)               | NO   | PRI | 0       |       |
| name            | varchar(100)          | NO   |     | NULL    |       |
| gender          | enum('male','female') | NO   |     | NULL    |       |
| skills          | varchar(255)          | NO   |     | NULL    |       |
| summoner_skills | enum('flush','ghost') | NO   |     | flush   |       |
| price           | int(11)               | NO   |     | NULL    |       |
| introduction    | varchar(255)          | NO   |     | NULL    |       |
| camp            | varchar(50)           | YES  |     | NULL    |       |
| pic             | varchar(255)          | YES  |     | NULL    |       |
+-----------------+-----------------------+------+-----+---------+-------+
9 rows in set (0.00 sec)
```

### 开头添加字段

```
alter table <表名> ADD <新字段名> <数据类型> [约束条件] first;
```

在开头添加id

```
mysql> alter table tanks add id int(11) primary key auto_increment first;
Query OK, 0 rows affected (0.07 sec)
Records: 0  Duplicates: 0  Warnings: 0

```

### 在末尾添加字段

```
alter table <表名> ADD <新字段名><数据类型>[约束条件];
```

### 在中间位置添加字段

```
alter table <表名> add <新字段名> <数据类型> [约束条件] agter <已经存在的字段名>;
```



### 删除字段

> 删除性别字段

```sqlite
mysql> alter table tanks drop gender;
```

### 修改字段

> 修改名字以及数据类型

```sql
mysql> alter table tanks change pic pic_url char(255);


mysql> desc tanks;
+-----------------+-----------------------+------+-----+---------+-------+
| Field           | Type                  | Null | Key | Default | Extra |
+-----------------+-----------------------+------+-----+---------+-------+
| id              | int(11)               | NO   | PRI | 0       |       |
| name            | varchar(100)          | NO   |     | NULL    |       |
| skills          | varchar(255)          | NO   |     | NULL    |       |
| summoner_skills | enum('flush','ghost') | NO   |     | flush   |       |
| price           | int(11)               | NO   |     | NULL    |       |
| introduction    | varchar(255)          | NO   |     | NULL    |       |
| camp            | varchar(50)           | YES  |     | NULL    |       |
| pic_url         | char(255)             | YES  |     | NULL    |       |
+-----------------+-----------------------+------+-----+---------+-------+

```

## 索引

### 创建索引

> 索引功能，就像一本书的目录，知道目录，可以非常快速找到我们想要的内容在哪
>
> 通过给字段添加索引，能够加快该列字段的查询速度

```sql
#查看表默认索引
mysql> show index from tanks\G
*************************** 1. row ***************************
        Table: tanks
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: id
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE
      Comment: 
Index_comment: 
1 row in set (0.00 sec)

```

> alter创建索引

```sql
#给当前的表字段，添加索引
mysql> alter table tanks add index index_name(name);

#查看索引
mysql> show index from tanks\G
```

### 删除索引

```sql
mysql> alter table tanks drop index index_name;

mysql> show index from tanks\G
```

## 插入数据

语法

```perl
insert into table_name  field1,field2,field3 values(值1,值2,值3);
```

### 插入英雄数据

> 1.根据指定的列名，每一列都插入值，插入完全的值，列和值要对应

```sql
#查看现有表，插入对应的英雄
mysql> desc tanks;
+-----------------+-----------------------+------+-----+---------+-------+
| Field           | Type                  | Null | Key | Default | Extra |
+-----------------+-----------------------+------+-----+---------+-------+
| id              | int(11)               | NO   | PRI | 0       |       |
| name            | varchar(100)          | NO   |     | NULL    |       |
| skills          | varchar(255)          | NO   |     | NULL    |       |
| summoner_skills | enum('flush','ghost') | NO   |     | flush   |       |
| price           | int(11)               | NO   |     | NULL    |       |
| introduction    | varchar(255)          | NO   |     | NULL    |       |
| camp            | varchar(50)           | YES  |     | NULL    |       |
| pic_url         | char(255)             | YES  |     | NULL    |       |
+-----------------+-----------------------+------+-----+---------+-------+

#表无数据
mysql> select * from tanks;
Empty set (0.00 sec)


# 插入数据
# 字段和值要对应上
mysql> insert into tanks(id,name,skills,summoner_skills,price,introduction,camp,pic_url) values(1,'亚瑟','圣剑裁决','flush','5888','能抗能打，技能沉默','近战','https://img.181833.com/uploads/allimg/190924/266-1Z9241Q224.jpg');



mysql> select * from tanks;
+----+--------+--------------+-----------------+-------+-----------------------------+--------+----------------------------------------------------------------+
| id | name   | skills       | summoner_skills | price | introduction                | camp   | pic_url                                                        |
+----+--------+--------------+-----------------+-------+-----------------------------+--------+----------------------------------------------------------------+
|  1 | 亚瑟   | 圣剑裁决     | flush           |  5888 | 能抗能打，技能沉默          | 近战   | https://img.18183.com/uploads/allimg/190924/266-1Z9241Q224.jpg |
+----+--------+--------------+-----------------+-------+-----------------------------+--------+----------------------------------------------------------------+
1 row in set (0.01 sec)


#格式化查询
mysql> select * from tanks\G
*************************** 1. row ***************************
             id: 1
           name: 亚瑟
         skills: 圣剑裁决
summoner_skills: flush
          price: 5888
   introduction: 能抗能打，技能沉默
           camp: 近战
        pic_url: https://img.18183.com/uploads/allimg/190924/266-1Z9241Q224.jpg
```

> 2.插入部分数据

- id自增可不填

- not null 不可以插入null值，可以插入‘  ’值
- 不指定列按照规则

```sql
#注意哪些字段是not null
mysql> desc tanks;

#插入部分数据 ，插入名字
mysql> insert into tanks(name)values("凯");


# 查看值
mysql> select * from tanks\G
*************************** 1. row ***************************
             id: 0
           name: 凯
         skills: 
summoner_skills: flush
          price: 0
   introduction: 
           camp: NULL
        pic_url: NULL

```

> 3.批量插入数据

```sql
mysql> insert into tanks(id,name,skills) values(2,'东皇太一','堕神契约'),(3,'吕布','魔神降临');
Query OK, 2 rows affected, 2 warnings (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 2

```

## 删除数据

### 删除库、表

> 删除数据库

```perl
mysql> drop database k8s;
Query OK, 0 rows affected (0.00 sec)
```

> 删除数据表

```perl
# 查看当前在那个库
mysql> select database();
+------------+
| database() |
+------------+
| kings      |
+------------+
1 row in set (0.00 sec)

# 删除数据表
mysql> show tables;
+-----------------+
| Tables_in_kings |
+-----------------+
| Heros           |
| t1              |
| tanks           |
+-----------------+
3 rows in set (0.00 sec)

mysql> drop table t1;
Query OK, 0 rows affected (0.01 sec)
```



### 删除表数据delete

删除表数据

delete from语句

delete from语句可以使用where对要删除的记录进行选择，delete语句更灵活。

delete是一行一行的删除数据，对于大容量数据表，delete效率较低

delete语句对于带有自增id的列，数据删除后，会保留id的位置

```sql
# 没有自增的id列
mysql> desc heros;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | YES  |     | NULL    |       |
| name  | varchar(50) | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

# 查看表数据
mysql> select * from Heros;
+------+-----------+
| id   | name      |
+------+-----------+
|    1 | 孙悟空    |
+------+-----------+


#删除数据表中所有数据
mysql> delete from Heros;


mysql> select * from Heros;
Empty set (0.00 sec)

# 对于有自增id的列
# 给Heros表添加自增id
mysql> alter table Heros modify  id int(11)  primary key auto_increment;
```

### 修改表字段modify

> 给字段添加自增属性

```
alter table heros modify id int(11) primary key auto_increment;
```

### 自增id列保留数值

```
alter table heros modify id int(11) primary key auto_increment;

mysql> insert into heros(name)values('锐雯');


mysql> select * from heros;
+----+--------+
| id | name   |
+----+--------+
|  3 | 锐雯   |
+----+--------+
```

### truncate删除表数据

> truncate语句，清空表数据
>
> truncate清空表数据，是重新建立一个新表，能够删除id自增列的数值，重0开始

```
mysql> truncate table heros;

mysql> select * from heros;

```

## 查询数据

> 语法

> Select 字段1,字段2,字段3 from table_name where 表达式;

### 1.进入数据库查询表

```
mysql> use kings;
Database changed
mysql> select * from tanks;
```

### 2.不进入库查询表

> 通过库.表查询

```
mysql> select user,host from mysql.user;
```

### 条件表达式查询

### limit指定条数

```
# 限定查看2条数据
mysql> select * from tanks limit 2;

#指定开始位置，找出2条数据
#limit 起点,条数;
mysql> select * from tanks limit 2,2\G
```

### where指定查询条件

> where用于如网站的筛选功能

```
#找出凯的信息
mysql> select * from tanks where name='凯';


# 同时查询多个条件
# 找出id大于1，价格大于6000的英雄
mysql> select * from  tanks where id>1 and price>6000;

# 找出指定id的记录，指定记录的显示
mysql> select id,name,price from tanks where id=2;
+----+--------+-------+
| id | name   | price |
+----+--------+-------+
|  2 | 亚瑟   |  5888 |
+----+--------+-------+

```

### order排序功能

```
# desc 降序
# 找出id大于1，且价格大于5000的英雄，且降序排序，结果只输出id，name,price

mysql> select id,name,price from kings.tanks where id>1 and price>5000 order by price desc;
```

## 修改表数据

语法：

> update <表名> set [字段]=' ' where <字段>=' '

```
#修改吕布的价格为18888
mysql> update kings.tanks set price='18888' where name='吕布';


mysql> update kings.tanks set skills='大招' where name='露娜';

```

## 练习操作

1）登录MySQL数据库。

```
[root@sql ~]# mysql -uroot -S /data/3306/mysql.sock 
[root@sql ~]# mysql -uroot -P3306 -p1234 -h127.0.0.1
```

2）查看当前登录的用户。

```
select user();
```

3）创建数据库test01，并查看已建库完整语句。

```
create database test01;

show create database test01\G
```

4）创建用户usertest，使之可以管理数据库test01。

```
create user usertest@'localhost' identified by '1234';
```

5）查看创建的用户chaoge拥有哪些权限。

6）查看当前数据库里有哪些用户。

7）进入chaoge_linux数据库。

8）查看当前所在的数据库。

9）创建一张表test，字段id和name varchar(16)。

10）查看建表结构及表结构的SQL语句。

11）插入一条数据“1，yuchao”。

12）再批量插入2行数据“2，pyyu” ，“3，yuchao_linux”。

13）查询名字包含为yu的记录。

14）把数据id等于1的名字yuchao更改为yuchao888。

15）在字段name前插入age字段，类型tinyint(2)。

16）备份chaoge_linux数据库。

17）删除test表中的所有数据，并查看。

18）删除表test和chaoge_linux数据库并查看。

19）不退出数据库恢复以上删除的数据。

20）将id列设置为主键，在Name字段上创建普通索引。

21）在字段name后插入手机号字段（phone），类型为char(11)。

22）所有字段上插入2条记录（自行设定数据）。

23）删除Name列的索引。

24）查询手机号以152开头的，名字为yuchao的记录（提前插入）。

26）删除chaoge用户。

27）删除chaoge_linux数据库。

28）停止数据库。

29）如何重置mysql密码

30）简述SQL执行原理流程
