###  节点规划

部署k8s集群的节点按照用途可以划分为如下2类角色：

- **master**：集群的master节点，集群的初始化节点，基础配置不低于2C4G
- **node**：集群的node节点，可以多台，基础配置不低于2C4G

**本例为了演示node节点的添加，会部署一台master+2台node**，节点规划如下：

| 主机名      | 节点ip        | 角色   | 部署组件                                                     |
| ----------- | ------------- | ------ | ------------------------------------------------------------ |
| k8s-master1 | 172.21.51.67  | master | etcd, kube-apiserver, kube-controller-manager, kubectl, kubeadm, kubelet, kube-proxy, flannel |
| k8s-master2 | 172.21.51.68  | master | etcd, kube-apiserver, kube-controller-manager, kubectl, kubeadm, kubelet, kube-proxy, flannel |
| k8s-master3 | 172.21.51.55  | master | etcd, kube-apiserver, kube-controller-manager, kubectl, kubeadm, kubelet, kube-proxy, flannel |
|             | 172.21.51.120 | VIP    | 作为3台master节点的LB使用                                    |
| k8s-node1   | 172.21.51.143 | node   | kubectl, kubelet, kube-proxy, flannel                        |
| k8s-node2   | 172.21.52.84  | node   | kubectl, kubelet, kube-proxy, flannel                        |

### 组件版本

| 组件       | 版本                              | 说明                                    |
| ---------- | --------------------------------- | --------------------------------------- |
| CentOS     | 7.8.2003                          |                                         |
| Kernel     | Linux 3.10.0-1127.10.1.el7.x86_64 |                                         |
| etcd       | 3.4.13-0                          | 使用Pod方式部署，默认数据挂载到本地路径 |
| coredns    | 1.8.0                             |                                         |
| kubeadm    | v1.21.5                           |                                         |
| kubectl    | v1.21.5                           |                                         |
| kubelet    | v1.21.5                           |                                         |
| kube-proxy | v1.21.5                           |                                         |
| flannel    | v0.11.0                           |                                         |

### 设置hosts解析

操作节点：所有节点（`k8s-master，k8s-node`）均需执行

- **修改hostname** hostname必须只能包含小写字母、数字、","、"-"，且开头结尾必须是小写字母或数字

```python
# 以其中一台master和node为例：

# 在k8s-master1节点
$ hostnamectl set-hostname k8s-master1 #设置master节点的hostname

# 在node-1节点
$ hostnamectl set-hostname k8s-node1 #设置node1节点的hostname
```

- **添加hosts解析**

```python
$ cat >>/etc/hosts<<EOF
172.21.51.67 k8s-master1
172.21.51.68 k8s-master2
172.21.51.50 k8s-master3
172.21.51.143 k8s-node1
172.21.52.84 k8s-node1
EOF
```

### 调整系统配置

操作节点： 所有的master和node节点（`k8s-master,k8s-node`）需要执行

> 本章下述操作均以k8s-master为例，其他节点均是相同的操作（ip和hostname的值换成对应机器的真实值）

- **设置安全组开放端口**

如果节点间无安全组限制（内网机器间可以任意访问），可以忽略，否则，至少保证如下端口可通： k8s-master节点：TCP：6443，2379，2380，60080，60081UDP协议端口全部打开 k8s-node节点：UDP协议端口全部打开

- **设置iptables**

```python
iptables -P FORWARD ACCEPT
```

- **关闭swap**

```python
swapoff -a && sysctl -w vm.swappiness=0
# 防止开机自动挂载 swap 分区
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

- **关闭selinux和防火墙**

```python
sed -ri 's#(SELINUX=).*#\1disabled#' /etc/selinux/config
setenforce 0
systemctl disable firewalld && systemctl stop firewalld
```

- **修改内核参数**

```python
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1
vm.max_map_count=262144
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
fs.may_detach_mounts = 1
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl =15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
EOF
modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
```

- 设置yum源

```bash
$ curl -o /etc/yum.repos.d/Centos-7.repo http://mirrors.aliyun.com/repo/Centos-7.repo
$ curl -o /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
$ yum clean all && yum makecache
$ yum install ipvsadm ipset sysstat conntrack libseccomp -y
```

###  安装docker

操作节点： 所有节点

```python
 ## 查看所有的可用版本
$ yum list docker-ce --showduplicates | sort -r
##安装旧版本 yum install docker-ce-cli-18.09.9-3.el7  docker-ce-18.09.9-3.el7
## 安装源里最新版本
$ yum install docker-ce-20.10.12 -y

