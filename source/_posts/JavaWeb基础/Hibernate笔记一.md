---
title: Hibernate笔记一
categories: JavaEE
tags:
  - JavaEE
  - 数据库
  - Hibernate
description: Hibernate框架的介绍 / 配置方法 / 使用方法
abbrlink: dd159bfa
date: 2016-08-08 21:20:47
---


## Hibernate介绍

Hibernate是轻量级JavaEE应用的持久层解决方案，是一个关系数据库ORM框架.Hibernate的持久化方案，将用户从原始的JDBC底层SQL访问中解放出来,用户无须关注底层数据库操作，只要通过操作映射到数据表的Java对象，就可以对数据库进行增删改查

	ORM (Object Relational Mapping对象关系映射)
	ORM 就是通过将Java对象映射到数据库表，通过操作Java对象，就可以完成对数据表的操作

	流行数据库框架 
	1. JPA(Java Persistence API) : 通过注解描述对象与数据表映射关系 （只有接口规范）
	2. Hibernate : 最流行ORM框架，通过对象-关系映射配置，可以完全脱离底层SQL ， Hibernate实现JPA规范 
	3. MyBatis : 本是apache的一个开源项目 iBatis，支持普通 SQL查询，存储过程和高级映射的优秀持久层框架 （企业主流）
		* MyBaits 并不是完全ORM ， 需要在xml中配置SQL语句
	4. Apache DBUtils(超链接到DBUtils那节笔记里面去)
	5. Spring JDBCTemplate
	
	SQL语句封装程度 Hibernate > MyBatis > Apache DBUtils 、Spring JDBCTemplate

## Hibernate中的ORM实现
数据库中的表与java中的类对应
表中的记录是与类中的对象对应
表中的字段是与类中的属性对应

![ORM对应关系.png](https://ooo.0o0.ooo/2016/08/07/57a741f54f20a.png)

## Hibernate体系结构

![Hibernate架构.png](https://ooo.0o0.ooo/2016/08/07/57a7403d48e16.png)

## Hibernate目录结构

![Hibernate目录结构.png](https://ooo.0o0.ooo/2016/08/07/57a7433c54eb5.png)

## Hibernate使用
### 1.导入jar包
导包的内容参见上一步,目录结构中导入即可

特别介绍: log4j(链接上log4j的单独的笔记内容)

### 2.编写类和关系映射配置xx.hbm.xml

	public class Customer {
		private int id;
		private String name;
		private int age;
		private String city;
		...
	}
hibernate 完全ORM，只需要操作Customer类对象， 自动生成SQL 操作customer 表
但是需要为实体类和数据表进行关系映射配置
 1.在实体类所在的包下创建名为:`类名.hbm.xml`文件,eg:`Customer.hbm.xml`
 2.配置规则参见 hibernate3.jar org/hibernate/hibernate-mapping-3.0.dtd
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE hibernate-mapping PUBLIC 
	    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
	<hibernate-mapping>
		<!-- 配置类 和数据表 对应关系 -->
		<class name="cn.itcast.domain.Customer" table="customer" select-before-update="true">
			 <!-- 配置哪个属性 关联数据表主键 -->
			 <id name="id" column="id" type="integer">
			 	<!-- 主键生成策略 -->
			 	<generator class="identity"></generator>
			 </id>
			 <!-- 普通属性 -->
			 <property name="name" column="name" type="string"></property>
			 <!-- 如果属性名和列名相同 可以省略 column -->
			 <property name="age" type="integer" ></property>
			 <!-- 类型也可以使用默认生成规则，省略type -->
			 <property name="city"></property>
			 <property name="address" >
				<column name="address" sql-type="varchar(20)"/>
			</property>
		</class>
	</hibernate-mapping>
 配置属性到列映射时，指定类型，类型有三种写法
	第一种 java类型  java.lang.String
	第二种 hibernate类型 string
	第三种 SQL类型 varchar(20)

### 3.配置核心配置文件hibernate.cfg.xml
Hibernate框架支持两种 Hibernate属性配置方式
 1.hibernate.properties
	采用properties方式，必须手动编程加载 hbm文件或者 持久化类
 2.hibernate.cfg.xml 
	采用XML配置方式，可以通过配置添加hbm文件
	规则参见 `hibernate3.jar /org/hibernate/hibernate-configuration-3.0.dtd`

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE hibernate-configuration PUBLIC
		"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
		"http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
	<hibernate-configuration>
		<!-- 会话连接工厂，建立数据库连接需要SessionFactory -->
		<session-factory>
			<!-- JDBC连接基本参数 -->
			<property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
			<property name="hibernate.connection.url">jdbc:mysql:///hibernatetest</property>
			<property name="hibernate.connection.username">root</property>
			<property name="hibernate.connection.password">1234</property>
			<!-- 配置数据库方言，便于生成一些与数据库相关SQL方言 -->
			<property name="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</property>
			<!-- DDL策略   可以根据需要自动创建数据表 -->
			<property name="hibernate.hbm2ddl.auto">update</property>
			<!-- 将SQL语句 输出到控制台 -->
			<property name="hibernate.show_sql">true</property>
			<property name="hibernate.format_sql">true</property>
			
			<!-- 导入映射的po类的xx.hbm.xml映射文件-->			
			<mapping resource="cn/itcast/domain/Customer.hbm.xml"></mapping>
		</session-factory>
	</hibernate-configuration>


注意:
	配置文件必须放在根目录下
	通常采用方式2来配置

## 编程操作使用Hibernate框架

web层的javabean叫vo viewobject

service层    bo  business object

dao层   po 持久化对象
po类需要有个无参构造,get/set方法,私有化属性,属性使用包装类型,不要用基本类型,类不能为final(因为要为他生成代理),
还需要有一个与表中主键映射的字段

----------------
hbm映射介绍


pojo是个啥?  视频两分钟的时候介绍了下