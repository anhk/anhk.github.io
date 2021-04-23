---
layout: post
title: 安装伪集群版Hadoop-HBase
category: [Hadoop]
---

网上搜了一些中文资料，大多前言不搭后语，按照步骤做了几次都失败了。现在记录下成功记录。

 <!--more-->

## 官方文档

尽量参照[官方文档](https://hbase.apache.org/book.html#standalone_dist)

## 下载HBase包并解压

```bash
$ wget https://mirror.bit.edu.cn/apache/hbase/2.3.0/hbase-2.3.0-bin.tar.gz
$ tar xvf hbase-2.3.0-bin.tar.gz  -C /usr/local/
$ ln -s /usr/local/hbase-2.3.0/ /usr/local/hbase
```

## 修改配置文件

**conf/hbase-site.xml**
```xml
<property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
</property>
<property>
    <name>hbase.tmp.dir</name>
    <value>./tmp</value>
</property>
<property>
    <name>hbase.rootdir</name>
    <value>hdfs://localhost:9000/hbase</value>
</property>
<property>
    <name>hbase.wal.provider</name>
    <value>filesystem</value>
</property>
<property>
    <name>hbase.wal.meta_provider</name>
    <value>filesystem</value>
</property>

```

**conf/hbase-env.sh**
```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

## 启动HBase

```bash
./bin/start-hbase.sh
```

## 验证

```bash
$ ./bin/hbase shell
HBase Shell
Use "help" to get list of supported commands.
Use "exit" to quit this interactive shell.
For Reference, please visit: http://hbase.apache.org/2.0/book.html#shell
Version 2.3.0, re0e1382705c59d3fb3ad8f5bff720a9dc7120fb8, Mon Jul  6 22:27:43 UTC 2020
Took 0.0012 seconds
hbase(main):001:0>  create 'super','boy'
Created table super
Took 0.6404 seconds
=> Hbase::Table - super
hbase(main):002:0>  list 'super'
TABLE
super
1 row(s)
Took 0.0186 seconds
=> ["super"]
hbase(main):003:0>

```