## redis 各个版本下载地址

http://download.redis.io/releases/

```shell
#下载
wget http://download.redis.io/releases/redis-3.2.12.tar.gz

#解压缩
tar -xzf redis-3.2.12.tar.gz -C /data

#安装
cd redis-3.2.12
make

#修改配置文件
vim /etc/profile
export PATH=/data/redis3.2.12/src:$PATH

[root@docker01 data]# source /etc/profile

```

## 启动redis

```
[root@docker01 data]# redis-server &
[1] 43891
[root@docker01 data]# 43891:C 08 Feb 19:38:00.062 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.2.12 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 43891
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'            
```

```
[root@docker01 data]# ps -ef | grep redis
root      43891  43174  0 19:37 pts/0    00:00:00 redis-server *:6379
root      43975  43174  0 19:39 pts/0    00:00:00 grep --color=auto redis
```

```
[root@docker01 data]# redis-cli
127.0.0.1:6379> set name zhangsan
OK
127.0.0.1:6379> get name
"zhangsan"

```

