# 举个例子

**Python离线安装python3，pip3和离线安装、迁移第三方模块**

### 离线安装

```
yum install --downloadonly --downloaddir=/tmp/files \
zlib-devel bzip2-devel openssl-devel ncurses-devel  epel-release gcc gcc-c++ xz-devel readline-devel gdbm-devel sqlite-devel tk-devel db4-devel libpcap-devel libffi-devel

#将依赖包copy到离线服务器，进入目录
cd /home/files  

#安装所有rpm依赖包
rpm -Uvh ./*.rpm --nodeps --force   
```

### 离线安装python3.7.2

```
tar -zxvf Python-3.7.2.tgz	# 解压python3安装包
cd Python-3.7.2 	# 进入python3安装包目录
./configure --prefix=/usr/local/bin/python3	# 将python3安装在这个目录
make && make install	# 编译和安装
```

### 创建软连接

```
ln -s /usr/local/bin/python3/bin/python3 /usr/bin/python3	# 创建python3软连接
ln -s /usr/local/bin/python3/bin/pip3 /usr/bin/pip3	# 创建pip3的软连接
```

## 离线安python装第三方模块

在联网的centos中下载模块

```
# 如果新安装某一个模块，可以直接下载下来
pip3 download xxx  -d /tmp/packages/
```

```
# 默认情况download是最新版本模块，但有时候我们需要下载指定模块版本，比如下面我们下载paramiko的2.4.2版本。

pip3 download paramiko==2.4.2  -d /tmp/packages/
```

有网络的服务器查看服务器已安装的模块，下载并安装到离线服务器（迁移模块）

```
# pip3 list可以查看已安装的模块
[root@localhost py_model]# pip3 list
```

```
# 将pip3 list的信息生成文档
pip3 freeze >requirements.txt
```

```
# 将requirement.txt文档中列出的模块信息下载到指定目录
pip3 download -r requirements.txt -d /tmp/packages/  #推荐使用

或pip3 install --download /tmp/packages -r requirements.txt
```

```
# 将下载好的模块copy到离线服务器
pip3 install xxx.tar.gz
pip3 install xxx.whl
pip3 install xxx.xx  #是什么格式就安装什么格式的文件即可。

# 如果有要安装的包和依赖包有多个，且不知道先装哪个，那么就把这些文件放在一个目录中，然后进入该目录使用下面命令一起安装

pip3 install ./*
```

```
# 批量离线安装requirments.txt中的模块，需要将下载好的模块和requirments.txt都copy到一个目录，然后执行下面的命令
pip3 install --no-index --find-links=/tmp/packages  -r requirments.txt 
```

