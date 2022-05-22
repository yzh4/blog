## linux文件属性与管理

文件或目录属性主要包括：

- 索引节点，inode
- 文件类型
- 文件权限
- 硬链接个数
- 归属的用户和用户组
- 最新修改时间

查看命令

```
[root@sql tmp]# ls -lhi
total 4.0K
17866214 -rw-r--r-- 1 root root 1.6K Feb 11 20:22 aaa.txt
```

1. Inode索引节点号，（人的身份证，家庭地址等信息，唯一），系统寻找文件名 > Inode号 > 文件内容
2. 文件权限，第一个字符是文件类型，随后9个字符是文件权限，最后一个. 有关selinux
3. 文件硬链接数量，与ln命令配合
4. 文件所属用户
5. 文件所属用户组
6. 文件目录大小
7. 文件修改时间
8. 文件名

## 文件扩展名

Linux文件的扩展名只是方便阅读，对文件类型不影响

Linux通过文件属性区分文件类型

- .txt文本类型
- .conf .cfg .configure 配置文件
- .sh .bash 脚本后缀
- .py 脚本后缀
- .rpm 红帽系统二进制软件包名
- .tar .gz .zip 压缩后缀

## 文件类型

> 可以通过ls -F 给文件结尾加上特殊标识

| 格式              | 类型                                                |
| ----------------- | --------------------------------------------------- |
| ls -l看第一个字符 |                                                     |
| -                 | 普通文件regular file，（二进制，图片，日志，txt等） |
| d                 | 文件夹directory                                     |
| b                 | 块设备文件，/dev/sda1，硬盘，光驱                   |
| c                 | 设备文件，终端/dev/tty1,网络串口文件                |
| s                 | 套接字文件，进程间通信（socket）文件                |
| p                 | 管道文件pipe                                        |
| l                 | 链接文件,link类型，快捷方式                         |

**普通文件**

通过如下命令生成都是普通文件(windows中各种扩展名的文件，放入linux也是普通文件类型)

- echo
- touch
- cp
- cat
- 重定向符号 >

普通文件特征就是文件类型是，"-"开头，以内容区分一般分为

- 纯文本，可以用cat命令读取内容，如字符、数字、特殊符号等
- 二进制文件（binary），Linux中命令属于这种格式，例如ls、cat等命令

**文件夹**

文件权限开头带有d字符的文件表示文件夹，是一种特殊的Linux文件

- mkdir
- cp拷贝文件夹

**链接类型**

- ln命令创建

类似windows的快捷方式

## file

> 显示文件的类型

```
[root@docker01 tmp]# file /usr/bin/python2.7        #二进制解释器类型
/usr/bin/python2.7: ELF 64-bit LSB executable

[root@docker01 tmp]# file /usr/bin/yum                    #yum是python的脚本文件
/usr/bin/yum: Python script, ASCII text executable

[root@docker01 tmp]# file /usr/bin/cd                #shell脚本，内置命令
/usr/bin/cd: POSIX shell script, ASCII text executable

[root@docker01 tmp]# file hehe.txt            #text类型
hehe.txt: ASCII text

[root@docker01 tmp]# file heihei            #文件夹
heihei: directory

[root@docker01 tmp]# file /usr/bin/python2            #软链接类型
/usr/bin/python2: symbolic link to `python2.7'
```

## which

> 查找PATH环境变量中的文件，linux内置命令不在path中

```
[root@docker01 tmp]# which python
/usr/bin/python
```

## whereis

> whereis命令用来定位指令的二进制程序、源代码文件和man手册页等相关文件的路径。

```
[root@docker01 tmp]# whereis python
python: /usr/bin/python /usr/bin/python2.7 /usr/lib/python2.7 /usr/lib64/python2.7 /etc/python /usr/include/python2.7 /usr/share/man/man1/python.1.gz
```

## tar

> tar命令在linux系统里，可以实现对多个文件进行，压缩、打包、解包

**打包**：将一大堆文件或目录汇总成一个整体。

**压缩**：将大文件压缩成小文件，节省磁盘空间。

> 语法

```
tar(选项)(参数)

