---
layout: post
title: 为CentOS&Ubuntu升级最新版本Golang
category: [Golang, Upgrade]
---

发现操作系统默认带的golang版本都比较低，这里描述CentOS7和Ubuntu 18.04升级到最新版本golang的步骤。



 <!--more-->



## CentOS 7

[官方连接](https://go-repo.io/)

```bash
$ rpm --import https://mirror.go-repo.io/centos/RPM-GPG-KEY-GO-REPO
$ curl -s https://mirror.go-repo.io/centos/go-repo.repo | tee /etc/yum.repos.d/go-repo.repo
$ yum install golang
```

## CentOS 6
如果是CentOS6，访问`mirror.go-repo.io`会由于CA证书不信任而拒绝，故需要先更新系统的CA证书，然后使用epel的仓库，可以直接下载最新的golang

```bash
$ yum install ca-certificates
$ yum install epel-release

# 修改/etc/yum.repos.d/epel.repo，去掉baseurl=的注释，将mirrorlist注释
$ yum clean all && yum update
$ yum install go
```


## Ubuntu 18.04

[官方连接](https://github.com/golang/go/wiki/Ubuntu)

```bash
$ sudo add-apt-repository ppa:longsleep/golang-backports
$ sudo apt-get update
$ sudo apt-get install golang-go
```

如果报错 `add-apt-repository not found`，则需要安装
```bash
$ sudo apt install software-properties-common
```


如果 `ppa.launchpad.net` 比较卡的化，可以修改域名为：`https://launchpad.proxy.ustclug.org`；文件位置`/etc/apt/sources.list.d/longsleep-ubuntu-golang-backports-bionic.list `