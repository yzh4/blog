## 通配符

就是键盘上的一些特殊字符，可以实现特殊的功能，例如模糊搜索一些文件,利用通配符可以更轻松的处理字符信息。

### 常见通配符

| 符号     | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| *        | 匹配任意，0或多个字符，字符串                                |
| ?        | 匹配任意1个字符，有且只有一个字符                            |
| 符号集合 | 匹配一堆字符或文本                                           |
| [abcd]   | 匹配abcd中任意一个字符，abcd也可以是不连续任意字符           |
| [a-z]    | 匹配a到z之间任意一个字符，要求连续字符，也可以连续数字，匹配[1-9] |
| [!abcd]  | 不匹配括号中任意一个字符，也可以书写[!a-d]，同于写法         |
| [^abcd]  | 同上，！可以换成 ^                                           |

### 特殊通配符

| 符号          | 作用             |
| ------------- | ---------------- |
| `[[:upper:]]` | 所有大写字母     |
| `[[:lower:]]` | 所有小写字母     |
| `[[:alpha:]]` | 所有字母         |
| `[[:digit:]]` | 所有数字         |
| `[[:alnum:]]` | 所有的字母和数字 |
| `[[:space:]]` | 所有的空白字符   |
| `[[:punct:]]` | 所有标点符号     |

案例

```
1.找出根目录下最大文件夹深度是3，且所有以l开头的文件，且以小写字母结尾，中间出现任意一个字符的文本文件，
[root@chaogelinux luffy_data]# find / -maxdepth 3 -type f -name "l?[[:lower:]]"
/usr/bin/ldd
/usr/bin/lua
/usr/sbin/lvm
/usr/sbin/lid

2.找出/tmp下以任意一位数字开头，且以非数字结尾的文件
[root@chaogelinux luffy_data]# find /tmp  -type f -name '[0-9][^0-9]'

3.显示/tmp下以非字母开头，后面跟着一个字母以及其他任意长度的字符的文件
find /tmp -type f -name "[^a-zA-Z][a-z]*"

4.mv/tmp/下，所有以非字母开头的文件，复制到/tmp/allNum/目录下
mv /tmp/[^a-zA-Z]* /tmp/allNum/

5.复制/tmp目录下所有的.txt文件结尾的文件，且以y、t开头的文件，放入/data目录
[root@chaogelinux tmp]# cp -r /tmp/[y,t]*.txt /data/
```

### *号

准备数据如下

```
[root@pylinux tmp]# ls
chaoge.sh  chaoge.txt  oldboy.txt  oldgirl.txt  oldluffy.txt  yu.sh
```

1.匹配所有的txt文本

```
[root@pylinux tmp]# ls
chaoge.sh  chaoge.txt  oldboy.txt  oldgirl.txt  oldluffy.txt  yu.sh
```

2.匹配所有的sh脚本

```
[root@pylinux tmp]# ls *.sh
chaoge.sh  yu.sh
```

3.查看所有的old开头的文件

```
[root@pylinux tmp]# ls old*
oldboy.txt  oldgirl.txt  oldluffy.txt
```

### ？号

1.匹配一个任意字符

```
[root@pylinux tmp]# ls *.s?
chaoge.sh  yu.sh
```

2.匹配两个，三个任意字符

```
[root@pylinux tmp]# ls old???.txt
oldboy.txt
[root@pylinux tmp]# ls old????.txt
oldgirl.txt
```

### [abc]

1.匹配[abc]中的一个字符

```
[root@pylinux tmp]# ls
a.txt  b.txt  chaoge.sh  chaoge.txt  l.txt  oldboy.txt  oldgirl.txt  oldluffy.txt  yu.sh

[root@pylinux tmp]# ls [abc].txt
a.txt  b.txt
```

2.匹配如下类型文件

olda*.txt oldb*.txt oldc*.txt

```
[root@pylinux tmp]# ls old[abcd]*.txt
oldboy.txt
```

### [a-z]作用

[]中括号里面的a-z，表示从a到z之间的任意一个字符，a-z要连续，也可以用连续的数字，如[1-9]

### [!abcd]

```
[^abcd]  
[^a-d]
效果同上
```

除了abcd四个字符以外的任意一个字符

### 结合find命令使用

