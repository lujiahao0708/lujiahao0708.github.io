---
title: 《MySQL必知必会》第1-3章 MySQL介绍
categories: MySQL必知必会
toc: true
tags:
  - MySQL
  - MySQL必知必会
abbrlink: 8b4d1597
date: 2019-12-19 19:54:41
---

# 1. 术语介绍
- 数据库(database) : 保存有组织的数据的容器(通常是一个文件或一组文件)
- 表(table) : 某种特定类型数据的结构化清单(具有唯一性,同一数据库中表名不能相同,不同数据库可以相同)
- 模式(schema) : 关于数据库和表的布局及特性的信息
- 列(column) : 表中的一个字段,所有表都是由一个或多个列组成
- 数据类型(datatype) : 所容许的数据的类型.每个表列都有相应的数据类型,它限制该列中存储的数据
- 行(row) : 表中的一个记录
- 主键(primary key) : 一列(或一组列),其值能够唯一区分表中的每行,主键列不允许NULL值
- 关键字(key word) : MySQL语言保留字,不可用关键字命名表或列

<!-- more -->

# 2. SQL / DBMS / MySQL
## 2.1 SQL优点:
- SQL不是某个特定数据库供应商的专有语言,几乎所有重要的DBMS都支持SQL
- SQL简单易学
- SQL简单灵活,可以进行分厂复杂和高级的数据库操作

## 2.2 数据库管理系统DBMS
- 基于共享文件系统
  - Microsoft Access / FileMarker
- 基于客户机-服务器
  - MySQL / Oracle / Microsoft SQL Server

## 2.3 MySQL优点
- 低成本
- 高性能
- 可信赖
- 简单易用

# 3. MySQL工具
## 3.1 命令行
登录命令:
```shell
mysql -h localhost -P 3306 -uroot -p1234
```
> 注意 : 
> 最后这个`-p`和密码之间不能有空格
> 其他的`-h`或者`-u`和参数间的空格是可有可无的

登录后提示:
```shell
root@c67b08791485:/# mysql -h localhost -P 3306 -u root -p1234
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.6-m16 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

## 3.2 Navicat
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/MySQL/%E3%80%8AMySQL%E5%BF%85%E7%9F%A5%E5%BF%85%E4%BC%9A%E3%80%8B%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/Navicat%E9%93%BE%E6%8E%A5mysql.png)

> 其他工具,请自行Google

# 4. 简单SQL语句
## 4.1 显示所有数据库
```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema      |
| eladmin            |
| example        |
| lfstudio           |
| mysql              |
| performance_schema |
| shiro              |
| shiro2          |
+--------------------+
18 rows in set (0.01 sec)
```
> 我这里有一些自己测试的数据库

## 4.2 切换到指定数据库
```sql
mysql> use shiro2;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

## 4.3 显示当前数据库中所有表
```sql
mysql> show tables;
+-----------------------+
| Tables_in_shiro2      |
+-----------------------+
| base_admin_permission |
| base_admin_role       |
| base_admin_user       |
+-----------------------+
3 rows in set (0.00 sec)
```

## 4.4 显示指定表中所有列
```sql
mysql> show columns from base_admin_role;
+-------------+--------------+------+-----+---------+----------------+
| Field       | Type         | Null | Key | Default | Extra          |
+-------------+--------------+------+-----+---------+----------------+
| id          | int(11)      | NO   | PRI | NULL    | auto_increment |
| role_name   | varchar(30)  | NO   |     | NULL    |                |
| role_desc   | varchar(100) | YES  |     | NULL    |                |
| permissions | varchar(20)  | YES  |     | NULL    |                |
| create_time | varchar(64)  | YES  |     | NULL    |                |
| update_time | varchar(64)  | YES  |     | NULL    |                |
| role_status | int(1)       | NO   |     | 1       |                |
+-------------+--------------+------+-----+---------+----------------+
7 rows in set (0.00 sec)
```

> 它对每个字段返回一行，行中包含字段名、数据 类型、是否允许NULL、键信息、默认值以及其他信息

```sql
mysql> describe base_admin_role;
+-------------+--------------+------+-----+---------+----------------+
| Field       | Type         | Null | Key | Default | Extra          |
+-------------+--------------+------+-----+---------+----------------+
| id          | int(11)      | NO   | PRI | NULL    | auto_increment |
| role_name   | varchar(30)  | NO   |     | NULL    |                |
| role_desc   | varchar(100) | YES  |     | NULL    |                |
| permissions | varchar(20)  | YES  |     | NULL    |                |
| create_time | varchar(64)  | YES  |     | NULL    |                |
| update_time | varchar(64)  | YES  |     | NULL    |                |
| role_status | int(1)       | NO   |     | 1       |                |
+-------------+--------------+------+-----+---------+----------------+
7 rows in set (0.00 sec)
```
> DESCRIBE语句 MySQL支持用DESCRIBE作为SHOW COLUMNS FROM的一种快捷方式。
> 换句话说,DESCRIBE base_admin_role;是 SHOW COLUMNS FROM base_admin_role;的一种快捷方式。

## 4.5 其他show语句
- SHOW STATUS，用于显示广泛的服务器状态信息
- SHOW CREATE DATABASE和SHOW CREATE TABLE，分别用来显示创建特定数据库或表的MySQL语句
- SHOW GRANTS，用来显示授予用户(所有用户或特定用户)的安全权限
- SHOW ERRORS和SHOW WARNINGS，用来显示服务器错误或警告消息


## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我：后端小哥，专注后端开发，希望和你一起进步！

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)