-A或--catenate：新增文件到以存在的备份文件；
-B：设置区块大小；
-c或--create：建立新的备份文件；
-C <目录>：这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项。
-d：记录文件的差别；
-x或--extract或--get：从备份文件中还原文件；
-t或--list：列出备份文件的内容；
-z或--gzip或--ungzip：通过gzip指令处理备份文件；
-Z或--compress或--uncompress：通过compress指令处理备份文件；
-f<备份文件>或--file=<备份文件>：指定备份文件；
-v或--verbose：显示指令执行过程；
-r：添加文件到已经压缩的文件；
-u：添加改变了和现有的文件到已经存在的压缩文件；
-j：支持bzip2解压文件；
-v：显示操作过程；
-l：文件系统边界设置；
-k：保留原有文件不覆盖；
-m：保留文件不被覆盖；
-w：确认压缩文件的正确性；
-p或--same-permissions：用原来的文件权限还原文件；
-P或--absolute-names：文件名使用绝对名称，不移除文件名称前的“/”号；不建议使用
-N <日期格式> 或 --newer=<日期时间>：只将较指定日期更新的文件保存到备份文件里；
--exclude=<范本样式>：排除符合范本样式的文件。
-h, --dereference跟踪符号链接；将它们所指向的文件归档并输出
```

> 案例

仅打包，不压缩

```
#tar 参数 包裹文件名  需要打包的文件
[alex@docker01 tmp]$ tar -cvf alltmp.tar ./*
```

打包后且用gzip命令压缩，节省磁盘空间

```
[alex@docker01 tmp]$ tar -zcvf alltmp.tar ./*
```

**注意**

- f参数必须写在最后，后面紧跟压缩文件名
- tar命令仅打包，习惯用.tar作为后缀
- tar命令加上z参数，文件以.tar.gz或.tgz表示

列出tar包内的文件

```
#根据tar包文件后缀，决定是否添加z参数，调用gzip
[alex@docker01 tmp]$ tar -ztvf alltmp2.tar.gz
```

拆开tar包

```
[root@docker01 tmp]# tar -xf alltmp.tar
```

拆开tar的压缩包

```
tar -zxvf ../alltmp2.tar.gz ./
```

拆除tar包中部分文件

```
#正常解压命令，单独加上你要拆除的文件名，指定路径
#先看下tar包中有什么内容，再指定文件解压

[root@docker01 tmp]# tar -ztvf ../alltmp2.tar.gz

[root@docker01 tmp]# tar -zxvf ../alltmp2.tar.gz ./alltmp.tar
./alltmp.tar
```

指定目录解tar包

```
[root@docker01 tmp]# tar -xf alltmp.tar -C /opt/data/
```

排除文件解包

```
#注意--exclude 跟着文件名或是文件夹，不得加斜杠，排除多个文件，就写多个--exclude
[root@docker01 tmp]# tar -zxvf ../alltmp2.tar.gz   --exclude data
```

打包链接文件

```
-h参数能够保证，打包的不仅仅是个快捷方式，而是找到源文件
```

打包/etc下所有普通文件

```
[root@docker01 tmp]# tar -zcvf etc.tgz `find /etc -type f`
[root@docker01 tmp]# tar -tzvf etc.tgz
```

## gzip

要说tar命令是个纸箱子用于打包，gzip命令就是压缩机器

gzip通过压缩算法*lempel-ziv* 算法(*lz77*) 将文件压缩为较小文件，节省60%以上的存储空间，以及网络传输速率

```
gzip(选项)(参数)

-a或——ascii：使用ASCII文字模式；
-c或--stdout或--to-stdout 　把解压后的文件输出到标准输出设备。 
-d或--decompress或----uncompress：解开压缩文件；
-f或——force：强行压缩文件。不理会文件名称或硬连接是否存在以及该文件是否为符号连接；
-h或——help：在线帮助；
-l或——list：列出压缩文件的相关信息；
-L或——license：显示版本与版权信息；
-n或--no-name：压缩文件时，不保存原来的文件名称及时间戳记；
-N或——name：压缩文件时，保存原来的文件名称及时间戳记；
-q或——quiet：不显示警告信息；
-r或——recursive：递归处理，将指定目录下的所有文件及子目录一并处理；
-S或<压缩字尾字符串>或----suffix<压缩字尾字符串>：更改压缩字尾字符串；
-t或——test：测试压缩文件是否正确无误；
-v或——verbose：显示指令执行过程；
-V或——version：显示版本信息；
-<压缩效率>：压缩效率是一个介于1~9的数值，预设值为“6”，指定愈大的数值，压缩效率就会愈高；
--best：此参数的效果和指定“-9”参数相同；
--fast：此参数的效果和指定“-1”参数相同。
```

案例

```
#压缩目录中每一个html文件为.gz,文件夹无法压缩，必须先tar打包
gzip *.html        #gzip压缩，解压都会删除源文件
```

列出压缩文件中信息

```
[root@docker01 tmp]# gzip -l *.gz        #不解压显示压缩文件内信息，以及压缩率
         compressed        uncompressed  ratio uncompressed_name
                 28                   0   0.0% 10.html
                 24                   0   0.0% 123
                 27                   0   0.0% 1.html
                 27                   0   0.0% 2.html
                 27                   0   0.0% 3.html
                 27                   0   0.0% 4.html
                 27                   0   0.0% 5.html
                 27                   0   0.0% 6.html
                 27                   0   0.0% 7.html
                 27                   0   0.0% 8.html
                 27                   0   0.0% 9.html
           23581672           118888884  80.2% alex.txt
           23582535           118896640  80.2% alltmp.tar
                289                 470  44.9% glances.log
                 45                  16 -12.5% hehe.txt
           47164836           237786010  80.2% (totals)
```

解压缩且显示过程

```
[root@docker01 tmp]# gzip -dv *.gz
10.html.gz:      0.0% -- replaced with 10.html
123.gz:      0.0% -- replaced with 123
1.html.gz:      0.0% -- replaced with 1.html
2.html.gz:      0.0% -- replaced with 2.html
3.html.gz:      0.0% -- replaced with 3.html
4.html.gz:      0.0% -- replaced with 4.html
5.html.gz:      0.0% -- replaced with 5.html
6.html.gz:      0.0% -- replaced with 6.html
7.html.gz:      0.0% -- replaced with 7.html
8.html.gz:      0.0% -- replaced with 8.html
9.html.gz:      0.0% -- replaced with 9.html
alex.txt.gz:     80.2% -- replaced with alex.txt
alltmp.tar.gz:     80.2% -- replaced with alltmp.tar
glances.log.gz:     44.9% -- replaced with glances.log
hehe.txt.gz:    -12.5% -- replaced with hehe.txt
```

压缩保留源文件

```
#-c参数
[root@docker01 tmp]# gzip -c  alltmp.tar > alltmp.tar.gz
```

**gzip套件提供了许多方便的工具命令，可以直接操作压缩文件内容**

- zcat，直接读取压缩文件内容`zcat hehe.txt.gz`
- zgrep
- zless
- zdiff

## xz

xz 是一个使用 LZMA压缩算法的无损数据压缩文件格式。和gzip与bzip2一样，同样支持多文件压缩，但是约定不能将多于一个的目标文件压缩进同一个档案文件。相反，xz通常作为一种归档文件自身的压缩格式，例如使用tar或cpioUnix程序创建的归档。xz 在GNU coreutils（版本 7.1 或更新）中被使用。xz 作为压缩软件包被收录在 Fedora (自Fedora 12起), Arch Linux, FreeBSD、 Slackware Linux、CRUX 和 Funtoo中。

**xz参数**

```
-z，——compress压缩力
- d,解压,解压
                      力减压
-t，——test测试压缩文件的完整性
-l，——list列出。xz文件的信息
-k，——keep保留(不删除)输入文件
-f，——force强制覆盖输出文件和(de)压缩链接
- c, stdout,——到stdout
写入标准输出，不要删除输入文件
0…9           压缩预设;默认是6;把压缩机* *
在使用7-9之前，要考虑解压器内存使用情况!
-e，——极端尝试通过使用更多的CPU时间来提高压缩比;
不会影响解压缩器的内存需求
-T，——threads=NUM最多使用NUM线程;默认值为1;设置为0
使用尽可能多的线程，因为有处理器核心
-q，——安静地抑制警告;指定两次也可以抑制错误
-v，——verbose verbose;指定两次会更详细
-h，——help显示此简短帮助并退出
-H，——long-help显示长帮助(同时列出高级选项)
-V，——version显示版本号并退出
```

**解压 xz 格式文件**
方法一：
需要用到两步命令，首先利用 xz-utils 的 xz 命令将 linux-3.12.tar.xz 解压为 linux-3.12.tar，其次用 tar 命令将 linux-3.12.tar 完全解压。

```
xz -d linux-3.12.tar.xz
tar -xf linux-3.12.tar
```

方法二（推荐）

```
tar -Jxf linux-3.12.tar.xz
```

**创建 xz 格式文件**
方法一：
也是用到两步命令，首先利用 tar 命令将 linux-3.12 文件夹打包成 linux-3.12.tar，其次用 xz-utils 的 xz 命令将 linux-3.12.tar 压缩成 linux-3.12.tar.xz。

```
tar -cf linux-3.12.tar linux-3.12/
xz -z linux-3.12.tar
```

方法二（推荐）

```
tar -Jcf linux-3.12.tar.xz linux-3.12/
```

## zip

zip 命令：是一个应用广泛的跨平台的压缩工具，压缩文件的后缀为 zip文件，还可以压缩文件夹

```
语法：
zip 压缩文件名  要压缩的内容

-A 自动解压文件
-c 给压缩文件加注释
-d 删除文件
-F 修复损坏文件
-k 兼容 DOS
-m 压缩完毕后，删除源文件
-q 运行时不显示信息处理信息
-r 处理指定目录和指定目录下的使用子目录
-v 显示信息的处理信息
-x “文件列表” 压缩时排除文件列表中指定的文件
-y 保留符号链接
-b<目录> 指定压缩到的目录
-i<格式> 匹配格式进行压缩
-L 显示版权信息
-t<日期> 指定压缩文件的日期
-<压缩率> 指定压缩率
最后更新 2018-03-08 19:33:4
```

案例

```
#压缩当前目录下所有内容为alltmp.zip文件
[root@docker01 tmp]# zip alltmp.zip ./*

#压缩多个文件夹
[root@docker01 tmp]# zip -r data.zip ./data ./data2
```

## unzip

解压



参数

```
-l：显示压缩文件内所包含的文件；
-d<目录> 指定文件解压缩后所要存储的目录。
```

案例

```
#查看压缩文件内容
[root@docker01 tmp]# unzip -l data.zip

#解压缩zip文件
[root@docker01 tmp]# unzip data.zip
```

## date

date命令用于显示当前系统时间，或者修改系统时间

```
语法

date  参数   时间格式
```

参数

```
-d, --date=STRING
    显示由 STRING 指定的时间, 而不是当前时间 

-s, --set=STRING
    根据 STRING 设置时间 

-u, --utc, --universal
    显示或设置全球时间(格林威治时间)
```

时间格式

```
%%
    文本的 % 
%a
    当前区域的星期几的简写 (Sun..Sat) 
%A
    当前区域的星期几的全称 (不同长度) (Sunday..Saturday) 
%b
    当前区域的月份的简写 (Jan..Dec) 
%B
    当前区域的月份的全称(变长) (January..December) 
%c
    当前区域的日期和时间 (Sat Nov 04 12:02:33 EST 1989) 
%d
    (月份中的)几号(用两位表示) (01..31) 
%D
    日期(按照 月/日期/年 格式显示) (mm/dd/yy) 
%e
    (月份中的)几号(去零表示) ( 1..31) 
%h
    同 %b 
%H
    小时(按 24 小时制显示，用两位表示) (00..23) 
%I
    小时(按 12 小时制显示，用两位表示) (01..12) 
%j
    (一年中的)第几天(用三位表示) (001..366) 
%k
    小时(按 24 小时制显示，去零显示) ( 0..23) 
%l
    小时(按 12 小时制显示，去零表示) ( 1..12) 
%m
    月份(用两位表示) (01..12) 
%M
    分钟数(用两位表示) (00..59) 
%n
    换行 
%p
    当前时间是上午 AM 还是下午 PM 
%r
    时间,按 12 小时制显示 (hh:mm:ss [A/P]M) 
%s
    从 1970年1月1日0点0分0秒到现在历经的秒数 (GNU扩充) 
%S
    秒数(用两位表示)(00..60) 
%t
    水平方向的 tab 制表符 
%T
    时间,按 24 小时制显示(hh:mm:ss) 
%U
    (一年中的)第几个星期，以星期天作为一周的开始(用两位表示) (00..53) 
%V
    (一年中的)第几个星期，以星期一作为一周的开始(用两位表示) (01..52) 
%w
    用数字表示星期几 (0..6); 0 代表星期天 
%W
    (一年中的)第几个星期，以星期一作为一周的开始(用两位表示) (00..53) 
%x
    按照 (mm/dd/yy) 格式显示当前日期 
%X
    按照 (%H:%M:%S) 格式显示当前时间 
%y
    年的后两位数字 (00..99) 
%Y
    年(用 4 位表示) (1970...) 
%z
    按照 RFC-822 中指定的数字时区显示(如, -0500) (为非标准扩充) 
%Z
    时区(例如, EDT (美国东部时区)), 如果不能决定是哪个时区则为空 

默认情况下,用 0 填充数据的空缺部分. GNU 的 date 命令能分辨在 `%'和数字指示之间的以下修改.

    `-' (连接号) 不进行填充 `_' (下划线) 用空格进行填充
```

案例

显示当前系统部分时间

```
1.显示短年份
date +%y

2.显示长年份
date +%Y

3.显示月份
date +%m

4.显示几号
date +%d

5.显示几时
date +%H

6.显示几分
date +%M

7.显示整秒
date +%S

8.显示时间如，年-月-日
date +%F

9.显示时间如，时：分：秒
date +%T
```

-d参数指定时间显示，仅仅是显示

```
1.显示昨天
 date +%F -d "-1day"

2.显示昨天
date +%F -d "yesterday"

3.显示前天
date +%F -d "-2day"

4.显示明天日期
date +%F -d "+1day"

5.显示明天，英文表示
date +%F -d "tomorrow"

6.显示一个月之前，之后
[root@pylinux /]# date +%F -d "1month"
2019-12-01
[root@pylinux /]# date +%F -d "-1month"
2019-10-01

7.显示一年后
date +%F -d "1year"

8.显示60分钟后
date +%T -d "60min"


+表示未来
-表示过去
day表示日
month表示月份
year表示年
min表示分钟
```

-s设置时间

```
设置时间较少，一般配置ntp时间服务器

1.设置时间
[root@pylinux /]# date -s "20170808"
2017年 08月 08日 星期二 00:00:00 CST
[root@pylinux /]#
[root@pylinux /]# date
2017年 08月 08日 星期二 00:00:00 CST



2.修改分钟
[root@pylinux /]# date -s "05:06:33"
2017年 08月 08日 星期二 05:06:33 CST
[root@pylinux /]# date
2017年 08月 08日 星期二 05:06:33 CST


3.修改日期和分钟
[root@pylinux /]# date -s "20180606 05:30:30"
2018年 06月 06日 星期三 05:30:30 CST
[root@pylinux /]# date
2018年 06月 06日 星期三 05:30:31 CST

4.可设置不同格式的时间
date -s "2018-06-06 05:30:30"
date -s "2018/07/07 05:30:30"
```

## shred

> 文件粉碎工具

```
用法：shred [选项]... 文件...

多次覆盖文件，使得即使是昂贵的硬件探测仪器也难以将数据复原。

-u, --remove 覆盖后截断并删除文件
shred heihei.txt  随机覆盖文件内容，不删除源文件
```

案例

彻底粉碎且删除文件

```
[root@pylinux tmp]# ls -lh
总用量 25M
-rw-r--r-- 1 root root 25M 10月 14 15:02 heihei.txt
[root@pylinux tmp]#
[root@pylinux tmp]# shred -u heihei.txt
```