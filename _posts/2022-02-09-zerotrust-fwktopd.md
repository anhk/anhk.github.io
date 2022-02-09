---
layout: post
title: 零信任敲门fwktop调研
---
fwknop文档描述其主要功能是隐藏ssh服务的daemon服务。

<!--more-->

# 零信任敲门fwknop调研

代码路径：https://www.github.com/mrash/fwknop 

文档路径：https://www.cipherdyne.org/fwknop/

fwknop已经内置在最新版本的ubuntu官方仓库了，直接使用apt install即可安装。

## 服务端安装及配置

本次在带sshd的ubuntu容器中安装fwknop-server

```bash
# 启动带sshd的ubuntu容器
$ docker run -itd --cap-add NET_ADMIN -p 2222:22 -p 62201:62201/udp --name ubuntu ir0cn/ubuntu

# 安装fwknop-server
$ docker exec -it ubuntu bash
$ apt update -y && apt install -y fwknop-server
```

*注：如果国内云主机访问archive.ubuntu.com超时或失败，则更换为国内源*

安装完成后会在 /etc/fwknop目录出现两个配置文件：

```bash
$ ls /etc/fwknop/
access.conf  fwknopd.conf
```

**配置access.conf**

这里使用`fwknopd`生成共享密钥，并将密钥配置在 `/etc/fwknop/access.conf`中的`__CHANGEME__`位置。

```bash
$ fwknopd --key-gen
KEY_BASE64: XyrrRvr50P7XCx8HYy6Svev2XS2vSHMMAEJbl883WLw=
HMAC_KEY_BASE64: dhZSbzAdHpr1OHSKuVE4sFPcaIOxdpI2LiWeO4CUJV9RgGhJkFV9VjEkdWYqW5VKK9EmSnV1ekr0uPc8Q4r9/w==

```

fwknopd同时支持使用gnuPG的非对称密钥进行认证，这里暂时忽略。



**配置fwknopd.conf**

fwknopd支持使用udp协议敲门，在udp被防火墙禁用的场景下也支持使用tcp协议敲门。

```bash
# 修改 /etc/fwknop/fwknopd.conf
FIREWALL_EXE                /usr/sbin/iptables;
```



**设置默认防火墙规则**

```bash
$ iptables -I INPUT 1 -i eth0 -p tcp --dport 22 -j DROP
$ iptables -I INPUT 1 -i eth0 -p tcp --dport 22 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```



**启动fwknopd**

由于fwknopd在启动的时候读取 /dev/log，故需要安装并运行syslogd

```bash
$ apt install -y inetutils-syslogd && syslogd

# 启动fwknopd
$ fwknopd
```



## 客户端验证

**安装客户端**

```bash
$ apt install -y fwknop-client
```

修改配置文件`~/.fwknoprc`

```
[default]
KEY_BASE64: XyrrRvr50P7XCx8HYy6Svev2XS2vSHMMAEJbl883WLw=
HMAC_KEY_BASE64     dhZSbzAdHpr1OHSKuVE4sFPcaIOxdpI2LiWeO4CUJV9RgGhJkFV9VjEkdWYqW5VKK9EmSnV1ekr0uPc8Q4r9/w==
USE_HMAC                    Y

[116.196.113.26]
SPA_SERVER          116.196.113.26
ACCESS              tcp/22
ALLOW_IP            resolve
```



**执行命令**

```bash
$ fwknop -A tcp/22 -n 116.196.113.26 --wget-cmd /usr/bin/wget
```



## **服务端验证**

```bash
$ fwknopd --fw-list
Listing rules in fwknopd iptables chains...

Chain FWKNOP_INPUT (1 references)
num  target     prot opt source               destination
1    ACCEPT     tcp  --  101.200.174.196      0.0.0.0/0            tcp dpt:22 /* _exp_1644395048 */
```

*注：此敲门记录的有效期是30秒*


