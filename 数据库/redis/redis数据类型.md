## redis数据类型

|                      |    key(键)     |                value(值)                |
| :------------------: | :------------: | :-------------------------------------: |
|                      | 自主定义的名字 |         多种数据类型的存储模式          |
|    string(字符串)    |      name      |               "zhangsan"                |
|      hash(字典)      |      stu       |         {id:101,name:zhangsan}          |
|      list(列表)      |     wechat     |          (v1,v2,v3)<br />0 1 2          |
|      set(集合)       |      set1      |          [m1,m2,m3]<br />0 1 2          |
| sorted set(有序集合) |     zset1      | [socre m1,score m2,score m3]<br />0 1 2 |

## string

应用场景：

- 基本键值对存储
- 计数器(互联网当中，点击量，访问量，关注量，游戏应用)

```bash
#定义一个键值对
127.0.0.1:6379> set name zhangsan
OK
127.0.0.1:6379> get name
"zhangsan"

#定义多个键值对
127.0.0.1:6379> mset id 101 name zhangsan age 20 gender maile
OK

#获取多个值
127.0.0.1:6379> mget age id name gender
1) "20"
2) "101"
3) "zhangsan"
4) "maile"

#查看定义键
127.0.0.1:6379> keys *
1) "age"
2) "id"
3) "name"
4) "gender"

#统计计量增加
127.0.0.1:6379> incr fensi
(integer) 1
127.0.0.1:6379> incr fensi
(integer) 2

#减少
127.0.0.1:6379> decr fensi
(integer) 1
127.0.0.1:6379> decr fensi
(integer) 0

#获取统计
127.0.0.1:6379> get fensi
"2"

#增加多条数据
127.0.0.1:6379> incrby fensi 20
(integer) 20
#减少
127.0.0.1:6379> decrby fensi 10
(integer) 10

```

## hash

应用场景：

- 最接近于mysql表结构的数据类型,存储部分变更的数据，如用户信息等
- 最多应用与数据库缓存(提前将mysql数据库数据灌入到redis中)

```bash
127.0.0.1:6379> hset zhangsan name zs
(integer) 1
127.0.0.1:6379> hmset student id 101 name zs age 20 gender male
OK
127.0.0.1:6379> hmset stu id 102 name lisi age 21 gender male
OK
127.0.0.1:6379> hmget stu
(error) ERR wrong number of arguments for 'hmget' command
127.0.0.1:6379> hmget stu id name age gender
1) "102"
2) "lisi"
3) "21"
4) "male"

127.0.0.1:6379> hgetall stu
1) "id"
2) "102"
3) "name"
4) "lisi"
5) "age"
6) "21"
7) "gender"
8) "male"

```

## list

应用场景：

- 通常应用于微信朋友圈、ins、微博、抖音等
- 简单的说，后入库的数据先访问到，先入库的数据后访问到

```bash
#举例：微信朋友圈更新动态
[root@sql 6379]# redis-cli -a root
127.0.0.1:6379> lpush wechat "today is 1"
(integer) 1
127.0.0.1:6379> lpush wechat "today is 2"
(integer) 2
127.0.0.1:6379> lpush wechat "today is 3"
(integer) 3

#取值
127.0.0.1:6379> lrange wechat 0 -1
1) "today is 3"
2) "today is 2"
3) "today is 1"

```

## set

应用场景：

- 在B站里，将一个up所有的关注人存在一个集合中，将其所有粉丝存在一个集合中
- redis还为集合提供了求交集，并集，差集等操作，可以非常方便的实现如共同关注，共同喜好，二度好友等功能，对所有的集合操作，可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中

```bash
127.0.0.1:6379> sadd lxl pg1 pg2 baoqiang masu marong
(integer) 5
127.0.0.1:6379> sadd jnl baoqiang yufan basobeier zhouxingchi
(integer) 4

#并集操作(两个值集合中其他的所有值)
127.0.0.1:6379> sunion lxl jnl
1) "basobeier"
2) "yufan"
3) "baoqiang"
4) "pg2"
5) "zhouxingchi"
6) "masu"
7) "pg1"
8) "marong"

#交集(集合中共同拥有的值)
127.0.0.1:6379> sinter lxl jnl
1) "baoqiang"

#差集(取后面集合中没有的值)
127.0.0.1:6379> sdiff lxl jnl
1) "masu"
2) "pg1"
3) "marong"
4) "pg2"
127.0.0.1:6379> sdiff jnl lxl
1) "basobeier"
2) "zhouxingchi"
3) "yufan"
```

## sorted set

应用场景：

- 排行榜应用，取top n操作
- 例如网易云听歌次数排序，将要排序的值设置成sorted set 的score,将具体的数据设置成相对应的value，每次只需要执行一条zadd命令即可

```bash
127.0.0.1:6379> zadd music 0 ruyuan 0 chuanqi 0 tianshangrenjian
(integer) 3
127.0.0.1:6379> zincrby music 1000 ruyuan
"1000"
127.0.0.1:6379> zincrby music 1000 chuanqi
"1000"
127.0.0.1:6379> zincrby music 10000 tianshangrenjian
"10000"

#0 -1 从开始到最后
127.0.0.1:6379> zrevrange music 0 -1
1) "tianshangrenjian"
2) "ruyuan"
3) "chuanqi"

127.0.0.1:6379> zrevrange music 0 -1 withscores
1) "tianshangrenjian"
2) "10000"
3) "ruyuan"
4) "1000"
5) "chuanqi"
6) "1000"

```

