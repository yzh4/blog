## redis配置文件

```shell
[root@docker01 ~]# mkdir /data/6379

[root@docker01 ~]# vim /data/6379/redis.conf 
#是否打开redis后台运行
daemonize yes
#端口号
port 6379
#日志存放位置
logfile /data/6379/redis.log
#设定redis数据存储位置
dir /data/6379
#redis持久化的数据文件
dbfilename dump.rdb

```

## 关闭redis

```
[root@docker01 ~]# redis-cli
127.0.0.1:6379> shutdown
53486:M 09 Feb 00:20:40.341 # User requested shutdown...
53486:M 09 Feb 00:20:40.341 * Saving the final RDB snapshot before exiting.
53486:M 09 Feb 00:20:40.342 * DB saved on disk
53486:M 09 Feb 00:20:40.342 # Redis is now ready to exit, bye bye...
not connected> exit
[1]+  Done                    redis-server

#命令行关闭redis
[root@docker01 ~]# redis-cli shutdown
```

## 使用新的配置文件启动

```
[root@docker01 ~]# redis-server /data/6379/redis.conf 
[root@docker01 ~]# redis-cli
127.0.0.1:6379> 
```