## 配置docker加速
$ mkdir -p /etc/docker
vi /etc/docker/daemon.json
{
  "insecure-registries": [    
    "172.21.51.67:5000" 
  ],                          
  "registry-mirrors" : [
    "https://8xpk5wnt.mirror.aliyuncs.com"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
## 启动docker
$ systemctl enable docker && systemctl start docker
```

### 安装 kubeadm, kubelet 和 kubectl

操作节点： 所有的master和node节点(`k8s-master,k8s-node`) 需要执行

```bash
$ yum install -y kubelet-1.21.5 kubeadm-1.21.5 kubectl-1.21.5 --disableexcludes=kubernetes
## 查看kubeadm 版本
$ kubeadm version
## 设置kubelet开机启动
$ cat >/etc/sysconfig/kubelet<<EOF
KUBELET_EXTRA_ARGS="--cgroup-driver=systemd --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.2"
EOF
$ systemctl enable kubelet 
```

### 安装配置haproxy、keepalived

操作节点： 所有的master

```bash
$ yum install keepalived haproxy -y

# 所有master节点执行,注意替换最后的master节点IP地址
$ vi /etc/haproxy/haproxy.cfg
global
  maxconn  2000
  ulimit-n  16384
  log  127.0.0.1 local0 err
  stats timeout 30s
defaults
  log global
  mode  http
  option  httplog
  timeout connect 5000
  timeout client  50000
  timeout server  50000
  timeout http-request 15s
timeout http-keep-alive 15s
frontend monitor-in
  bind *:33305
  mode http
  option httplog
  monitor-uri /monitor
frontend k8s-master
  bind 0.0.0.0:7443
  bind 127.0.0.1:7443
  mode tcp
  option tcplog
  tcp-request inspect-delay 5s
  default_backend k8s-master
backend k8s-master
  mode tcp
  option tcplog
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server k8s-master1    172.21.51.67:6443  check
  server k8s-master2    172.21.51.68:6443   check
  server k8s-master3    172.21.51.55:6443   check 
  
  
# 在k8s-master1节点，注意mcast_src_ip换成实际的master1ip地址，virtual_ipaddress换成lb地址
$ vi /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2
    rise 1
}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    mcast_src_ip 172.21.51.67
    virtual_router_id 60
    priority 101
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        172.21.51.120
    }
    track_script {
       chk_apiserver
    }
}
# 在k8s-master2和k8s-master3分别创建/etc/keepalived/keepalived.conf，注意修改mcast_src_ip和virtual_ipaddress和state 改为BACKUP
# 在k8s-master2节点
$ cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2
rise 1
}
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    mcast_src_ip 172.21.51.68
    virtual_router_id 60
    priority 101
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        172.21.51.120
    }
    track_script {
       chk_apiserver
    }
}



#所有master节点配置KeepAlived健康检查文件：
$ cat /etc/keepalived/check_apiserver.sh
#!/bin/bash
err=0
for k in $(seq 1 3)
do
    check_code=$(pgrep haproxy)
    if [[ $check_code == "" ]]; then
        err=$(expr $err + 1)
        sleep 1
        continue
    else
        err=0
        break
    fi
done
if [[ $err != "0" ]]; then
    echo "systemctl stop keepalived"
    /usr/bin/systemctl stop keepalived
    exit 1
else
    exit 0
fi

# 启动haproxy和keepalived---->所有master节点
$ chmod +x /etc/keepalived/check_apiserver.sh
$ systemctl daemon-reload
$ systemctl enable --now haproxy
$ systemctl enable --now keepalived 


# 测试lbip是否生效
$ telnet 172.21.51.120 7443
```

###  初始化配置文件

操作节点： 只在master节点（`k8s-master`）执行

```yaml
$ cat kubeadm.yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: 7t2weq.bjbawausm0jaxury
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 172.21.51.67
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:
  - 172.21.51.120
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: 172.21.51.120:7443
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.21.5
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

> 对于上面的资源清单的文档比较杂，要想完整了解上面的资源对象对应的属性，可以查看对应的 godoc 文档，地址: https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2。

### 提前下载镜像

操作节点：所有master节点执行

```python
  # 查看需要使用的镜像列表,若无问题，将得到如下列表
$ kubeadm config images list --config kubeadm.yaml
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.21.5
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.21.5
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.21.5
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.21.5
registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.4.1
registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.13-0
registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.8.0
  # 提前下载镜像到本地
$ kubeadm config images pull --config kubeadm.yaml
```

### 初始化master节点

操作节点：只在master节点（`k8s-master`）执行

```python
$ kubeadm init --config kubeadm.yaml --upload-certs
```

若初始化成功后，最后会提示如下信息：

```python
...
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.21.51.120:7443 --token 7t2weq.bjbawausm0jaxury         --discovery-token-ca-cert-hash sha256:b0d875f1dafe9f479b23603c3424cad5e0e3aa0a47a8274f9d24432e97e3dbde         --control-plane --certificate-key 0ea981458813160b6fbc572d415e14cbc28c4bf958a765a7bc989b7ecc5dcdd6
```

接下来按照上述提示信息操作，配置kubectl客户端的认证

