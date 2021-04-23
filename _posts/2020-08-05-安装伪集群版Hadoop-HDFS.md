---
layout: post
title: 安装伪集群版Hadoop-HDFS
category: [Hadoop]
---

网上搜了一些中文资料，大多前言不搭后语，按照步骤做了几次都失败了。现在记录下成功记录。

 <!--more-->

## 官方文档

尽量参照[官方文档](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)

## 安装Java

**安装Java-1.8**
```bash
$ apt install -y openjdk-8-jre-headless openjdk-8-jdk-headless
```

**配置JAVA_HOME**

```bash
$ echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64' >> ~/.bashrc
$ . ~/.bashrc
```

## 配置SSH Key & Host

**配置hostname**
```bash
$ echo hadoop > /etc/hostname
$ hostname hadoop
```

**配置sshKey**
```bash
$ ssh-keygen -t rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

## 下载Hadoop并解压

```bash
$ wget 'https://mirror.bit.edu.cn/apache/hadoop/common/hadoop-3.3.0/hadoop-3.3.0.tar.gz'
$ tar xvf hadoop-3.3.0.tar.gz -C /usr/local
$ ln -s /usr/local/hadoop-3.3.0 /usr/local/hadoop
```

## 修改配置文件

**etc/hadoop/core-site.xml**
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

**etc/hadoop/hdfs-site.xml**
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

**etc/hadoop/hadoop-env.sh**
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
```

## 启动HDFS

```bash
# format filesystem
$ bin/hdfs namenode -format

# start namenode & datanode
$ sbin/start-dfs.sh

# test
$ curl http://localhost:9870
```
