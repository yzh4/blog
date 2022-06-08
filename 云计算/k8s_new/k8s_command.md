> 参考：https://www.cnblogs.com/zsmooth/p/13673084.html

查看版本信息

```
kubectl version
```

查看命名空间

```
kubectl get ns
```

查看所有pods

```
kubectl get pods -A
```

查看某个命名空间下的pod

```
kubectl get pods 
```

简介

master机器服务：

- api Server
  	- 提供集群管理的REST API接口，包括认证授权、数据校验以及集群状态变更等
  	- 只有API Server才直接操作etcd
  	- 其他模块通过API Server查询或修改数据
  	- 提供其他模块之间的数据交互和通信的枢纽
- scheduler
  	- 负责分配调度Pod到集群内的node节点
  	- 监听kube-apiserver，查询还未分配Node的Pod
  	- 根据调度策略为这些Pod分配节点
- controller manager
  	- 由一系列的控制器组成，它通过API Server监控整个 集群的状态，并确保集群处于预期的工作状态
- kubelet
- kube-proxy
- docker

Node机器服务：
- docker

- kubelet

- kube-proxy

  常用命令

  查看Kubernates的版本

  ```
  kubectl version
  ```

  查看istio-system镜像

  ```
  kubectl get pod -n istio-system
  ```

  查看k8基础镜像(kube-system)

  ```
  kubectl get pod -n kube-system
  ```

  修改node为准备状态

  ```
  kubectl uncordon (IP地址)
  ```

  修改node为不可调度状态

  ```
  kubectl cordon (IP地址)
  ```

  将某node机器上的pod平滑的赶到其它节点上

  ```
  kubectl drain (IP地址)
  ```

  临时删除node节点

  ```
  kubectl delete nodes (IP地址)
  ```

  卸载node节点

  ```
  easzctl del-node (IP地址)
  ```

  删除node

  ```
  kubectl delete node (IP地址)
  ```

  新加node节点机器

  ```
  easzctl add-node 机器ip
  ```

  新加master节点机器

  ```
  easzctl add-master 机器ip
  ```

  查看运行实例日志

  ```
  kubectl logs -f -n cloud（容器实例）-c（容器名称）
  ```

  查看容器运行状态

  ```
  kubectl get pod -n cloud 
  ```

  显示Pod的更多信息

  ```
  kubectl get pod -n cloud -o wide
  ```

  查看运行实例pod描述

  ```
  kubectl describe pod -n cloud （容器实例）
  ```

  取日志并转为txt格式

  ```
  kubectl logs osg-openpf0001-group-1-v2-6d4db69f5c-dhppv >1.txt
  例：kubectl describe pod -n cloud newstu-75cf556fdf-pcq2j
  ```

  推送本地镜像到镜像仓库

  ```
  docker push 镜像IP/cloud/镜像名称：版本号
  例：docker push 192.168.111.18/cloud/auth-center:v1
  ```

  

保存镜像

```
docker save （镜像名称）:（版本） -o （保存镜像的名称）
例：docker save 222.30.194.226/cloud/app-manager:v1 -o app-manager.tar
```

删除lod文件

```
kubectl delete -f kubectl istio-svc-dpt.yaml
```

添加lod文件

```
kubectl apply -f kubectl istio-svc-dpt.yaml
```

列出所有容器中的服务器节点ip

```
kubectl get services
```

列出node节点

```
kubectl get nodes
```

获取自动部署实例列表

```
kubectl get deployments -n cloud     （获取所有实例信息）
例：kubectl get deployments redis -n cloud    （获取指定redis实例）
```


删除pod自动部署实例信息

```
kubectl delete  deployments -n cloud     （删除所有pod实例，慎用）
kubectl delete  deployments redis -n cloud      （删除指定redis实例）
```


获取pod service信息

```
kubectl get services -n cloud
```

修改镜像名

```
docker tag 原镜像名 新镜像名
```

删除镜像实例

```
kubectl delete deployments -n cloud（容器名称）
例：kubectl delete deployments -n cloud salary
```


强制删除并自动重启pod

```
kubectl delete pod -n cloud （pod名称）
例：kubectl delete pod -n cloud newstu-75cf556fdf-pcq2j
```


进入容器内部

```
kubectl exec -it -n cloud（容器实例）-c （容器名称）/bin/sh
例：kubectl exec -it -n cloud cas-686757cfcd-dxtxr -c cas /bin/sh
```


从pod里面拷贝文件到本地

```
kubectl cp -n cloud (pod实例):(要拷贝的文件) -c (pod名称) (拷贝的位置)
例：kubectl cp -n cloud pitcher-dash-5746cff75-429qd:app.jar -c pitcher-dash /opt
```


添加标签

```
kubectl label nodes(节点) 192.168.0.74(所选节点) name=office
```

查看node节点标签

```
kubectl get nodes --show-labels
```

删除node节点标签

```
kubectl label nodes (node节点IP) 标签-
例: kubectl label nodes 192.168.111.112 name-
```


查看pod cpu、内存占用情况

```
kubectl top pod --all-namespaces
```

清理僵尸pod

```
kubectl delete pod -n cloud --force --grace-period=0  podname
```

查看pod使用内存

```
kubectl top pod -n cloud
```

docker批量删除无tag标签的无用镜像

```
docker images|grep none|awk '{print $3}'|xargs docker rmi
```

问题处理

```
问题处理
如pod无法下载出现以下问题

这个问题大概意思是说http返回值503无法探测此pod在node节点的状态

原因是因为创建node节点或者是新添加node节点是创建了多余的网卡 mynet0

ip link set mynet0 down

ip link delete mynet0

service kubelet restart
如没有这个网卡重启网络插件即可flannel （多重启几次）

如网关文件无法下载

k8s-2

这个问题是网络插件导致重启网络插件flannel （多重启几次）
```

