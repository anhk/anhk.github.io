---
layout: post
title: Kubernetes 常用命令
---
Kubernetes 常用命令
<!--more-->

# 重启StatefulSet/Deployment

```bash
kubectl rollout restart statefulset elasticsearch
kubectl rollout restart deployment elasticsearch
```

# 多集群切换上下文

### 查看上下文
```bash
$ kubectl config get-contexts
CURRENT   NAME              CLUSTER           AUTHINFO    NAMESPACE
          bastion-133.174   bastion-133.174   crd-admin   skywing
*         bastion-133.9     bastion-133.9     crd-admin   skywing
          minikube          minikube          minikube    default
```

### 切换上下文
```bash
$ kubectl config use-context minikube
Switched to context "minikube".
```