---
layout: post
title: 安装最新的Kubernetes
category: [Kubernetes]
---

由于阿里源的kubeadm与容器镜像的版本对不上，这里使用另外一种方式进行安装；【众所周知，中国访问谷歌服务有些问题】

 <!--more-->

## 安装最新版本的kubeadm和docker-ce

这里不在赘述了，看 [这篇文章](https://ir0.cn/quick-install-kubernetes.html)


调整docker使用`cgroupdriver=systemd`

```bash
$ cat > /etc/docker/daemon.json << EOF
{
    "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

$ systemctl restart docker
```


## 国外站点上传镜像到dockerhub

使用脚本 k8s-outside.sh，从国外站点拉取最新镜像，上传到dockerhub

```bash
#!/bin/bash

images=$(kubeadm config images list 2>/dev/null | awk '{print $1}')

for imageName in  ${images[@]}; do
    docker pull $imageName || exit -1
    newImageName=$(echo $imageName | awk -F/ '{print $NF}' | sed 's@:@__@')
    docker tag $imageName ir0cn/google_containers:$newImageName || exit -1
    docker push ir0cn/google_containers:$newImageName || exit -1
done
```



## 国内站点下载镜像

使用脚本 k8s-inside.sh，从dockerhub拉取镜像并改名，每台机器均需执行

```bash
#!/bin/bash

images=$(kubeadm config images list 2>/dev/null | awk '{print $1}')

for imageName in ${images[@]}; do
    newImageName=$(echo $imageName | awk -F/ '{print $NF}' | sed 's@:@__@')
    docker pull ir0cn/google_containers:$newImageName || exit -1
    docker tag ir0cn/google_containers:$newImageName $imageName
    docker rmi ir0cn/google_containers:$newImageName
done
```



## 继续安装kubernetes

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16
```



## 配置kubeadm

```bas
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```



## 配置Network

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```


## kubernetes重置

```bash
kubeadm reset
rm -rf /etc/kubernetes /etc/cni/net.d
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```