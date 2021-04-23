---
layout: post
title: 为CentOS6系统升级较新的GCC
category: [CentaOS, Upgrade]
---

公司为了节省成本，需要将CentOS7上的一些程序运行在CentOS6.6上，检查了一下依赖发现最主要C++11特性，而CentOS6自带的GCC4.4.7是不支持C++11的，故需要升级系统的GCC版本。


 <!--more-->
## 安装CentOS的仓库

Centos的devtool仓库里有比较新的gcc版本，当前应当是4.8.2版本，正好满足C++11的最低要求。

```bash
# 安装仓库
curl https://people.centos.org/tru/devtools-2/devtools-2.repo -o /etc/yum.repos.d/devtools-2.repo
```

## 安装GCC软件
```bash
# 安装软件
yum install devtoolset-2-gcc devtoolset-2-binutils devtoolset-2-gcc-c++

# 安装完成的软件目录
ls -l /opt/rh/devtoolset-2/root/usr/bin/

# 查看版本
/opt/rh/devtoolset-2/root/usr/bin/gcc -v
```

## 修改CMake的编译器

由于项目是使用的cmake，故不用修改操作系统，直接通过环境变量改就行了

```bash
CC=/opt/rh/devtoolset-2/root/usr/bin/gcc CXX=/opt/rh/devtoolset-2/root/usr/bin/g++ cmake
```



嗯嗯，看一下输出信息，ok了。

```bash
-- The C compiler identification is GNU 4.8.2
-- The CXX compiler identification is GNU 4.8.2
-- Check for working C compiler: /opt/rh/devtoolset-2/root/usr/bin/gcc
-- Check for working C compiler: /opt/rh/devtoolset-2/root/usr/bin/gcc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features

......
```