```
[root@pylinux tmp]# find /tmp  -type f  -name "[a-z].txt"        #找出a到z之间单个字符的文件
/tmp/b.txt
/tmp/e.txt
/tmp/a.txt
/tmp/l.txt
/tmp/d.txt
/tmp/c.txt

[root@pylinux tmp]# find /tmp  -type f  -name "[!a-d].txt"        #找出除了a到d之间单个字符的文件
/tmp/e.txt
/tmp/2.txt
/tmp/1.txt
/tmp/l.txt

[root@pylinux tmp]# find /tmp  -type f  -name "?.txt"            #找出所有单个字符的文件
/tmp/b.txt
/tmp/e.txt
/tmp/2.txt
/tmp/a.txt
/tmp/1.txt
/tmp/l.txt
/tmp/d.txt
/tmp/c.txt

[root@pylinux tmp]# find /tmp  -type f  -name "*.txt"            #找出所有的txt文本
```

## Linux特殊符号

### 路径相关

| 符号 | 作用                                 |
| ---- | ------------------------------------ |
| ~    | 当前登录用户的家目录                 |
| -    | 上一次工作路径                       |
| .    | 当前工作路径，或隐藏文件 .chaoge.txt |
| ..   | 上一级目录                           |

波浪线案例

```
[root@pylinux tmp]# cd ~
[root@pylinux ~]# pwd
/root

[yu@pylinux ~]$
[yu@pylinux ~]$ pwd
/home/yu
```

短横杠案例

```
[root@pylinux opt]# cd /tmp
[root@pylinux tmp]# cd -
/opt
[root@pylinux opt]# cd -
/tmp

[root@pylinux tmp]# echo $OLDPWD
/opt
```

点案例

```
[root@pylinux tmp]# find .  -name "*.sh"
./yu.sh
./chaoge.sh
```

点点案例

```
[root@pylinux tmp]# mkdir ../opt/超哥nb
[root@pylinux tmp]#
[root@pylinux tmp]# ls /opt/
超哥nb

[root@pylinux home]# ls -a
.  ..  py  testyu  yu
```

### 特殊引号

在linux系统中，单引号、双引号可以用于表示字符串

| 名称      | 解释                                                         |
| --------- | ------------------------------------------------------------ |
| 单引号 '' | 所见即所得，强引用，单引号中内容会原样输出                   |
| 双引号 "" | 弱引用，能够识别各种特殊符号、变量、转义符等，解析后再输出结果 |
| 没有引号  | 一般连续字符串、数字、路径可以省略双引号，遇见特殊字符，空格、变量等，必须加上双引号 |
| 反引号 `` | 常用于引用命令结果，同于$(命令)                              |

### 反引号案例

用反引号进行命令解析

```
[root@pylinux tmp]# date +%F
2019-11-05
[root@pylinux tmp]# touch `date +%F`.txt    #创建文件，名字是当前时间格式
[root@pylinux tmp]# ls
2019-11-05.txt

[root@pylinux tmp]# echo date
date
[root@pylinux tmp]# echo `date`        #反引号中命令会被执行
2019年 11月 05日 星期二 16:29:28 CST

[root@pylinux tmp]# which cat
/usr/bin/cat
[root@pylinux tmp]#
[root@pylinux tmp]# ls -l `which cat`        #反引号中命令被执行
-rwxr-xr-x. 1 root root 54080 4月  11 2018 /usr/bin/cat
```

### 双引号案例

当输出双引号内所有内容时，内容中有命令需要用反引号标记

```
[root@pylinux tmp]# echo "date"
date
[root@pylinux tmp]#
[root@pylinux tmp]#
[root@pylinux tmp]# echo "`date`"
2019年 11月 05日 星期二 16:30:42 CST

[root@pylinux tmp]# echo "今天是星期 `date +%w`"
今天是星期 2

[root@pylinux tmp]# echo "今年是$(date +%Y)年"
今年是2019年
```

### 单引号案例

单引号中内容是强引用，保持原样输出

```
[root@pylinux tmp]# echo "今天日期是 `date +%F`"        #双引号可以
今天日期是 2019-11-05

[root@pylinux tmp]# echo '今天日期是 `date +%F`'        #单引号不可以
今天日期是 `date +%F`
```

### 无引用案例

没有引号，很难确定字符串的边界，且linux命令是以空格区分的

建议用双引号代替不加引号

