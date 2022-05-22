## 查看filter表中的规则

本地的虚拟机，开机默认没有规则，规则我们可以自己手动添加

```
[root@node02 ~]# iptables -t filter -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

> 语法
>
> iptables -t filter -L
>
> iptables -t raw -L
>
> iptables -t mangle -L
>
> iptables -t nat -L
>
> -t选项对应表，若是不指定表，默认就是filter表
>
> -L选项列出该表规则

## 查看INPUT链规则

本地的虚拟机，开机默认没有规则，规则我们可以自己手动添加

```
[root@node02 ~]# iptables -L INPUT
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
```

这样看的链信息不详细，可以：

```
[root@node02 ~]# iptables -vL INPUT
Chain INPUT (policy ACCEPT 334 packets, 31525 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

使用-v参数后，显示的信息更详细了

## iptables字段属性

> 其实，这些字段就是规则对应的属性，说白了就是规则的各种信息，那么我们来总结一下这些字段的含义。
>
> **pkts**:对应规则匹配到的报文的个数。
>
> **bytes**:对应匹配到的报文包的大小总和。
>
> **target**:规则对应的target，往往表示规则对应的"动作"，即规则匹配成功后需要采取的措施。
>
> **prot**:表示规则对应的协议，是否只针对某些协议应用此规则。
>
> **opt**:表示规则对应的选项。
>
> **in**:表示数据包由哪个接口(网卡)流入，我们可以设置通过哪块网卡流入的报文需要匹配当前规则。
>
> **out**:表示数据包由哪个接口(网卡)流出，我们可以设置通过哪块网卡流出的报文需要匹配当前规则。
>
> **source**:表示规则对应的源头地址，可以是一个IP，也可以是一个网段。
>
> **destination**:表示规则对应的目标地址。可以是一个IP，也可以是一个网段。

## 查看规则命令

来看一些存在规则的情况

```
[root@chaogelinux ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
DOCKER-ISOLATION  all  --  anywhere             anywhere
DOCKER     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
ACCEPT     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain DOCKER (1 references)
target     prot opt source               destination
ACCEPT     tcp  --  anywhere             172.17.0.2           tcp dpt:mysql
ACCEPT     tcp  --  anywhere             172.17.0.3           tcp dpt:teradataordbms

Chain DOCKER-ISOLATION (1 references)
target     prot opt source               destination
RETURN     all  --  anywhere             anywhere
```

每个人的防火墙规则，都不会一模一样，这里的是超哥一台docker服务器上的防火墙规则，这里我们先不用在意，只需要知道，docker容器技术的通信是借助于iptables实现的。

细心如你一定发现了，上图中的源地址与目标地址都为anywhere，看来，iptables默认为我们进行了名称解析，但是在规则非常多的情况下如果进行名称解析，效率会比较低，所以，在没有此需求的情况下，我们可以使用-n选项，表示不对IP地址进行名称反解，直接显示IP地址，示例如下。

```
[root@chaogelinux ~]# iptables -vL -n
Chain INPUT (policy ACCEPT 7245K packets, 510M bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
60164  104M DOCKER-ISOLATION  all  --  *      *       0.0.0.0/0            0.0.0.0/0
32219  101M DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0
21174  100M ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
27945 3820K ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0

Chain OUTPUT (policy ACCEPT 7295K packets, 576M bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain DOCKER (1 references)
 pkts bytes target     prot opt in     out     source               destination
10070  758K ACCEPT     tcp  --  !docker0 docker0  0.0.0.0/0            172.17.0.2           tcp dpt:3306
  762  100K ACCEPT     tcp  --  !docker0 docker0  0.0.0.0/0            172.17.0.3           tcp dpt:8002

Chain DOCKER-ISOLATION (1 references)
 pkts bytes target     prot opt in     out     source               destination
60164  104M RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0
[root@chaogelinux ~]#
```

如上图所示，规则中的源地址与目标地址已经显示为IP，而非转换后的名称。

当然，我们也可以只查看某个链的规则，并且不让IP进行反解，这样更清晰一些，比如 iptables -nvL FORWARD

```
[root@chaogelinux ~]# iptables -nvL FORWARD
Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
60164  104M DOCKER-ISOLATION  all  --  *      *       0.0.0.0/0            0.0.0.0/0
32219  101M DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0
21174  100M ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
27945 3820K ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0
```

如果你习惯了查看有序号的列表，你在查看iptables表中的规则时肯定会很不爽，没有关系，满足你，使用--line-numbers即可显示规则的编号，示例如下。

```
[root@chaogelinux ~]# iptables --line-number -nvL FORWARD
Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1    60164  104M DOCKER-ISOLATION  all  --  *      *       0.0.0.0/0            0.0.0.0/0
2    32219  101M DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0
3    21174  100M ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
4    27945 3820K ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0
5        0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0
```

--line-numbers选项并没有对应的短选项，不过我们缩写成--line时，centos中的iptables也可以识别。

我知道你目光如炬，你可能早就发现了，表中的每个链的后面都有一个括号，括号里面有一些信息，如下图红色标注位置，那么这些信息都代表了什么呢？我们来看看。

> 上图中INPUT链后面的括号中包含policy ACCEPT ，0 packets，0bytes 三部分。
>
> **policy**表示当前链的默认策略，policy ACCEPT表示上图中INPUT的链的默认动作为ACCEPT，换句话说就是，默认接受通过INPUT关卡的所有请求，所以我们在配置INPUT链的具体规则时，应该将需要拒绝的请求配置到规则中，说白了就是"黑名单"机制，默认所有人都能通过，只有指定的人不能通过，当我们把INPUT链默认动作设置为接受(ACCEPT)，就表示所有人都能通过这个关卡，此时就应该在具体的规则中指定需要拒绝的请求，就表示只有指定的人不能通过这个关卡，这就是黑名单机制，**但是**，你一定发现了，上图中所显示出的规则，大部分都是接受请求(ACCEPT)，并不是想象中的拒绝请求(DROP或者REJECT)，这与我们所描述的黑名单机制不符啊，按照道理来说，默认动作为接受，就应该在具体的规则中配置需要拒绝的人，但是上图中并不是这样的，之所以出现上图中的情况，是因为IPTABLES的工作机制导致到，上例其实是利用了这些"机制"，完成了所谓的"白名单"机制，并不是我们所描述的"黑名单"机制，我们此处暂时不用关注这一点，之后会进行详细的举例并解释，此处我们只要明白policy对应的动作为链的默认动作即可，或者换句话说，
>
> 我们只要理解，policy为链的默认策略即可。
>
> **packets**表示当前链（上例为INPUT链）默认策略匹配到的包的数量，0 packets表示默认策略匹配到0个包。
>
> **bytes**表示当前链默认策略匹配到的所有包的大小总和。
>
> 其实，我们可以把packets与bytes称作"计数器"，上图中的计数器记录了默认策略匹配到的报文数量与总大小，"计数器"只会在使用-v选项时，才会显示出来。
>
> 当被匹配到的包达到一定数量时，计数器会自动将匹配到的包的大小转换为可读性较高的单位，如下图所示。