### 服务器安装redis-cli

> 模糊查询获取具体key

```
redis-cli -h r-pc6fcef2da4c2f94.redis.rds.ops.sgsc.sgcc.com.cn -p 16379 -a New567oa_202112  keys key值

例：
redis-cli -h r-pc6fcef2da4c2f94.redis.rds.ops.sgsc.sgcc.com.cn -p 16379 -a New567oa_202112  keys osg-os0001d:key_seat_address_*
```

> 获取单个key值

```
redis-cli -h r-pc6fcef2da4c2f94.redis.rds.ops.sgsc.sgcc.com.cn -p 16379 -a New567oa_202112  get key

例：
redis-cli -h r-pc6fcef2da4c2f94.redis.rds.ops.sgsc.sgcc.com.cn -p 16379 -a New567oa_202112  get osg-os0001d:key_seat_address_02004976
```

> 批量获取key值

```
for i in $(cat 1.txt);do redis-cli -h r-pc6fcef2da4c2f94.redis.rds.ops.sgsc.sgcc.com.cn -p 16379 -a New567oa_202112 get $i>>2.txt ;done
```

> 删除单个key值（慎用）

```
redis-cli -h r-pc6fcef2da4c2f94.redis.rds.ops.sgsc.sgcc.com.cn -p 16379 -a New567oa_202112  del key

例：
redis-cli -h r-pc6fcef2da4c2f94.redis.rds.ops.sgsc.sgcc.com.cn -p 16379 -a New567oa_202112  del osg-os0001d:key_seat_address_02004976
```

> 批量删除key值 （慎用）

```
for i in $(cat 1.txt);do redis-cli -h r-pc6fcef2da4c2f94.redis.rds.ops.sgsc.sgcc.com.cn -p 16379 -a New567oa_202112 del $i;done
```

