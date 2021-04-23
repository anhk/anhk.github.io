---
layout: post
title: 安装伪集群版Hadoop-HIVE
category: [Hadoop]
---

网上搜了一些中文资料，大多前言不搭后语，按照步骤做了几次都失败了。现在记录下成功记录。

 <!--more-->

## 官方文档

尽量参照[官方文档](https://cwiki.apache.org/confluence/display/Hive/GettingStarted)

## 下载HIVE包并解压

```bash
$ wget https://mirror.bit.edu.cn/apache/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
$ tar xvf apache-hive-3.1.2-bin.tar.gz  -C /usr/local/
$ ln -s /usr/local/apache-hive-3.1.2-bin/ /usr/local/hive
```

## 环境变量

**~/.bashrc**
```bash
export HADOOP_HOME=/usr/local/hadoop
export HIVE_HOME=/usr/local/hive
```


## 使用Hadoop设置Hive所用的目录

```bash
$ $HADOOP_HOME/bin/hadoop fs -mkdir       /tmp
$ $HADOOP_HOME/bin/hadoop fs -mkdir -p    /user/hive/warehouse
$ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /tmp
$ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /user/hive/warehouse 
```

## 初始化库

```bash
$ $HIVE_HOME/bin/schematool -dbType derby -initSchema
$ $HIVE_HOME/bin/hive
```

## 运行HIVE出错解决
**错误1：** 执行`hive`命令，由于hive和hadoop的guava包版本不一致，造成抛异常

```bash
$ $HIVE_HOME/bin/hive
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/apache-hive-3.1.2-bin/lib/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop-3.3.0/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Exception in thread "main" java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V
        at org.apache.hadoop.conf.Configuration.set(Configuration.java:1380)
        at org.apache.hadoop.conf.Configuration.set(Configuration.java:1361)
        at org.apache.hadoop.mapred.JobConf.setJar(JobConf.java:536)
        at org.apache.hadoop.mapred.JobConf.setJarByClass(JobConf.java:554)
        at org.apache.hadoop.mapred.JobConf.<init>(JobConf.java:448)
        at org.apache.hadoop.hive.conf.HiveConf.initialize(HiveConf.java:5141)
        at org.apache.hadoop.hive.conf.HiveConf.<init>(HiveConf.java:5099)
        at org.apache.hadoop.hive.common.LogUtils.initHiveLog4jCommon(LogUtils.java:97)
        at org.apache.hadoop.hive.common.LogUtils.initHiveLog4j(LogUtils.java:81)
        at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:699)
        at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:683)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.apache.hadoop.util.RunJar.run(RunJar.java:323)
        at org.apache.hadoop.util.RunJar.main(RunJar.java:236)
```

**解决办法：**  [参照链接](https://issues.apache.org/jira/browse/HIVE-22915)
```bash
$ mv  /usr/local/apache-hive-3.1.2-bin/lib/guava-19.0.jar /usr/local/apache-hive-3.1.2-bin/lib/guava-19.0.jar.bak
$ cp /usr/local/hadoop/share/hadoop/common/lib//guava-27.0-jre.jar /usr/local/apache-hive-3.1.2-bin/lib/
```


**错误2：** schematool运行出错

```
$ $HIVE_HOME/bin/schematool -dbType derby -initSchema
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/apache-hive-3.1.2-bin/lib/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop-3.3.0/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:        jdbc:derby:;databaseName=metastore_db;create=true
Metastore Connection Driver :    org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:       APP
Starting metastore schema initialization to 3.1.0
Initialization script hive-schema-3.1.0.derby.sql


Error: FUNCTION 'NUCLEUS_ASCII' already exists. (state=X0Y68,code=30000)
org.apache.hadoop.hive.metastore.HiveMetaException: Schema initialization FAILED! Metastore state would be inconsistent !!
Underlying cause: java.io.IOException : Schema script failed, errorcode 2
Use --verbose for detailed stacktrace.
*** schemaTool failed ***

```

**解决办法：** 
```bash
$ mv metastore_db metastore_db.tmp
$ $HIVE_HOME/bin/schematool -dbType derby -initSchema
```