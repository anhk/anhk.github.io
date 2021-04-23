---
layout: post
title: 快速部署Kubernetes
category: Kubernetes
---

这里介绍快速部署Kubernetes的方法，需要准备至少2台主机，可以是云主机也可以是物理机；其中一台作为管理节点-Master，另外一台或多台作为Pod节点。

 <!--more-->

## 准备工作

首先对主机进行依赖软件及环境的部署，我这里以ubuntu为例（当前版本为18.10）



### 安装curl & ca证书

```bash
$ apt-get update && apt-get install -y apt-transport-https ca-certificates curl
```



### 安装Kube* & Docker

```bash
# 这里使用阿里云的ubuntu源，加快下载速度，其他系统可以去https://mirrors.aliyun.com/看看

# --- kubernetes 源
$ curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
$ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

# --- docker 源
$ curl -s http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | apt-key add -
$ cat <<EOF >/etc/apt/sources.list.d/docker-ce.list
deb [arch=amd64] https://mirror.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu xenial stable
EOF

# 安装软件
$ apt-get update
$ apt-get install -y kubelet kubeadm kubectl docker-ce

# 修改Docker使用国内镜像，加快下载速度
$ cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "registry-mirrors": ["https://mirror.baidubce.com"]
}
EOF

# 让docker随系统启动，它会把kubernetes的镜像全都启动起来
$ systemctl daemon-reload && service docker restart && systemctl enable docker
```



### 修改Hostname 以及配置 ssh-key

由于kubernetes大量使用了hostname，如果没有预先设置好的话，后续改起来很麻烦。

```bash
# Master 主机
$ echo 'k8s-master' > /etc/hostname
$ hostname k8s-master
$ echo '127.0.0.1 k8s-master' >> /etc/hosts
$ echo '192.168.0.6 k8s-pod' >> /etc/hosts  # 这里192.168.0.6为Pod主机的IP地址

# pod
$ echo 'k8s-pod' > /etc/hostname
$ hostname k8s-pod
$ echo '127.0.0.1 k8s-pod' >> /etc/hosts
$ echo '192.168.0.2 k8s-master' >> /etc/hosts # 这里192.168.0.2为Master主机的IP地址

# Master主机 --- 创建SSH密钥并同步公钥到Pod主机
$ ssh-keygen -t rsa
$ ssh-copy-id -i /root/.ssh/id_rsa.pub 127.0.0.1
$ ssh-copy-id -i /root/.ssh/id_rsa.pub k8s-pod
```



## 部署Master主机-管理节点



### 管理节点

在Master主机上部署Kubernetes的管理节点，由于需要更换下载源为阿里云，所以需要自定义yaml文件。

```bash
cat > k8s-init.yml << EOF
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
nodeRegistration:
  kubeletExtraArgs:
    pod-infra-container-image: registry.aliyuncs.com/google_containers/pause-amd64:3.1
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
imageRepository: registry.aliyuncs.com/google_containers
kubernetesVersion: v1.17.0
networking:
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
EOF
```

文件编辑好后，就可以创建集群了，命令如下：

```bash
$ kubeadm init --config k8s-init.yml --v=5   # --v=5为打印日志的级别
```

**创建集群的命令会告诉如何进行后续操作，包括如何配置kubectl和slave节点。**



### 配置kubectl

```bash
# 这三条命令为刚刚kubeadm init输出的结果
$  mkdir -p $HOME/.kube
$  cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$  chown $(id -u):$(id -g) $HOME/.kube/config
```



### 安装Flannel

由于刚刚初始化的节点没有网络配置，所以需要使用Flannel初始化网络，我在官网下载的kube-flannel.yml无法创建成功，在大量Google之后找到一个好用的。这里提供[下载链接](/files/kubernetes/kube-flannel.yml)。

```bash
$ kubectl apply -f kube-flannel.yml
```

稍等一会就会创建成功，而后网络的状态信息就从NotReady变为OK了



### 配置Pod节点

这里需要使用的是刚刚Master节点 `kubeadm init` 命令输出的结果，我这里是个示例。

```bash
$ kubeadm join 192.168.0.62:6443 --token 28ln2e.6xocvsm7vqmhd1zt \
    --discovery-token-ca-cert-hash sha256:0da0e28f7fa611e3a6ee49c303e7283f653696eee53a0180ec2e02a7a9a3a797
```




## 结尾

现在kubernetes集群就搭建好了，可以去学习kubectl命令如何使用了。