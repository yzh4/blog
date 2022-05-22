## 规则练习

清空INPUT链中规则

```
[root@node02 ~]# iptables -F INPUT
[root@node02 ~]#
[root@node02 ~]# iptables -vnL INPUT
Chain INPUT (policy ACCEPT 189 packets, 29462 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

清空INPUT链以后，filter表中的INPUT链已经不存在任何的规则，但是可以看出，INPUT链的默认策略是ACCEPT，也就是说，INPUT链默认"放行"所有发往本机的报文，当没有任何规则时，会接受所有报文，当报文没有被任何规则匹配到时，也会默认放行报文。

那么此刻，我们就在另外一台机器上，使用ping命令，向当前机器发送报文，如下图所示，ping命令可以得到回应，证明ping命令发送的报文已经正常的发送到了防火墙所在的主机，ping命令所在机器IP地址为`192.168.1.34`，当前测试防火墙主机的IP地址为`192.168.1.42`，我们就用这样的环境，对iptables进行操作演示。

```
[root@jenkins01 ~]# ping 192.168.1.42
PING 192.168.1.42 (192.168.1.42) 56(84) bytes of data.
64 bytes from 192.168.1.42: icmp_seq=1 ttl=64 time=0.680 ms
64 bytes from 192.168.1.42: icmp_seq=2 ttl=64 time=0.927 ms
^C
--- 192.168.1.42 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.680/0.803/0.927/0.126 ms
[root@jenkins01 ~]#
```

## 添加规则

> 拒绝来自于34机器的所有报文

添加42机器的规则

```
[root@node02 ~]# iptables -t filter -I INPUT -s 192.168.1.34 -j DROP
[root@node02 ~]# iptables -vnL INPUT
Chain INPUT (policy ACCEPT 12 packets, 962 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       all  --  *      *       192.168.1.34         0.0.0.0/0
```

> 上图中，使用 -t选项指定了要操作的表，此处指定了操作filter表，与之前的查看命令一样，不使用-t选项指定表时，默认为操作filter表。
>
> 使用-I选项，指明将"规则"插入至哪个链中，-I表示insert，即插入的意思，所以-I INPUT表示将规则插入于INPUT链中，即添加规则之意。
>
> 使用-s选项，指明"匹配条件"中的"源地址"，即如果报文的源地址属于-s对应的地址，那么报文则满足匹配条件，-s为source之意，表示源地址。
>
> 使用-j选项，指明当"匹配条件"被满足时，所对应的动作，上例中指定的动作为DROP，在上例中，当报文的源地址为192.168.1.34时，报文则被DROP（丢弃）。
>
> 再次查看filter表中的INPUT链，发现规则已经被添加了，在iptables中，动作被称之为"target"，所以，上图中taget字段对应的动作为DROP。

34机器再访问

```
[root@jenkins01 ~]# ping 192.168.1.42
PING 192.168.1.42 (192.168.1.42) 56(84) bytes of data.
```

能够发现请求直接被丢弃了。

我们再去42机器上检查防火墙数据包信息

```
[root@node02 ~]# iptables -nvL INPUT
Chain INPUT (policy ACCEPT 261 packets, 35021 bytes)
 pkts bytes target     prot opt in     out     source               destination
  168 14112 DROP       all  --  *      *       192.168.1.34         0.0.0.0/0
```

*只要34机器的ping命令不结束，42机器上就会一直收到报文，总大小目前是14112bytes，还会一直增长*

> 再添加一个接受34机器的报文规则

使用如下命令在filter表的INPUT链中追加一条规则，这条规则表示接受所有来自192.168.1.34的发往本机的报文。

```
[root@node02 ~]# iptables -A INPUT -s 192.168.1.34 -p icmp -j ACCEPT
```

上图中的命令并没有使用-t选项指定filter表，我们一直在说，不使用-t选项指定表时表示默认操作filter表。

上图中，使用-A选项，表示在对应的链中"追加规则"，-A为append之意，所以，-A INPUT则表示在INPUT链中追加规则，而之前示例中使用的-I选项则表示在链中"插入规则"，聪明如你一定明白了，它们的本意都是添加一条规则，只是-A表示在链的尾部追加规则，-I表示在链的首部插入规则而已。

使用-j选项，指定当前规则对应的动作为ACCEPT。

执行完添加规则的命令后，再次查看INPUT链，发现规则已经成功"追加"至INPUT链的末尾，那么现在，第一条规则指明了丢弃所有来自192.168.1.34的报文，第二条规则指明了接受所有来自192.168.1.34的报文，那么结果到底是怎样的呢？实践出真知，在34主机上再次使用ping命令向42主机发送报文，发现仍然是ping不通的，看来第二条规则并没有生效。

```
[root@node02 ~]# iptables -vnL INPUT
Chain INPUT (policy ACCEPT 11 packets, 768 bytes)
 pkts bytes target     prot opt in     out     source               destination
  195 16380 DROP       all  --  *      *       192.168.1.34         0.0.0.0/0
    0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0
```

并且可以根据规则的数据包大小查看，第二条规则，没有收到任何的报文。

### 规则加载顺序

其实iptables规则加载是有顺序的，来试着再加一个规则

> 允许接受所有来自于34机器的报文
>
> 看下是否有用，这一次不一样的，是我们把语句添加到最前面。

```
[root@node02 ~]# iptables -I INPUT -s 192.168.1.34 -j ACCEPT
[root@node02 ~]#
[root@node02 ~]#
[root@node02 ~]# iptables -nvL INPUT
Chain INPUT (policy ACCEPT 45 packets, 3132 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     all  --  *      *       192.168.1.34         0.0.0.0/0
  784 65856 DROP       all  --  *      *       192.168.1.34         0.0.0.0/0
    0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0
```

查看42机器是否正常接受

```
[root@jenkins01 ~]# ping 192.168.1.42
PING 192.168.1.42 (192.168.1.42) 56(84) bytes of data.
64 bytes from 192.168.1.42: icmp_seq=1 ttl=64 time=0.350 ms
```

此时我们发现，42机器已经正常可以接受报文了，且报文的统计也有了结果。

```
[root@node02 ~]# iptables -nvL INPUT
Chain INPUT (policy ACCEPT 138 packets, 12642 bytes)
 pkts bytes target     prot opt in     out     source               destination
    3   252 ACCEPT     all  --  *      *       192.168.1.34         0.0.0.0/0
  784 65856 DROP       all  --  *      *       192.168.1.34         0.0.0.0/0
    0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0
```

> 总结：所以iptables的规则是有顺序加载的，自上而下，只要有规则匹配到，符合条件，那么就执行对应的动作
>
> 例如上述，报文被第一条规则匹配到了，且放行了，因为报文已经被放行了，即使第二条规则匹配到了刚才已经被“放行”的报文，也无法进行丢弃操作了。

### 按照号码添加规则

```
1.查看号码
[root@node02 ~]# iptables --line -vnL INPUT
Chain INPUT (policy ACCEPT 308 packets, 43019 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        9   756 ACCEPT     all  --  *      *       192.168.1.34         0.0.0.0/0
2      784 65856 DROP       all  --  *      *       192.168.1.34         0.0.0.0/0
3        0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0

2.在指定编号出插入规则。
[root@node02 ~]# iptables -I INPUT 2 -s 192.168.1.34 -j REJECT
[root@node02 ~]# iptables --line -vnL INPUT
Chain INPUT (policy ACCEPT 13 packets, 916 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        9   756 ACCEPT     all  --  *      *       192.168.1.34         0.0.0.0/0
2        0     0 REJECT     all  --  *      *       192.168.1.34         0.0.0.0/0            reject-with icmp-port-unreachable
3      784 65856 DROP       all  --  *      *       192.168.1.34         0.0.0.0/0
4        0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0
```

此时就插入在了编号为2的位置。a

## 删除规则

- 根据编号删除
- 根据匹配条件删除

```
1.查看规则
[root@node02 ~]# iptables --line -vnL INPUT
Chain INPUT (policy ACCEPT 13 packets, 916 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        9   756 ACCEPT     all  --  *      *       192.168.1.34         0.0.0.0/0
2        0     0 REJECT     all  --  *      *       192.168.1.34         0.0.0.0/0            reject-with icmp-port-unreachable
3      784 65856 DROP       all  --  *      *       192.168.1.34         0.0.0.0/0
4        0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0

2.删除第三条规则
[root@node02 ~]# iptables -t filter -D INPUT 3
[root@node02 ~]# iptables --line -vnL INPUT
Chain INPUT (policy ACCEPT 7 packets, 488 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        9   756 ACCEPT     all  --  *      *       192.168.1.34         0.0.0.0/0
2        0     0 REJECT     all  --  *      *       192.168.1.34         0.0.0.0/0            reject-with icmp-port-unreachable
3        0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0



3.根据具体动作删除
[root@node02 ~]# iptables  -t filter -D INPUT -s 192.168.1.34 -j ACCEPT
[root@node02 ~]#
[root@node02 ~]# iptables --line -vnL INPUT
Chain INPUT (policy ACCEPT 9 packets, 628 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 REJECT     all  --  *      *       192.168.1.34         0.0.0.0/0            reject-with icmp-port-unreachable
2        0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0
```

> 删除某条链所有的规则
>
> iptables -t 表 -F 链
>
> 其实，-F选项不仅仅能清空指定链上的规则，其实它还能清空整个表中所有链上的规则，不指定链名，只指定表名即可删除表中的所有规则，命令如下
>
> iptables -t 表名 -F 链
>
> 不过再次强调，在没有保存iptables规则时，请勿随便清空链或者表中的规则，除非你明白你在干什么。

## 修改规则

修改规则可以用-R选项，修改规则有坑，慎用，建议是，不如删除对应规则，重新添加。

```
# 当前规则
[root@node02 ~]# iptables --line -vnL INPUT
Chain INPUT (policy ACCEPT 9 packets, 628 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 REJECT     all  --  *      *       192.168.1.34         0.0.0.0/0            reject-with icmp-port-unreachable
2        0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0


# 修改条目1，REJECT为DROP
[root@node02 ~]# iptables -t filter -R INPUT 1 -s 192.168.1.34 -j DROP
[root@node02 ~]#
[root@node02 ~]# iptables --line -vnL INPUT
Chain INPUT (policy ACCEPT 9 packets, 628 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 DROP       all  --  *      *       192.168.1.34         0.0.0.0/0
2        0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0
```

> 这里的坑是，修改时，必须填入具体的匹配规则，例如-s地址，否则会导致ip变化，请看

```
[root@node02 ~]# iptables --line -vnL INPUT
Chain INPUT (policy ACCEPT 9 packets, 628 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 DROP       all  --  *      *       192.168.1.34         0.0.0.0/0
2        0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0


[root@node02 ~]# iptables -t filter -R INPUT 1 -j ACCEPT
[root@node02 ~]#
[root@node02 ~]# iptables --line -vnL INPUT
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        9   628 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0
2        0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0
```

> 注意，这里没加上-s，导致source原地址，变为了0.0.0.0，如果动作是REJECT，会导致你的服务器立即拒绝所有流量，ssh断开，无法连接！！！

## 修改默认策略policy

每条链都有自己的策略

```
[root@node02 ~]# iptables --line -vnL
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1      122 12232 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0
2        0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 22 packets, 2752 bytes)
num   pkts bytes target     prot opt in     out     source               destination
```

默认策略，理解为就是默认的动作。

当报文没有被任何的规则匹配到时，防火墙会执行默认的策略，放行或是拒绝。

若是要修改默认策略。

```
# 修改默认策略选项 -P
[root@node02 ~]# iptables -t filter -P FORWARD DROP
[root@node02 ~]#
[root@node02 ~]#
[root@node02 ~]# iptables --line -vnL
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1      708 92461 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0
2        0     0 ACCEPT     icmp --  *      *       192.168.1.34         0.0.0.0/0

Chain FORWARD (policy DROP 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 7 packets, 848 bytes)
num   pkts bytes target     prot opt in     out     source               destination
```

## 保存规则

默认的情况下，我们对"防火墙"所做出的修改都是"临时的"，换句话说就是，当重启iptables服务或者重启服务器以后，我们平常添加的规则或者对规则所做出的修改都将消失，为了防止这种情况的发生，我们需要将规则"保存"。

> centos7保存iptables语句

```
# 安装服务，管理规则
[root@node02 ~]# yum install iptables-services -y

# 停止firewalld
[root@node02 ~]# systemctl stop firewalld
[root@node02 ~]#
[root@node02 ~]# systemctl disable  firewalld
[root@node02 ~]#

# 启用iptables服务
systemctl start iptables

systemctl enable iptables

# 使用命令保存规则
[root@node02 ~]# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
[root@node02 ~]#
[root@node02 ~]#
[root@node02 ~]# cat /etc/sysconfig/iptables
# Generated by iptables-save v1.4.21 on Wed Oct 14 10:53:06 2020
*mangle
:PREROUTING ACCEPT [33135:11923265]
:INPUT ACCEPT [33123:11923025]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [5349:370320]
:POSTROUTING ACCEPT [5349:370320]
COMMIT
# Completed on Wed Oct 14 10:53:06 2020
# Generated by iptables-save v1.4.21 on Wed Oct 14 10:53:06 2020
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [385:36841]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
# Completed on Wed Oct 14 10:53:06 2020
[root@node02 ~]#
```

由于我们只是在filter表中定义了每条链的规则，其他表中并没有设置规则，因此保存的只有filter表的规则。

> 每当我们修改了规则，想要永久生效，必须使用service iptables save保存规则，会写入文件
>
> 如果误操作了规则，也没有保存，可以service iptables restart重新加载文件里的规则。

# iptables匹配条件

## 匹配更多源地址

前面我们的规则都是在对单个ip地址进行匹配，其实也可以指定多个地址段

```
# 清空规则
[root@node02 ~]# iptables -t filter -F

# 插入规则
[root@node02 ~]# iptables -t filter -I INPUT -s 192.168.1.34,192.168.1.33 -j DROP
[root@node02 ~]#
[root@node02 ~]#
[root@node02 ~]# iptables -vnL INPUT
Chain INPUT (policy ACCEPT 42 packets, 3046 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       all  --  *      *       192.168.1.33         0.0.0.0/0
    0     0 DROP       all  --  *      *       192.168.1.34         0.0.0.0/0

# 此时33,34两台机器的流量，就被42机器拒绝了


# 除了指定ip，还可以指定网段192.168.0.1~192.168.255.254，注意语句，别把ssh禁止了
[root@node02 ~]# iptables -t filter -I INPUT -s 192.168.1.0/16 -j ACCEPT
[root@node02 ~]#
[root@node02 ~]#
[root@node02 ~]# iptables -nvL INPUT
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
   61  8092 ACCEPT     all  --  *      *       192.168.0.0/16       0.0.0.0/0
```

条件取反

```
# 条件取反写法
# 只要报文的源ip不是192.168.1.34，就接受此报文（注意，这里不存在说法，如果是34就拒绝！，即使是34也不会自动拒绝）
[root@node02 ~]# iptables -t filter -A INPUT ! -s 192.168.1.34 -j ACCEPT
[root@node02 ~]#
[root@node02 ~]#
[root@node02 ~]# iptables -vnL INPUT
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
   45  3132 ACCEPT     all  --  *      *      !192.168.1.34         0.0.0.0/0

# 34机器尝试访问，还是可以访问的
[root@jenkins01 ~]# ping 192.168.1.42
PING 192.168.1.42 (192.168.1.42) 56(84) bytes of data.
64 bytes from 192.168.1.42: icmp_seq=1 ttl=64 time=0.355 ms
64 bytes from 192.168.1.42: icmp_seq=2 ttl=64 time=0.383 ms
^C
--- 192.168.1.42 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.355/0.369/0.383/0.014 ms
[root@jenkins01 ~]#
```

这是因为，INPUT链默认规则是ACCEPT，该条语句，只是对ip范围取反，但是也没有指定34机器到来的报文就拒绝，因此走了默认规则，所以可以通信。

## 匹配目标地址

> 除了通过-s指定源地址作为条件，还可以通过-d指定目标地址作为匹配条件
>
> 源地址表示报文从哪来
>
> 目标地址表示报文要去哪

```
# 当前机器的ip
[root@node02 ~]# ip a | awk '/inet /{print $1,$2}'
inet 127.0.0.1/8
inet 172.18.0.69/24
inet 192.168.1.42/24
inet 192.168.1.150/24

# 例如该42机器，想要拒绝34机器的请求，且只针对42这个地址，并不禁止向150地址发请求，就可以指定目标地址。

[root@node02 ~]# iptables -t filter -I INPUT -s 192.168.1.34 -d 192.168.1.42 -j DROP

[root@node02 ~]# iptables -vnL INPUT
Chain INPUT (policy ACCEPT 48 packets, 3276 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       all  --  *      *       192.168.1.34         192.168.1.42
[root@node02 ~]#
```

此时34机器向42地址发请求就会被拒绝了，然而150地址还是可以放行。

```
[root@jenkins01 ~]# ping 192.168.1.42
PING 192.168.1.42 (192.168.1.42) 56(84) bytes of data.
^C
--- 192.168.1.42 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2001ms

[root@jenkins01 ~]#
[root@jenkins01 ~]#
[root@jenkins01 ~]# ping 192.168.1.150
PING 192.168.1.150 (192.168.1.150) 56(84) bytes of data.
64 bytes from 192.168.1.150: icmp_seq=1 ttl=64 time=0.362 ms
64 bytes from 192.168.1.150: icmp_seq=2 ttl=64 time=0.898 ms
^C
--- 192.168.1.150 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.362/0.630/0.898/0.268 ms
[root@jenkins01 ~]#
[root@jenkins01 ~]#
[root@jenkins01 ~]#
```

规则的匹配条件，是`与`的关系，也就是得满足`来源地址，目标地址`才会被正确的匹配，执行动作。

## 协议类型

我们可以使用-p参数，指定匹配的报文协议类型，例如tcp，udp等。

```
# 拒绝来自于34的tcp协议请求。
[root@node02 ~]# iptables -t filter -I INPUT -s 192.168.1.34 -d 192.168.1.150 -p tcp -j REJECT
[root@node02 ~]#
[root@node02 ~]#
[root@node02 ~]# iptables -vnL INPUT
Chain INPUT (policy ACCEPT 41 packets, 2852 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 REJECT     tcp  --  *      *       192.168.1.34         192.168.1.150        reject-with icmp-port-unreachable
```

34机器发请求测试

> ssh属于tcp协议
>
> ping属于icmp协议

```
[root@jenkins01 ~]# ssh root@192.168.1.150
ssh: connect to host 192.168.1.150 port 22: Connection refused
[root@jenkins01 ~]#
[root@jenkins01 ~]# ping 192.168.1.150
PING 192.168.1.150 (192.168.1.150) 56(84) bytes of data.
64 bytes from 192.168.1.150: icmp_seq=1 ttl=64 time=0.357 ms
64 bytes from 192.168.1.150: icmp_seq=2 ttl=64 time=0.336 ms
^C
--- 192.168.1.150 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.336/0.346/0.357/0.021 ms
```

> centos7中，-p选项支持如下协议类型
>
> tcp, udp, udplite, icmp, icmpv6,esp, ah, sctp, mh
>
> 当不使用-p指定协议类型时，默认表示所有类型的协议都会被匹配到，与使用-p all的效果相同。

## 网卡接口

每台Linux服务器或许会有多个网卡，可以通过`iptables 的-i选项`匹配报文从哪块网卡进入。

```
# 查看网络接口
ifconfig

# 拒绝ens37进入的ping请求
[root@node02 ~]# iptables -t filter -I INPUT -i ens37 -p icmp -j DROP
[root@node02 ~]#
[root@node02 ~]# iptables -nvL
Chain INPUT (policy ACCEPT 27 packets, 1872 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       icmp --  ens37  *       0.0.0.0/0            0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 14 packets, 1320 bytes)
 pkts bytes target     prot opt in     out     source               destination
[root@node02 ~]#
```

发请求测试

```
[root@jenkins01 ~]# ping 192.168.1.150
PING 192.168.1.150 (192.168.1.150) 56(84) bytes of data.
^C
--- 192.168.1.150 ping statistics ---
6 packets transmitted, 0 received, 100% packet loss, time 5001ms

[root@jenkins01 ~]# ping 192.168.1.42
PING 192.168.1.42 (192.168.1.42) 56(84) bytes of data.


^C
--- 192.168.1.42 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1000ms


# ssh是tcp协议，所以可以通过
[root@jenkins01 ~]# ssh root@192.168.1.42
Last login: Wed Oct 14 14:39:19 2020 from 192.168.1.34
[root@node02 ~]#
```

## 流量 进/出

> -i选项只能用于PREROUTING链、INPUT链、FORWARD链，控制流量进入
>
> -o选项只能用于FORWARD链、OUTPUT链、POSTROUTING链，控制流量的出去
>
> -o选项是用于匹配报文将由哪个网卡"流出"的

## 端口匹配

### --dport

> 使用--dport可以匹配报文的目标端口，destination-port，目标端口
>
> 使用该参数，必须指定好某种协议，也就是必须用-p参数
>
> -m 参数，指定对应的扩展模块，使用--dport必须指定某个扩展模块，也就是-m参数

```
[root@node02 ~]# iptables -t filter -I INPUT -s 192.168.1.34 -p tcp -m tcp --dport 22 -j REJECT
[root@node02 ~]#
[root@node02 ~]#
[root@node02 ~]# iptables -nvL
Chain INPUT (policy ACCEPT 29 packets, 2012 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 REJECT     tcp  --  *      *       192.168.1.34         0.0.0.0/0            tcp dpt:22 reject-with icmp-port-unreachable
```

> 这里要记住的是iptables在指定端口匹配时，必须指定-p 协议，-m 扩展模块

### 匹配多个目标端口

> iptables可以基于tcp扩展模块的--dport选项，指定一个连续的端口范围，例如22:25，也就是22，23，24，25。
>
> 但是无法指定多个离散不连续的端口，这就得借助于multiport扩展模块，该模块只支持tcp，udp

```
[root@node02 ~]# iptables -t filter -I INPUT -s 192.168.1.34 -p tcp -m multiport --dports 22,3306,80 -j DROP
[root@node02 ~]#
[root@node02 ~]# iptables -vnL INPUT
Chain INPUT (policy ACCEPT 39 packets, 2712 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       tcp  --  *      *       192.168.1.34         0.0.0.0/0            multiport dports 22,3306,80
    0     0 DROP       tcp  --  *      *       192.168.1.34         0.0.0.0/0            tcp spt:22
[root@node02 ~]#
```

> 上述就是禁止了34机器，访问该node02机器的22,3306,80三个端口

## 常用扩展模块

在上一节针对多个端口的规则匹配，我们使用了`-m multiport`模块，这一节再看看其他模块。

### iprange扩展模块

> 前提
>
> 在不使用扩展模块情况下，使用-s或-d选项可以匹配报文的源地址和目标地址
>
> 指定ip地址时，-s选项可以指定多个IP地址
>
> 若是要指定一段连续的IP范围地址，可以用iprange扩展模块。
>
> --src-range 源地址范围
>
> --dst-range 目标地址范围

```
[root@chaoge01 ~]# iptables -t filter -I INPUT -m iprange --src-range 192.168.1.10-192.168.1.40 -j DROP
[root@chaoge01 ~]#
[root@chaoge01 ~]#
[root@chaoge01 ~]# iptables -vnL INPUT
Chain INPUT (policy ACCEPT 15 packets, 1064 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            source IP range 192.168.1.10-192.168.1.40
```

测试

```
[root@jenkins01 ~]# ssh root@192.168.1.42

^C
[root@jenkins01 ~]#
[root@jenkins01 ~]#
[root@jenkins01 ~]# ping 192.168.1.42
PING 192.168.1.42 (192.168.1.42) 56(84) bytes of data.
^C
--- 192.168.1.42 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 999ms

[root@jenkins01 ~]#
```

### string扩展模块

string扩展模块，是根据报文中对应的字符串，进行条件匹配。

> -m string 指定string模块
>
> --algo bm指定用bm算法匹配字符串
>
> --string "xxoo" 定义字符串内容，可以是URL里任意字符，如果是需要block下载某些类型的文件或请求，这个有很大的发挥空间。

```
# 准备好一台nginx服务器，展示页面34地址的机器
[root@chaoge01 ~]# curl 192.168.1.34/index.html
hello,xxoo

# 在42机器上添加防火墙规则，禁止含有xxoo的报文，进入42这台机器
[root@chaoge01 ~]# iptables -F
[root@chaoge01 ~]#
[root@chaoge01 ~]#
[root@chaoge01 ~]# iptables -t filter -I INPUT -m string --algo bm --string "xxoo" -j REJECT
[root@chaoge01 ~]#
[root@chaoge01 ~]#
[root@chaoge01 ~]# curl 192.168.1.34/index.html

[root@chaoge01 ~]# curl 192.168.1.34
```

> 我们会发现请求无法进入42这台机器了

利用这个功能可以实现一些防止入侵的规则。

> 保护该机器的80端口，拒收含有cmd.exe字符串的报文请求

```
# 在42这台机器添加
[root@chaoge01 ~]# iptables -I INPUT 1 -p tcp --dport 80 -m string --string "cmd.exe" --algo bm -j DROP

# 客户端测试，无法访问
[root@jenkins01 ~]# curl 192.168.1.42/123cmd.exe123123
```

> 防止电子邮件诈骗

```
iptables -I INPUT -p tcp --dport 25 -m string --string "BTC" --algo bm -j DROP
```

### time模块

通过time模块进行根据时间段区匹配，报文到达的时间范围内，就符合匹配规则。

> 限制早上9点到晚上6点，该机器，无法上网（访问80，443端口）
>
> 思路：限制该机器，出口的流量
>
> -m time 表示time模块
>
> --timestart 表示时间的开始时间
>
> --timestop 停止时间

```
# 此时电脑就无法发出80端口的请求了
[root@chaoge01 ~]# iptables -t filter -I OUTPUT -p tcp --dport 80 -m time --timestart 09:00:00 --timestop 22:00:00 -j REJECT

# 无法发出
[root@chaoge01 ~]# curl http://baidu.com

# 禁止443端口的流量发出
[root@chaoge01 ~]# iptables -t filter -I OUTPUT -p tcp --dport 443 -m time --timestart 09:00:00 --timestop 22:00:00 -j REJECT
```

> 禁止访问百度页面

```
[root@chaoge01 ~]# iptables -t filter -I OUTPUT -m string --algo bm --string "baidu" -j REJECT
[root@chaoge01 ~]#
```

> 周六日，禁止访问网页
>
> --weekdays 可以指定每个星期具体到哪一天，可以指定多个数字或Mon,Tue,Wed

```
[root@chaoge01 ~]# iptables -t filter -I OUTPUT -p tcp --dport 80 -m time --weekdays 6,7 -j REJECT
```

> 周六日的具体时间段，禁止浏览网页

```
[root@chaoge01 ~]# iptables -t filter -I OUTPUT -p tcp --dport 80 -m time --weekdays 6,7 --timestart 09:00:00 --timestop 18:00:00 -j REJECT
```

### connlimit扩展模块

该模块可以限制每个ip地址同时连接到server的链接数量。

> 限制每个ip地址，只能连接2个ssh终端到server

```
[root@chaoge01 ~]# iptables -I INPUT -p tcp --dport 22 -m connlimit --connlimit-above 2 -j REJECT
```