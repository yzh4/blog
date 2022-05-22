## redis安全配置

- redis没有用户概念，redis只有密码
- redis默认在工作在保护模式下,不允许远程用户登录

## 尝试远程登录

```
[root@docker01 ~]# redis-cli -h 192.168.6.200
192.168.6.200:6379> set name zhangsan
(error) DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In this mode connections are only accepted from the loopback interface.
If you want to connect from external computers to Redis you may adopt one of the following solutions: 
1) Just disable protected mode sending the command 'CONFIG SET protected-mode no' from the loopback interface by connecting to Redis from the same host the server is running, however MAKE SURE Redis is not publicly accessible from internet if you do so. Use CONFIG REWRITE to make this change permanent. 
2) Alternatively you can just disable the protected mode by editing the Redis configuration file, and setting the protected mode option to 'no', and then restarting the server. 
3) If you started the server manually just for testing, restart it with the '--protected-mode no' option. 
4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.

```

```
1)只需要在loopback接口上使用CONFIG SET protectmode no命令来禁用保护模式，并确保你不能从互联网上公开访问Redis。使用配置重写使此更改永久。
2)或者，你可以通过编辑Redis配置文件来禁用保护模式，并将保护模式选项设置为“no”，然后重新启动服务器。
3)如果你手动启动服务器只是为了测试，用“——protected-mode no”选项重新启动它。
4)设置绑定地址或认证密码。注意:为了让服务器开始接受来自外部的连接，您只需要做上面的一件事。
```

- protected-mode yes/no(保护模式，是否只允许本地访问)

## 设置登录规则

(1)bind：指定ip进行监听

(2)增加requirepass root

```bash
 [root@docker01 ~]# cat /data/6379/redis.conf
 bind 192.168.6.200 127.0.0.1
 requirepass root
```

## 重启生效

```bash
[root@docker01 ~]# redis-cli shutdown
[root@docker01 ~]# redis-server /data/6379/redis.conf
```

## 验证

```bash
#方法一
[root@docker01 ~]# redis-cli -h 192.168.6.200 -a root
192.168.6.200:6379> set name zhangsan
OK
192.168.6.200:6379> get name
"zhangsan"

#方法二
[root@docker01 ~]# redis-cli
127.0.0.1:6379> set name zhangsan
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth root
OK

```

## 获取所有配置参数

```bash
[root@docker01 ~]# redis-cli -a root
127.0.0.1:6379> config get *
```

## 在线修改配置

```bash
127.0.0.1:6379> config set requirepass 1234
OK

[root@docker01 ~]# redis-cli -a root
127.0.0.1:6379> set a b
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth 1234
OK
127.0.0.1:6379> set a b
OK
127.0.0.1:6379> get a
"b"
```



