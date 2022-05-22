## 1. 介绍

​    和Apache等web服务软件一样，Nginx Rewrite的主要功能也是实现URL地址重写，Nginx的 Rewrite规则需要PCRE软件的支持，即通过Perl兼容正则表达式语法进行规则匹配的，前文在安装Nginx软件时就已经安装了这个PCRE软件，以及让Nginx支持Rewrite的功能，默认参数编译 Nginx就会安装支持Rewrite的模块，但是，也必须要PCRE软件的支持。

## 2. 语法

指令语法：rewrite regex replacement [flag];

默认值:none

应用位置：server、location、if

​    rewrite是实现URL重写的关键指令，根据regex（正则表达式）部分内容，重定向到replacement部分内容，结尾是flag标记。下面是一个简单的URL Rewrite跳转的例子：

```
rewrite ^/(.*) http://www.caimengzhi.org/$1 permanent;
```

> 解释

```
在上述指令中，rewrite为固定关键字，表示开启一条rewrite匹配规则，regex部分是^/(.*),这是一个正则表达式，匹配所有，匹配成功后跳转到http://www.caimengzhi.org/$1,这里
的$1是取前面regex部分()里的内容，结尾permanent;表示永久301重定向标记，即跳转到后面的
http://www.caimengzhi.org/$1地址上。
```

### 2.1 例子1

```
 server {
    listen       80;
    server_name  www.abc.org;
    location / {
         rewrite ^/(.*) https://www.caimengzhi.cn/$1 permanent;
    }
    access_log  logs/www_access.log main;
}
```

> 解释

### 2.2 例子2

```
 server {
    listen       80;
    server_name  www.abc.org;
    location / {
              root html/www;
              index index.html;
    }
    location ^~ /images/ {
         rewrite ^/(.*) https://www.caimengzhi.cn/$1 permanent;
    }
    access_log  logs/www_access.log main;
}
```

> 解释

## 3. Rewrite regex常用正则表达式字符

| 字符      | 描述                                                         |
| :-------- | :----------------------------------------------------------- |
| \         | 将后面接着的字符标记为一个特殊的字符或者原义字符或者一个向后的引用，例如"\n"匹配一个换行符号，序列`"\\"`和`"\$"`则匹配`"$"` |
| ^         | 匹配输入字符串的起始位置，如果设置regexp对象的mutiline属性。也匹配`"\n"`或者`"r"`之后的位置 |
| $         | 匹配输入字符串的结束位置，如果设置了regexp对象mutiline属性，`$`也匹配`"\n"`或者`"\r"`之前位置 |
| *         | 匹配前面的字符零次或者多次，例如`cai\*.`能匹配ca或者cai或者caii等,*等价于{0,} |
| +         | 匹配前面的字符一次或者多次，例如cai+，能匹配cai或者caii，但是不能匹配ca，+等价于{1，} |
| ?         | 匹配前面的字符零次或者一次，例如cai?可匹配cai或者ca，?等价于{0,1}，当该字符紧跟着任何一个其他的限制符[*,+,?,{n},{m,n}]后面时，匹配模式是非贪婪模式。非贪婪模式尽可能少匹配所搜索的字符串，而默认的贪婪匹配模式则是尽可能的匹配所搜索的字符串，例如,对于字符串`"caiiii"`,`"cai+?”`，将匹配单个i，而i+将匹配所有i |
| .         | 匹配除`"\n"`之外的任何单个字符，要匹配包括`"\n"`在内的任何字符，请使用`"[.\n]"`的模式 |
| (pattern) | 匹配括号内pattern并可以在后面获取对于的匹配，常常用\(0...\)9属性获取小括号中的匹配内容，要匹配圆括号内字符，请使用`"\("`或`"\)"` |

## 4. rewrite指令结尾flag标记说明

rewrite指令最后一项参数为flag标记，该标记说明见下表。

| flag标记符号 | 说明                                                   |
| :----------- | :----------------------------------------------------- |
| last         | 本条规则匹配完成后，继续向下匹配新的location URI规则。 |
| break        | 本条规则匹配完成后即终止，不再匹配后面的任何规则。     |
| redirect     | 返回302临时重定向，浏览器地址栏会显示跳转后的URL地址。 |
| permanent    | 返回301永久重定向，浏览器地址栏会显示跳转后的URL地址。 |

​    在以上的flag标记中，last和break用来实现URL重写，浏览器地址栏URL地址不变。 但在服务器端访问的程序及路径发生了变化。redirect和permanent用来实现URL跳转， 浏览器地址栏会显示跳转后的URL地址。

​    last和break标记的实现功能类似，但二者之间有细微的差别，使用alias指令时必须 用last标记，使用proxy_pass指令时要使用break标记。last标记在本条rewrite规则 执行完毕后，会对其所在的server{……}标签重新发起请求，而break标记则在本条规则匹配 完成后，终止匹配，不在匹配后面的规则。

## 6. Nginx Rewrite的企业应用场景

Nginx的Rewrite功能在企业里应用非常广泛：

- 可以调整用户浏览的URL，看起来更规范，合乎开房及产品人员的需求。
- 为了让搜索引擎收录网站内容及用户体验更好，企业会将动态URL地址伪装成静态地址提供服务
- 网站换新域名后，让旧的域名的访问跳转到新的域名上，例如：京东的360buy.com换成jd.com
- 根据特殊变量、目录、客户端的信息进行URL跳转等。

## 5. 企业实战301

​    公司换新域名，需要做301永久重定向，老域名为： [www.360buy.com](http://www.360buy.com/) 新域名为：[www.jd.com](http://www.jd.com/) 配置信息如下：

```
server {
    listen       80;
    server_name  www.360buy.com;
    rewrite ^/(.*) http://www.jd.com/$1 permanent;
}
server {
    listen      80;
    server_name  www.jd.com jd.com;
    location / {
        root html/www;
        index index.php index.html;
    }
}
```

当用户访问 [www.360buy.com的时候](http://www.360buy.xn--com-5s0em31hr0u/)，就自动跳转到了https://[www.jd.com上](http://www.jd.xn--com-x28d/)，实现了301永久重定向。