---
layout: post
title: 容器中的iptables技术调研
---

在容器中使用iptables的场景调研

<!--more-->

# 最小权限

**特权模式**

容器具备真正的root权限，可获取宿主机的设备信息，以及可以mount到容器的路径中。

```bash
$ docker run -itd --privileged=true -p 2222:22 -p 8080:8080 --name ubuntu ir0cn/ubuntu
```

**最小网络权限**

```bash
$ docker run -itd --cap-add NET_ADMIN -p 2222:22 -p 8080:8080 --name ubuntu ir0cn/ubuntu
```

NET_ADMIN: 使容器具有网络管理权限

# 容器内iptables

在运行的容器中执行iptables，查看到如下：

```bash
root@4cc2431b406d:/# iptables -nvL
Chain INPUT (policy ACCEPT 25 packets, 4521 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 19 packets, 3953 bytes)
 pkts bytes target     prot opt in     out     source               destination
root@4cc2431b406d:/# iptables -I INPUT -s 111.202.148.47 -j DROP
root@4cc2431b406d:/# iptables -nvL
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       all  --  *      *       111.202.148.47       0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
root@4cc2431b406d:/#
```

此iptables信息只在容器中能够查看，在宿主机上查看不到该记录

**使用nsenter查看**

docker使用linux的namespace机制进行了网络的隔离，可以使用nsenter来跨namespace查看

```bash
root@Server-i-nxqp2oi4ad:~# nsenter -t $(docker inspect -f '{{ .State.Pid }}' ubuntu) -n iptables -nvL
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       all  --  *      *       111.202.148.47       0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
root@Server-i-nxqp2oi4ad:~#
```


# docker-compose中配置

```yaml
services:
  ubuntu:
    image: ir0cn/ubuntu
    ports:
      - "2022:22"
      - "8080:80"
    cap_add:
      - NET_ADMIN
```

# kubernetes中配置

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-demo
spec:
  containers:
  - name: ubuntu-demo
    image: ir0cn/ubuntu
    securityContext:
      capabilities:
        add: ["NET_ADMIN"]
```