---
layout: post
title: PostgreSQL常用语句
---
PostgreSQL常用语句
<!--more-->

## databases

1. 查看数据库列表
```sql
\l
--- 或
SELECT datname FROM pg_database;
```

2. 创建数据库
```sql
create database school;
```

3. 切换数据库
```sql
\c school
```

## tables
1. 查看表列表
```sql
\d 
--- 或
\d 数据库名称
```

2. 查看表结构
```sql
\d student
```

3. 创建表
```sql
CREATE TABLE student (
   s_id SERIAL,
   s_name VARCHAR(64) DEFAULT NULL,
   s_birth TIMESTAMP DEFAULT NULL,
   s_addr VARCHAR(255) DEFAULT NULL,
   s_phone VARCHAR(64) DEFAULT NULL,
  PRIMARY KEY (s_id)
) ;
```
