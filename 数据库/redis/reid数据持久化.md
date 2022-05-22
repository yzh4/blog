## redis数据持久化

## rdb持久化

`RDB` 持久化(内存数据保存到硬盘)

可以在指定的时间间隔内生成数据集的时间点快照(point-in-time snapshot)

`优点`：速度块，适用于做备份，主从复制也是基于`RDB`持久化功能实现的

`缺点`：会有数据流失

## rdb持久化核心配置参数

```
[root@docker01 ~]# cat /data/6379/redis.conf 
dir /data/6379
dbfilename dump.rdb
save 900 1
save 300 10
save 60 10000
```

```
配置分别表示:
900秒(15分钟)内有一个更改
300秒(5分钟)内有10个更改
60秒内有10000个更改
```

## aof持久化

记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集

`aof`文件中的命令全部以`redis`协议的格式来保存，新命令会被追加到文件的末尾

`优点`：可以最大程度保证数据不丢

`缺点`：日志记录量级比较大

## AOF持久化配置参数

```
[root@docker01 ~]# vim /data/6379/redis.conf
appendonly yes
appendfsync everysec
appendfsync always
```

```
配置表示:
是否开启aof日志功能
开启写入，每秒写一次
每一个命令都立即同步到aof
```

## 面试

redis持久化方式有哪些，有什么区别？

`rdb`:基于快照持久化，速度更快，一般用作备份，主从复制也是依赖于rdb持久化功能

`aof`：以追加的方式记录redis操作日志的文件。可以最大程度保证redis数据安全，类似于mysql的binlog