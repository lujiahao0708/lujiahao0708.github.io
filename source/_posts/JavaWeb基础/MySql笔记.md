---
title: MySql笔记
categories: JavaEE
tags:
  - JavaEE
  - 数据库
  - MySql
description: MySql笔记
abbrlink: cb940dd7
date: 2016-06-30 10:01:06
---

## 1.数据库知识
分类:
1. 网状型数据库
2. 层次型数据库
3. 关系型数据库
	- Oracle ： 甲骨文，oracle公司，大型数据，收费
	- db2 ： IBM，大型数据，收费
	- Sql server ： 微软，中性数据，收费。(access office 小型)
	- Mysql ： 免费。小型。（开源） --oracle 

## 2.Mysql安装

1. 设置数据库存放目录
![QQ截图20160630165126.png](https://ooo.0o0.ooo/2016/06/30/5774def7033f8.png)
2. 选择配置类型   ![QQ截图20160630165743.png](https://ooo.0o0.ooo/2016/06/30/5774e07246de2.png)
3. 选择服务类型   ![QQ截图20160630165834.png](https://ooo.0o0.ooo/2016/06/30/5774e09946abe.png)
4. 选择数据库类型  ![QQ截图20160630165920.png](https://ooo.0o0.ooo/2016/06/30/5774e0c7adeb1.png)
5. 设置并发连接数  ![QQ截图20160630170001.png](https://ooo.0o0.ooo/2016/06/30/5774e0f110a4f.png)
6. 配置网络参数    ![QQ截图20160630170103.png](https://ooo.0o0.ooo/2016/06/30/5774e131a2591.png)
7. 设置字符集      ![QQ截图20160630170134.png](https://ooo.0o0.ooo/2016/06/30/5774e14d9aaaf.png)
8. 设置服务和环境静变量 ![QQ截图20160630170302.png](https://ooo.0o0.ooo/2016/06/30/5774e1a4e7024.png)
9. 设置数据库密码  ![QQ截图20160630170328.png](https://ooo.0o0.ooo/2016/06/30/5774e1cab29f8.png)
10. 测试

		C:\Users\xxxxx>mysql --version
		mysql  Ver 14.14 Distrib 5.5.27, for Win64 (x86)

## 3.Mysql登录&修改数据库密码
	mysql -h 主机(ip地址) -u 账号 -p 密码

	use mysql;
	update user set password=password('1234') where user='root';

## 4.常用命令
- 显示当前数据库服务器中的数据库列表 : `show databases;`
- 使用数据库 : `use 数据库名;`
- 显示数据库中的数据表 : `show tables;`
- 显示当前所使用的数据库名称 : `select database();`
- 显示当前数据库的状态 : `status;`
- 显示某个表的表结构 : `desc 表名;`
- 显示所有支持的字符集 : `show character set;`
- 查看创建表的sql语句 : `show create table 表名;`

## 5.SQL语句

### 5.1 DDL数据定义语言(了解)
	> 操作数据库或表结构

#### 5.1.1 数据库操作
- 创建数据库 : `create database 数据库名称;`
- 查询数据库创建语句 : `show create database 数据库名称;`
- 删除数据库 : `drop database [if exists] 数据库名称;`
- 修改数据 : `alter database 数据库名  character set 字符集 collate 比较方式`不建议修改。

#### 5.1.2 表结构
	> 操作表之前,必须先切换数据库.

- 创建表 : `create table 表名(字段描述,字段描述,...);`
	
		create table users(
			id varchar(32),
			name varchar(50),
			age int
		);

- 删除表 : `drop table 表名;`
- 修改字段类型 : `alter table 表名 modify 字段名称 新类型;`
- 修改字段名称 : `alter table 表名 change 旧字段 新字段 新字段类型;`
- 添加字段 : `alter table 表名 add column 字段名称 字段类型;`
- 删除字段 : `alter table 表名 drop column 字段名称;`
- 重命名表名 : `alter table 表名 rename [to] 新表明;`

#### 5.1.3 数据类型
字符串

	char(n) 固定字符串，例如：char(5) 表示可以存放5个字符，且必须是5个。
		如果插入 “abc”,结果“abc  ”  右边自动添加空格。
	varchar(n) 可变长字符串，例如：varchar(5),表示最多存放5个字符，如果不够就原样存放。
		如果插入“abc”，结果“abc”
数字

	bit			比特
	tinyint		byte
	mediumint	short
	int			int		【】
	bigint		long
	float			float
	double(m,d)	double 【】 --m数字长度，d精度及小数位
	numeric		Number	所有数字
		例如：double(5,2) 5表示整个数字为5位，2表示小数位2个。最大值。999.99
时间日期

	** 之后使用java日期时间类型：java.util.Date 。如果要使用java.sql..类型，只能存放dao层
	date 日期				java.sql.Date
	datetime 日期时间			---
	time 时间				java.sql.Time
	timestamp 时间戳			java.sql.Timestamp
		 
		sql转util ： java.util.Date date = new java.sql.Date(long);
		util转sql ： new java.sql.Date(  new java.util.Date().getTime()  )
		
大数据

	字节：存放二进制  （java.sql.Blob ：Binary Large Object 二进制大对象）
		TINYBLOB  255
		blob		64k
		longblob	4G
	字符：存放文本 （java.sql.Clob ：Character Large Object 字符大对象）
		TINYTEXT	255
		text		64k
		longtext 4G


### 5.2 DML数据操作语言(掌握)
	> 对表中的数据进行增删改的操作
#### 5.2.1 插入数据
	insert into 表名(字段列表) values(字段对应值);

注意:

	多个字段之间使用逗号分隔
	字段值必须使用引号(建议单引号),如果是整形数据引号可以省略
	字段默认值为null

#### 5.2.2 更新数据

	update 表名 set 字段名=字段值,字段名=字段值,...;
	update 表名 set 字段名=字段值,字段名=字段值,...where 条件;
#### 5.2.3 删除数据
	delete from 表名 [where 条件];
注意:

	delete from users; 删除表中的所有数据.进行删除操作留心,一般情况应用系统不进行数据删除,提供一个标记字段,逻辑删除(0表示已删除,1表示没有删除).

### 5.3 约束
> 给字段添加规则，约定内容编写。最终作用保证数据的完整性，一致性等。

#### 5.3.1 主键约束
	关键字:primary key
	要求:数据唯一,不能为null.

1.定义表:声明字段时,定义主键.(primary key)只能修饰一个字段.
	create table pk01(
	  id varchar(32) primary key ,
	  content varchar(50)
	);
2.定义表，声明字段之后，在约束区域定义主键。---特点  constraint primary key (字段1,字段2,....) 可以设置多个字段
	create table pk02(
	  id varchar(32),
	  content varchar(50),
	  constraint primary key (id)
	);
3.定义表，声明字段，表创建之后。修改表结构添加约束。--特点：也可以设置多个字段，更灵活。
	create table pk03(
	  id varchar(32),
	  content varchar(50)
	);
	alter table pk03 add constraint primary key (id);
#### 5.3.2 唯一约束
	关键字:unique
	要求:被修饰的字段不能重复
1.定义表，声明字段时，定义唯一约束。--- 特点：unique只能修饰一个字段
	create table un01(
	  id varchar(32),
	  content varchar(50) unique
	);
2.定义表，声明字段之后，在约束区域定义唯一约束。---特点  constraintunique (字段1,字段2,....) 可以设置多个字段
	create table un02(
	  id varchar(32),
	  content varchar(50),
	  constraint unique (content)
	);
3.定义表，声明字段，表创建之后。修改表结构添加唯一约束。--特点：也可以设置多个字段，更灵活。
	create table un03(
	  id varchar(32),
	  content varchar(50)
	);
	alter table un03 add constraint unique (content);
#### 5.3.3 非空约束
	关键字:not null
	要求:被修饰的字段不能为null

1.定义表，声明字段时，添加约束。
	create table nn01(
	  id varchar(32),
	  content varchar(50) not null
	);
	create table nn02(
	  id varchar(32),
	  content varchar(50) not null default 'dzd'
	);

> 总结: 主键 = 唯一 + 非空

#### 5.3.4 自动增长(mysql特有)
定义:
	关键字:auto_increment
	mysql特有的一个特殊的关键字,被修饰的字段将自动的累加(oracle没有,但提供徐磊sequence)
	注意:
		1.字段类型必须是整型,一般使用int
		2.必须是key(主键/唯一),一般使用主键primary key
		3.被auto_increment修饰的字段,不需要手动维护数据,mysql将自动维护
代码示例:
	create table ai01(
		id varchar(32) auto_increment, # 错误代码  不能用于字符串
		content varchar(50)
	);
	create table ai02(
		id int auto_increment, # 错误代码 必须是primary key
		content varchar(50)
	);
	create table ai03(
		id int primary key auto_increment, # 正确代码
		content varchar(50)
	);
面试题:
	drop table ai03;    #删除表，数据和表都不存在了。
	delete from ai03;   #删除所有数据，表仍然存在。表中计数器没有重置。
	truncate table ai03;  #清空所有数据，表中的计数器将重置归0。
	delete 和 truncate 对比
		delete 将数据删除了
		truncate 先删除了表，再创建表。

#### 5.3.5 外键约束
	关键字:foreign key

#### 5.3.6 删除约束
	删除主键：mysql>   `alter table 表名 drop primary key;`
	删除唯一：修改列
	删除外键：mysql>   `alter table 表名 drop foreign key 名称;`
#### 5.3.7 cmd命令行中文乱码处理
	set names gbk;

### 5.4 DQL数据查询语言(掌握)

#### 5.4.1 无条件查询
查询所有:

	select * from users;
	select id,name,age from user;
查询部分字段:

	select id,name from user;
合并查询条件:

	select id,concat(firstname,secondname),age from users;
字段别名:

	select id,concat(firstname,secondname) as `姓名`,age from users;
特殊字符或者关键字使用重音符 ``
#### 5.4.2 带条件查询
查询分数等于60的:

	select * from users where count='60';
查询年龄大于18的学生:

	select * from users where age > 18;
查询分数在60-80之间:

	select * from users where count >= 60 and count <= 80;
	select * from users where count between 60 and 80;
查询年龄是18或20的学生:

	select * from users where age = 18 or age = 20;
模糊查询,不完全匹配:

	字段:like
	符号:	
		%匹配多个数据
			'云' 只能匹配一个云
			'%云' 匹配以云结尾的
			'云%' 匹配以云开头的
			'%云%' 包含云
		_匹配一个数据
查询分数等于60 或者  分数大于90并且年龄大于23

	select * from users where count = 60 or count > 90 and age > 23;
	select * from users where count = 60 or (count > 90 and age > 23);
	### 运算符优先级  and 优先 or
查询没有考试的学生

	select * from users where count is null;
查询所有考试的学生

	select * from users where count is not null;
运算符 不相等   !=   <>	

#### 5.4.3 聚合函数
聚合函数：对表中的数据进行统计，显示一个数据（一行一列的数据）

	聚合函数不统计 null值。
	
有多少条记录  count(* | 字段)

	select count(*) from users;
	select count(id) from users;	#7
	select count(count) from users;	#6
平均成绩  avg

	select avg(count) from users;	#不精准（没有null）
	select sum(count)/count(id) from users; #精准（计算null）
最高成绩  max

	select max(count) from users;
最小年龄  min

	select min(age) from users;
班级总成绩 sum

	select sum(count) from users;
查询所有的年龄数(排序)

	去重复 distinct
	排序 select..... order by 字段1 关键字, 字段2 关键字,....
	### 关键字 asc 升序 ， desc 降序
	select distinct age from users order by age desc;   # age asc 等效 age [asc] 
		
#### 5.4.4 分组查询
添加班级字段（classes）  

	alter table users add column classes varchar(3);
	update users set classes = 1;
	update users set classes = 2 where id='u005' or id ='u006' or id ='u007';
	update users set classes = 2 where id in ('u005','u006','u007');
查询1班和2班的平均成绩

	分组  select ... group by 分组字段;
	select classes,avg(count) from users group by classes;
	select classes,sum(count)/count(id) from users group by classes;

查询班级，平均成绩不及格的，班级成员信息

	(查询2班，成员信息)
	select * from users where classes = 2;

### 多表操作
select * from A,B where A.classes = B.classes and avg < 60;
#### 表的别名
	select ... from 表名 [as] 别名
### 子查询，一条select语句，作为另一个select一部分。
select * from users,(
	select classes,sum(count)/count(id) as cavg from users group by classes) as B 
where users.classes = B.classes and cavg < 60;

#### 子查询特点
	#查询结果一行一列，可以使用
		select id,(xxx) from
		select ... from ... where id= (xxxx)
	#查询结果一行多列(查询多个值)，使用关键字 in ，all 等
	#查询结果多行多列，可以当另一个表使用