```
[root@pylinux tmp]# echo "今天是 `date +%Y`年"
今天是 2019年

[root@pylinux tmp]# echo 今天是 `date +%Y`年
今天是 2019年

[root@pylinux tmp]# ls "/tmp"
2019-11-05.txt
[root@pylinux tmp]#
[root@pylinux tmp]# ls /tmp
2019-11-05.txt
```

## 输出重定向特殊符号

输入设备

- 键盘输入数据
- 文件数据导入

输出设备

- 显示器、屏幕终端
- 文件

### 数据流

程序的数据流：

- 输入流：<---标准输入 （stdin），键盘
- 输出流：-->标准输出（stdout），显示器，终端
- 错误输出流：-->错误输出（stderr）

### 文件描述符

在Linux系统中，一切设备都看作文件。

而每打开一个文件，就有一个代表该打开文件的文件描述符。

程序启动时默认打开三个I/O设备文件：

- 标准输入文件stdin，文件描述符0
- 标准输出文件stdout，文件描述符1
- 标准错误输出文件stderr，文件描述符2

| 符号                   | 特殊符号            | 简介                                 |
| ---------------------- | ------------------- | ------------------------------------ |
| 标准输入stdin          | 代码为0，配合< 或<< | 数据流从右向左                       |
| 标准输出stdout         | 代码1，配合>或>>    | 数据从左向右                         |
| 标准错误stderr         | 代码2，配合>或>>    | 数据从左向右                         |
|                        |                     |                                      |
| 重定向符号             |                     | 数据流是箭头方向                     |
| 标准输入重定向         | 0< 或 <             | 数据一般从文件流向处理命令           |
| 追加输入重定向         | 0<<或<<             | 数据一般从文件流向处理命令           |
| 标准输出重定向         | 1>或>               | 正常输出重定向给文件，默认覆盖       |
| 标准输出追加重定向     | 1>>或>>             | 内容追加重定向到文件底部，追加       |
| 标准错误输出重定向     | 2>                  | 讲标准错误内容重定向到文件，默认覆盖 |
| 标准错误输出追加重定向 | 2>>                 | 标准错误内容追加到文件底部           |

错误输出

```
[root@chaogelinux tmp]# ls yyy
ls: 无法访问yyy: 没有那个文件或目录
[root@chaogelinux tmp]#
[root@chaogelinux tmp]# ls yyy > cuowu.txt
ls: 无法访问yyy: 没有那个文件或目录
[root@chaogelinux tmp]# ls yyy >> cuowu.txt
ls: 无法访问yyy: 没有那个文件或目录
[root@chaogelinux tmp]# ls yyy 2> cuowu.txt
[root@chaogelinux tmp]#
[root@chaogelinux tmp]#
[root@chaogelinux tmp]# cat cuowu.txt
ls: 无法访问yyy: 没有那个文件或目录
```

### 特殊重定向，合并重定向

- `2>&1`把标准错误，重定向到标准输出

把命令的执行结果写入文件，标准错误当做标准输出处理，也写入文件

- Command > /path/file 2>&1

```
echo "I am oldboy" 1>>oldboy.txt 2>>oldboy.txt

echo  "I am oldboy"  >> /dev/null 2>&1            #  命令已经将结果重定向到/dev/null，2>&1符号也会将标准错误输出到/dev/null，/dev/null是一个黑洞，只写文件
```

### 输入重定向

数据流输入

```
[root@chaogelinux tmp]# cat < yu2.txt
我是 yu2，你是谁，想偷看我？

#mysql数据导入
mysql -uroot -p < db.sql

[root@chaogelinux ~]# cat chaoge.txt
a b c d e f g
[root@chaogelinux ~]# tr -d 'a-c' < chaoge.txt
   d e f g

[root@chaogelinux ~]# wc -l < chaoge.txt
1
```

### 其他特殊符号

| 符号 | 解释                                          |
| ---- | --------------------------------------------- |
| ;    | 分号，命令分隔符或是结束符                    |
| #    | 1.文件中注释的内容 2.root身份提示符           |
|      | 管道符，传递命令结果给下一个命令              |
| $    | 1.$变量，取出变量的值 2.普通用户身份提示符    |
| \    | 转义符，将特殊含义的字符还原成普通字符        |
| {}   | 1.生成序列 2.引用变量作为变量与普通字符的分割 |

