---
layout: post
title: 为Kubernetes配置私有仓库
category: [Kubernetes]
---

如题

 <!--more-->


## 创建访问私有仓库的令牌-Secret

```bash
kubectl create secret docker-registry regcred   \
    --docker-server=<your-registry-server>      \
    --docker-username=<your-name>               \
    --docker-password=<your-pword>
```



## 查看私有仓库的令牌

```bash
kubectl get secret regcred -o json
```



## 创建一个使用私有仓库的Pod

创建 [pods/private-reg-pod.yaml](https://k8s.io/examples/pods/private-reg-pod.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred
```

其中使用imagePullSecrets配置的凭证来获取私有镜像

```bash
kubectl create -f my-private-reg-pod.yaml
kubectl get pod private-reg
```

