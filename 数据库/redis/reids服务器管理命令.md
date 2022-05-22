## reids服务器管理命令

## 版本信息

```bash
127.0.0.1:6379> info
#info信息关注的重点信息
#redis内存信息
# Memory
used_memory:824328
used_memory_human:805.01K
used_memory_rss:2469888
used_memory_rss_human:2.36M
used_memory_peak:844592
used_memory_peak_human:824.80K
total_system_memory:3953971200
total_system_memory_human:3.68G

#复制
## Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

## 会话占用资源情况

```bash
127.0.0.1:6379> client list
id=21 addr=127.0.0.1:58250 fd=7 name= age=88 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
```

## 杀掉进程

```bash
#一般用于杀掉远程链接
127.0.0.1:6379> client kill 127.0.0.1:58250
OK
```

## 查看当前配置情况

```bash
127.0.0.1:6379> config get bind
1) "bind"
2) "192.168.6.80 127.0.0.1"

#查询所有
127.0.0.1:6379> config get *
  1) "dbfilename"
  2) "dump.rdb"
  3) "requirepass"
  4) "root"

```

## 动态修改

```bash
127.0.0.1:6379> config set requirepass 1234
OK
```

## 查看key的个数

```bash
127.0.0.1:6379> dbsize
(integer) 9
```

## 清空键值对缓存数据

```bash
#(清空0-15所有库)有风险，慎用
127.0.0.1:6379> flushall
OK
127.0.0.1:6379> keys *
(empty list or set)

```

## 清空数据

```
#清空当前库
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> keys *
(empty list or set)

```

## 切换库

```
#默认有16个库从0-15,通常集群环境统一0号库
127.0.0.1:6379> select 0
OK
127.0.0.1:6379[0]>
```

## 监控实时指令

```bash
#通常用作redis审计日志
#窗口1
127.0.0.1:6379> monitor
OK
1644557973.699574 [0 127.0.0.1:58254] "set" "a" "10"
#窗口2
127.0.0.1:6379> set a 10
OK
```

## 关闭服务器

```
127.0.0.1:6379> shutdown
not connected>
#
[root@sql ~]# redis-cli -a root shutdown
```

## 数据保存

```
save
```

- slaveof host port 主从配置
- slaveof no one
- sync 主从同步
- role返回主从角色 

## 常用全局key操作

```bash
#查看所有键
127.0.0.1:6379> keys *
1) "a"
#删除键
127.0.0.1:6379> del a
(integer) 1
127.0.0.1:6379> exists a
(integer) 0
127.0.0.1:6379> set a 10
OK
#判断键是否存在
127.0.0.1:6379> exists a
(integer) 1
#判断键数据类型
127.0.0.1:6379> type a
string
#以秒设定键生存时间
127.0.0.1:6379> expire a 120
(integer) 1
#返回键所剩生存时间
127.0.0.1:6379> ttl a
(integer) 117
#取消键设定的生存时间
127.0.0.1:6379> persist a
(integer) 1

```