### ；号

- 表示命令的结束
- 命令间的分隔符
- 配置文件的注释符

```
[root@pylinux tmp]# pwd;ls            #执行两条命令
/tmp
2019-11-05.txt  oldboy.txt  txt
```

### #号

- 注释行

```
# nginx.conf

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;
```

### |号

比如生活中的管道，能够传输物质

Linux管道符 | 用于传输数据，对linux命令的处理结果再次处理，直到得到最终结果

```
[root@pylinux ~]# ifconfig |grep inet
        inet 10.141.32.137  netmask 255.255.192.0  broadcast 10.141.63.255
        inet 127.0.0.1  netmask 255.0.0.0

[root@pylinux tmp]# ls | grep .txt
2019-11-05.txt
oldboy.txt

[root@pylinux tmp]# netstat -tunlp|grep 22
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1401/sshd


[root@pylinux tmp]# netstat -tunlp|grep 3306

[root@pylinux tmp]# netstat -tunlp|grep 80

[root@pylinux tmp]# ps aux|grep python

[root@pylinux tmp]# ps aux|grep mariadb
```

能一次出结果的命令，尽量不要二次过滤，效率并不高

### $符

Linux系统命令行中，字符串前加$符，代表字符串变量的值

```
[root@pylinux tmp]# echo OLDPWD
OLDPWD
[root@pylinux tmp]# echo $OLDPWD
/root
[root@pylinux tmp]# echo PATH
PATH
[root@pylinux tmp]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/python37/bin:/root/bin
[root@pylinux tmp]# name="超哥带你学linux"
[root@pylinux tmp]# echo name
name
[root@pylinux tmp]# echo $name
超哥带你学linux
```

### {}符

1.生成序列，一连串的文本

```
[root@pylinux tmp]# echo {1..100}
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100

[root@pylinux tmp]# echo {a..i}
a b c d e f g h i

[root@pylinux tmp]# echo {i..a}
i h g f e d c b a

[root@pylinux tmp]# echo {1..9}
1 2 3 4 5 6 7 8 9

[root@pylinux tmp]# echo {o,l,d}
o l d
```

2.利用{}快速备份文件

```
[root@pylinux tmp]# cp /etc/sysconfig/network-scripts/ifcfg-eth0{,.ori}
```

3.将变量括起来作为变量的分隔符

```
[root@pylinux tmp]# echo $week
3
[root@pylinux tmp]# echo "$week_oldboy"      #输出为空，系统人为week_oldboy是整个变量

[root@pylinux tmp]# echo "${week}_oldboy"        #花括号中才会识别是变量，作了分割
3_oldboy
```

### 逻辑操作符

逻辑符既可以在linux系统中直接用，也可以在Bash脚本中用

| 命令 | 解释                                                         |      |                                    |
| ---- | ------------------------------------------------------------ | ---- | ---------------------------------- |
| &&   | 前一个命令成功，再执行下一个命令                             |      |                                    |
| \|\| | \|\|                                                         |      | 前一个命令失败了，再执行下一个命令 |
| !    | 1.在bash中取反 2.在vim中强制性 3.历史命令中 !ls找出最近一次以ls开头的命令 |      |                                    |

1.&&案例

```
[root@pylinux tmp]# ls && cd /opt && pwd        #执行成功会继续下一个命令
2019-11-05.txt  oldboy.txt  txt
/opt

[root@pylinux opt]# ls /tmpp && cd /tmp        #执行失败就结束
ls: 无法访问/tmpp: 没有那个文件或目录
```

1. ||案例

```
[root@pylinux opt]# ls /tmpp || cd /tmp            #执行失败才会继续下一个命令
ls: 无法访问/tmpp: 没有那个文件或目录
[root@pylinux tmp]#

[root@pylinux tmp]# cd /opt || cd /root            #执行成功则不会继续下一个命令
[root@pylinux opt]#
```

1. 感叹号
2. 取反

```
[root@pylinux tmp]# ls [a-f]*
a  b  c  d  e  f
[root@pylinux tmp]# ls [!a-f]*        #取反的意思
2019-11-05.txt  g  h  i  j  k  l  m  n  o  oldboy.txt  p  q  r  s  t  txt  u  v  w  x  y  z
```

- 感叹号的vim强制退出
- 找出历史命令