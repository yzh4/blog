#### 开发Shell脚本判断系统根分区剩余空间的大小，如果低于1000MB就提示不足，否则提示充足。

```shell
 #!/bin/bash
 #Author:yzh
 #xxx
 #Time:2022-02-03 19:55:58
 #Name:bb.sh
 #Version:V1.0
 #Description:This is a production script.

 
 mem=`df -m | awk 'NR==2{print $4}'`                                                                
 
 if [ $mem -lt 1000 ];then
     echo "内存不足"
 else
     echo "内存充足"
 fi
```

#### 分别使用变量定义、read读入及脚本传参方式实现比较2个整数的大小。

```shell
 #!/bin/bash
 #Author:yzh
 #xxx
 #Time:2022-02-03 19:55:58
 #Name:bb.sh
 #Version:V1.0
 #Description:This is a production script.

read -p "请输入两个数:" num1 num2
a=$num1
b=$num2
if [ -z "$b" ];then
    echo "缺少一个参数，请输入两个整数"
    exit 1
fi

expr 100 + $a + $b &>/dev/null
if [ $? -ne 0 ];then
    echo "请输入两个整数"
    exit 2
fi

if [ $a -eq $b ];then
    echo "两个数相等"
elif [ $a -gt $b ];then
    echo "第一个数大于第二个数"
else
    echo "第二个数大于第一个数"
fi

```

#### 打印一个菜单如下，当用户选择对应的数字时，就执行对应项的应用，最好对用户的输入进行是否为整数判断。

  1.install lamp
  2.install lnmp
  3.exit

```shell
 #!/bin/bash
 #Author:yzh
 #xxx
 #Time:2022-02-03 19:55:58
 #Name:bb.sh
 #Version:V1.0
 #Description:This is a production script.

cat <<EOF
	1.install lamp
	2.install lnmp
	3.exit
EOF
read -p "请输入一个序号(必须是整数):" num
#整数判断
expr 2 + $num &>/dev/null
if [ $? -ne 0 ];then
	echo "usage:$0{1|2|3}"
	exit 1
fi
#判断执行处理
if [ $num -eq 1 ];then
	echo "install lamp"
elif [ $num -eq 2 ];then
	echo "install lnmp"
elif [ $num -eq 3 ];then
	echo "bye"
	exit
else
	echo "usage:$0{1|2|3}"
```

#### 通过脚本传参的方式，检查Web网站URL是否正常（要求主体使用函数）。

```shell
 #!/bin/bash
 #Author:yzh
 #xxx
 #Time:2022-02-03 19:55:58
 #Name:bb.sh
 #Version:V1.0
 #Description:This is a production script.

usege(){
	echo "usage:$0 url"
	exit
}
checkurl(){
	wget -q -o /dev/null -t 2 -T 5 $1
	if [ $? -eq 0 ];then
		echo "$1 is ok"
	else
		echo "$1 is fail"
	fi
}
main(){
	if [ $# -ne 1 ];then
		usage
	fi
	checkurl $1
}
main $*
```

#### 利用case语句开发Rsync服务启动停止脚本，并能通过chkconfig实现开机自启动。

```shell
 #!/bin/bash
 #Author:yzh
 #xxx
 #Time:2022-02-03 19:55:58
 #Name:bb.sh
 #Version:V1.0
 #Description:This is a production script.

chkconfig rpcbind on
chkconfig rsync on
start(){
	rsync --daemon
	if [ $? -eq 0 ];then
		echo 'rsync is start'
	else
		echo 'error..'
	fi
}
stop(){
	pkill rsync 
	if [ $? -eq 0 ];then
		echo "rsync is stop"
	else
		echo "error"
	fi
}
restart(){
	stop
	start
}
case $1 in 
start)
	start
	;;
stop)
	stop
	;;
restart)
	restart
	;;
*)
	echo "usage:start|stop|restart"
esac
```

#### 猜数字游戏。首先让系统随机生成一个数字，给这个数字定一个范围（1-60），让用户输入猜的数字，对输入进行判断，如果不符合要求，就给予高或低的提示，猜对后则给出猜对用的次数，请用while语句实现。

```shell
 #!/bin/bash
 #Author:yzh
 #xxx
 #Time:2022-02-03 19:55:58
 #Name:bb.sh
 #Version:V1.0
 #Description:This is a production script.

random="$((RANDOM%60))"
while true
do
((count++))
read -p "请猜数字:" num
if [ $num -gt $random  ];then
    echo "猜高了"
elif [ $num -eq $random ];then
    echo "猜对了,共猜了${count}次"
else
    echo "猜低了"
fi
done

```

#### 分析nginx访问日志(自备)，把日志中每行的访问字节数对应字段数字相加，计算出总的访问量。给出实现程序，请用while循环实现。

```shell
 #!/bin/bash
 #Author:yzh
 #xxx
 #Time:2022-02-03 19:55:58
 #Name:bb.sh
 #Version:V1.0
 #Description:This is a production script.

sum=0
awk '{print $10}' /tools/nginx/logs/access.log >/tmp/access.log
while read line
do
    ((sum+=line))
done</tmp/access.log
echo $sum

```

#### 计算从1加到100之和（要求用for和while，至少给出两种方法）

```shell
 #!/bin/bash
 #Author:yzh
 #xxx
 #Time:2022-02-03 19:55:58
 #Name:bb.sh
 #Version:V1.0
 #Description:This is a production script.

for n in {1..100}
do
    ((sum1+=n))
done
echo $sum1

echo "-------------"

while ((i<=100))
do
    ((sum+=i))
    ((i++))
done
echo $sum
```

#### 利用bash for循环打印下面这句话中字母数不大于6的单词（某企业面试真题）。I am oldboy teacher welcome to oldboy training class

```
 #!/bin/bash
 #Author:yzh
 #xxx
 #Time:2022-02-03 19:55:58
 #Name:bb.sh
 #Version:V1.0
 #Description:This is a production script.

 a="I am oldboy teacher welcome to oldboy training class"
 b=`echo $a|tr " " "\n"`
 
 for i in $b
 do  
     if [[ `echo $i|wc -L` < 6  ]];then
         echo $i
     fi
 done

```

#### 使用read读入方式比较两个整数大小，要求对用户输入的内容严格判断是否为整数，是否输入了两个数字。

```
 #!/bin/bash
 #Author:yzh
 #xxx
 #Time:2022-02-03 19:55:58
 #Name:bb.sh
 #Version:V1.0
 #Description:This is a production script.

read -p "fist:" a
read -p "second" b

if [[ -z $a || -z $b  ]];then
    echo "请输入数字"
    exit 1
fi

if [[ -n "$(echo $a| sed -n "/^[0-9]\+$/p")" && -n "$(echo $b| sed -n "/^[0-9]\+$/p")" ]];then
    if [ $a -lt $b ];then
        echo "${b}>$a"
    elif [ $a -gt $b  ];then
        echo "${a}>$b"
    else
        echo "${a}=$b"
    fi
else
    echo "请不要输入非数字"
fi

```

#### 判断网段内主机存活案例ping分析及开发实现

```
#!/bin/bash

```