```python
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

> 若执行初始化过程中出错，根据错误信息调整后，执行 kubeadm reset -f ; ipvsadm --clear ; rm -rf ~/.kube

### 添加其他master节点到集群中

```bash
$ kubeadm join 172.21.51.120:7443 --token 7t2weq.bjbawausm0jaxury         --discovery-token-ca-cert-hash sha256:b0d875f1dafe9f479b23603c3424cad5e0e3aa0a47a8274f9d24432e97e3dbde         --control-plane --certificate-key 0ea981458813160b6fbc572d415e14cbc28c4bf958a765a7bc989b7ecc5dcdd6
```

### 添加node节点到集群中

操作节点：所有的node节点（`k8s-node`）需要执行 在每台node节点，执行如下命令，该命令是在kubeadm init成功后提示信息中打印出来的，需要替换成实际init后打印出的命令。

```python
kubeadm join 172.21.51.120:7443 --token 7t2weq.bjbawausm0jaxury         --discovery-token-ca-cert-hash sha256:b0d875f1dafe9f479b23603c3424cad5e0e3aa0a47a8274f9d24432e97e3dbde      
```

如果忘记添加命令，可以通过如下命令生成：

```bash
$ kubeadm token create --print-join-command
```

操作节点：只在master节点（`k8s-master`）执行，CNI

- 下载flannel的yaml文件

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

- 修改配置，指定网卡名称，大概在文件的190行，添加一行配置：

```bash
$ vi kube-flannel.yml
...      
      containers:
      - name: kube-flannel
        image: rancher/mirrored-flannelcni-flannel:v0.16.1
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=eth0  # 如果机器存在多网卡的话，指定内网网卡的名称，默认不指定的话会找第一块网卡
        resources:
          requests:
            cpu: "100m"
...
```

- 执行安装flannel网络插件

```bash
# 先拉取镜像,此过程国内速度比较慢
$ docker pull rancher/mirrored-flannelcni-flannel:v0.16.1
# 执行flannel安装
$ kubectl apply -f kube-flannel.yml
```

### 设置master节点是否可调度（可选）

操作节点：`k8s-master`

默认部署成功后，master节点无法调度业务pod，如需设置master节点也可以参与pod的调度，需执行：

```python
$ kubectl taint node k8s-master node-role.kubernetes.io/master:NoSchedule-
```

> 课程后期会部署系统组件到master节点，因此，此处建议设置k8s-master节点为可调度

### 设置kubectl自动补全

操作节点：`k8s-master`

```bash
$ yum install bash-completion -y
$ source /usr/share/bash-completion/bash_completion
$ source <(kubectl completion bash)
$ echo "source <(kubectl completion bash)" >> ~/.bashrc
```

### 调整证书时间

使用kubeadm安装的集群，证书默认有效期为1年，可以通过如下方式修改为10年。

```
$ cd /etc/kubernetes/pki

# 查看当前证书有效期
$ for i in $(ls *.crt); do echo "===== $i ====="; openssl x509 -in $i -text -noout | grep -A 3 'Validity' ; done

$ mkdir backup_key; cp -rp ./* backup_key/
$ git clone https://github.com/yuyicai/update-kube-cert.git
$ cd update-kube-cert/ 
$ bash update-kubeadm-cert.sh all

# 重建管理服务
$ kubectl -n kube-system delete po kube-apiserver-k8s-master01 kube-apiserver-k8s-master02 kube-apiserver-k8s-master03 kube-controller-manager-k8s-master01 kube-controller-manager-k8s-master02 kube-controller-manager-k8s-master03 kube-scheduler-k8s-master01 kube-scheduler-k8s-master02 kube-scheduler-k8s-master03
```

### 验证集群

操作节点： 在master节点（`k8s-master`）执行

```python
$ kubectl get nodes  #观察集群节点是否全部Ready
```

创建测试nginx服务

```python
$ kubectl run  test-nginx --image=nginx:alpine
```

查看pod是否创建成功，并访问pod ip测试是否可用

```bash
$ kubectl get po -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE         NOMINATED NODE   READINESS GATES
test-nginx-5bd8859b98-5nnnw   1/1     Running   0          9s    10.244.1.2   k8s-slave1   <none>           <none>
$ curl 10.244.1.2
...
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### 清理集群

```
如果你的集群安装过程中遇到了其他问题，我们可以使用下面的命令来进行重置：

# 在全部集群节点执行
kubeadm reset
ifconfig cni0 down && ip link delete cni0
ifconfig flannel.1 down && ip link delete flannel.1
rm -rf /run/flannel/subnet.env
rm -rf /var/lib/cni/
mv /etc/kubernetes/ /tmp
mv /var/lib/etcd /tmp
mv ~/.kube /tmp
iptables -F
iptables -t nat -F
ipvsadm -C
ip link del kube-ipvs0
ip link del dummy0
```

