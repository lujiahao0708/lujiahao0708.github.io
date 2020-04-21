---
title: JDBC笔记二
categories: JavaEE
tags:
  - JavaEE
  - 数据库
  - MySql
description: 'JDBC笔记第二部分,主要包括:JDBC批处理,事物,连接池三大部分.'
abbrlink: 84e56a04
date: 2016-07-09 16:17:13
---

## 1. JDBC批处理
Statement 和 PreparedStatement  都提供批处理。
	批处理：批量处理sql语句。
	Statement的批处理，可以一次性执行不同的sql语句。应用场景：系统初始化（创建数据库，创建不同表）
	PreparedStatement 的批处理，只能执行一条sql语句，实际参数可以批量。应用场景：数据的初始化

### 1.1	Statement
	addBatch(sql)  ,给批处理缓存中添加sql语句。
	clearBatch();  清空批处理缓存。
	executeBatch() , 执行批处理缓存所有的sql语句。注意：执行完成之后缓存中的内容仍然存在。

### 1.2	PreparedStatement
	addBatch()  , 将实际参数批量设置缓存中。注意：获得预处理之前必须提供sql语句。
	
	mysql 默认没开启批处理。
	通过URL之后参数开启， ?rewriteBatchedStatements = true

## 2. 事务
### 2.1 什么是事务
- 事务：一组业务逻辑操作，要么全部成功，要么全部不成功。
- 事务特性：ACID
	- 原子性：一个事务是不可划分的一个整体。要么成功，要么不成功。
	- 一致性：事务操作前后数据一致。（数据完整）。转账前后，两个人和一样的。
	- 隔离性：多个事务对同一个内容并发操作。
	- 持久性：事务提交，操作成功了。不能改变了。保存到数据库中了。
- 隔离问题
	- 脏读：一个事务 读到 另一个 事务 没有提交的数据。
	- 不可重复读：一个事务 读到 另一个事务 已经提交的数据。（update更新）
	- 虚读(幻读)：一个事务 读到 另一个事务 已经提交的数据。（insert插入） --理论时
- 事务隔离级别：用于解决隔离问题
	1. 读未提交：read uncommitted，一个事务读到另一个事务没有提交的数据。存放问题：3个
	2. 读已提交：read committed，一个事务读到另一个事务提交的数据。解决：脏读，存在2个问题
	3. 可重复读：repeatable read ，一个事务中读到数据重复的。及时另一个事务已经提交。解决：脏读、不可重复读，存放1个问题。
	4. 串行化，serializable，单事务，同时只能一个事务在操作。另一个事务挂起(暂停)。解决：3个问题。
- 默认隔离级别
mysql默认隔离级别：repeatable read
oracle默认隔离级别：read committed
对比：
		性能：read uncommitted > read committed > repeatable read > serializable
		安全：read uncommitted < read committed < repeatable read < serializable

### 2.2 事务操作
- mysql 命令操作
	开启事务：mysql>  start transaction;			--开启相当于关闭自动提交
	提交事务：mysql>  commit;				--全部成功
	回滚事务：mysql>  rollback;				--保证全部不成功
- jdbc java代码操作【掌握】-- JDBC中必须使用Connection连接进行事务操作。
	开启事务：conn.setAutoCommit(false);
	提交事务：conn.commit();
	回滚事务：conn.rollback();

mysql 默认事务提交的，及每执行一条sql语句就是一个事务。
扩展：oracle事务默认不提交，需要手动提交。

### 2.3 保存点
Savepoint 保存点，用于记录程序执行位置，方便可以随意回滚指定的位置。spring 事务的传播行为
AB整体（必须），CD整体（可选）

	Savepoint savepoint = null;
	try{
	   // 1 开启事务
	   conn.setAutoCommit(false);
	   A
	   B
	   // 记录保存点
	   savepoint = conn.setSavepoint();
	   C
	   D
	
	   //2 提交ABCD
	   conn.commit();
	} catch(){
	  if(savepoint != null){
	      //CD 有异常，回滚到CD之前
	      conn.rollback(savepoint);
	      // 提交AB
	      conn.commit();
	  } else {
	     //AB有异常 ,回滚到最开始处
	     conn.rollback();
	  }
	}

### 2.4 丢失更新
#### 案例
A 查询数据，username = 'jack' ,password = '1234'
B 查询数据，username="jack", password="1234"
A 更新用户名 username="rose",password='1234'    -->   username="rose",password="1234"
B 更新密码   password="9999" ,username="jack"  -->   username="jack",password='9999'
丢失更新：最后更新数据，将前面更新的数据覆盖了。

#### 解决方案：采用锁机制。
乐观锁：丢失更新肯定不会发生。

	给表中添加一个字段（标识），用于记录操作版本。
	username="jack",password="1234",version="1" ，比较版本号，如果一样，修改版本自动+1.。如果不一样，必须先查询，再更新。
悲观锁：丢失更新肯定会发生。采用数据库锁机制。

	读锁：共享锁，大家可以一起读。
		select .... from .... lock in share mode;
	写锁：排他锁，只能一个进行写，不能有其他锁（写、读），所有更新update操作丢将自动获得写锁。
		select ... from ... for update;
注意：数据库的锁，必须在事务中使用。
只要事务操作完成（commit|rollback|超时）自动释放锁

## 3. 事务实例

## 4. 连接池
1.为什么使用连接池：连接Connection 创建与销毁 比较耗时的。为了提供性能，开发连接池。
2.什么是连接池

	javaee规范规定：连接池必须实现接口，javax.sql.DataSource (数据源)
	为了获得连接 getConnection()
	连接池给调用者提供连接，当调用者使用，此链接只能供调动者是使用，其他人不能使用。
	当调用者使用完成之后，必须归还给连接池。连接必须重复使用。
3.自定义连接池
4.第三方连接池

	DBCP，apache
	C3P0 ，hibernate 整合
	tomcat 内置（JNDI）

### 4.1 DBCP
1.Apache提供的
2.导入jar包

	commons-dbcp-1.4.jar 核心包
	commons-pool-1.6.jar 依赖包
3.核心类

	public class BasicDataSource implements DataSource
4.手动调用

	//1 创建核心类
	BasicDataSource dataSource = new BasicDataSource();
	//2 配置4个基本参数
	dataSource.setDriverClassName("com.mysql.jdbc.Driver");
	dataSource.setUrl("jdbc:mysql:///day16_db");
	dataSource.setUsername("root");
	dataSource.setPassword("1234");
	//3 管理连接配置
	dataSource.setMaxActive(30);	//最大活动数
	dataSource.setMaxIdle(20);		//最大空闲数
	dataSource.setMinIdle(10);		//最小空闲数
	dataSource.setInitialSize(15);	//初始化个数

5.配置调用

	DBCP采用properties文件，key=value ，key为 BasicDataSource属性（及setter获得）
	//0 读取配置文件
	InputStream is = TestDBCP.class.getClassLoader().getResourceAsStream("dbcpconfig.properties");
	Properties properties = new Properties();
	properties.load(is);
	//1 加载配置文件，获得配置信息
	DataSource dataSource = BasicDataSourceFactory.createDataSource(properties);

6.配置文件 dbcpconfig.properties

	#连接设置
	driverClassName=com.mysql.jdbc.Driver
	url=jdbc:mysql://localhost:3306/day16_db
	username=root
	password=1234
	
	#<!-- 初始化连接 -->
	initialSize=10
	
	#最大连接数量
	maxActive=50
	
	#<!-- 最大空闲连接 -->
	maxIdle=20
	
	#<!-- 最小空闲连接 -->
	minIdle=5
	
	#<!-- 超时等待时间以毫秒为单位 6000毫秒/1000等于60秒 -->
	maxWait=60000
	
	
	#JDBC驱动建立连接时附带的连接属性属性的格式必须为这样：[属性名=property;] 
	#注意："user" 与 "password" 两个属性会被明确地传递，因此这里不需要包含他们。
	connectionProperties=useUnicode=true;characterEncoding=gbk
	
	#指定由连接池所创建的连接的自动提交（auto-commit）状态。
	defaultAutoCommit=true
	
	#driver default 指定由连接池所创建的连接的只读（read-only）状态。
	#如果没有设置该值，则“setReadOnly”方法将不被调用。（某些驱动并不支持只读模式，如：Informix）
	defaultReadOnly=
	
	#driver default 指定由连接池所创建的连接的事务级别（TransactionIsolation）。
	#可用值为下列之一：（详情可见javadoc。）NONE,READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE
	defaultTransactionIsolation=READ_UNCOMMITTED



### 4.2 C3P0
1.第三方提供,非常优秀
2.导入jar包

	c3p0-0.9.2-pre5.jar 核心包
	mchange-commons-java-0.2.3.jar 依赖包
	c3p0-oracle-thin-extras-0.9.2-pre5.jar 使用oracle的依赖
3.核心类

	ComboPooledDataSource
4.手动调用

	//1 核心类 (日志级别：debug info warn  error)
	ComboPooledDataSource dataSource = new ComboPooledDataSource();
	//2 基本4项
	dataSource.setDriverClass("com.mysql.jdbc.Driver");
	dataSource.setJdbcUrl("jdbc:mysql:///day16_db");
	dataSource.setUser("root");
	dataSource.setPassword("1234");
	//3 优化
	dataSource.setMaxPoolSize(30);		//最大连接池数
	dataSource.setMinPoolSize(10);		//最小连接池数
	dataSource.setInitialPoolSize(15);	//初始化连接池数
	dataSource.setAcquireIncrement(5);	//每一次增强个数

5.配置调用

	加载位置：WEB-INF/classes  (classpath , src)
	配置文件名称：c3p0-config.xml
	//1 c3p0...jar 将自动加载配置文件。打包后从WEB-INF/classes (src)目录加载名称为c3p0-config.xml的文件
	//ComboPooledDataSource dataSource = new ComboPooledDataSource(); //自动从配置文件 <default-config>
	ComboPooledDataSource dataSource = new ComboPooledDataSource("itheima"); //手动指定配置文件 <named-config name="itheima"> 
		
6.配置文件 c3p0-config.xml

	<c3p0-config>
		<!-- 默认配置，如果没有指定则使用这个配置 -->
		<default-config>
			<property name="driverClass">com.mysql.jdbc.Driver</property>
			<property name="jdbcUrl">jdbc:mysql:///day16_db</property>
			<property name="user">root</property>
			<property name="password">1234</property>
			
			<property name="checkoutTimeout">30000</property>
			<property name="idleConnectionTestPeriod">30</property>
			<property name="initialPoolSize">10</property>
			<property name="maxIdleTime">30</property>
			<property name="maxPoolSize">100</property>
			<property name="minPoolSize">10</property>
			<property name="maxStatements">200</property>
			<user-overrides user="test-user">
				<property name="maxPoolSize">10</property>
				<property name="minPoolSize">1</property>
				<property name="maxStatements">0</property>
			</user-overrides>
		</default-config> 
		<!-- 命名的配置 -->
		<named-config name="itheima">
			<property name="driverClass">com.mysql.jdbc.Driver</property>
			<property name="jdbcUrl">jdbc:mysql:///day16_db</property>
			<property name="user">root</property>
			<property name="password">1234</property>
	    <!-- 如果池中数据连接不够时一次增长多少个 -->
			<property name="acquireIncrement">5</property>
			<property name="initialPoolSize">20</property>
			<property name="minPoolSize">10</property>
			<property name="maxPoolSize">40</property>
			<property name="maxStatements">0</property>
			<property name="maxStatementsPerConnection">5</property>
		</named-config>
	</c3p0-config> 

### 4.3 JNDI
- tomcat管理连接池
- JNDI(Java Naming and Directory Interface,Java命名和目录接口)是SUN公司提供的一种标准的Java命名系统接口，用于存放java任意对象，给对象进行命名（使用目录结构）。例如：/a/b/c
- 作用：在多个项目之间共享数据，只需要传递名称，就可以获得对象。
- 理解：JNDI 就是一个容器，用于存放任意对象。
- tomcat 将连接池存放 JNDI容器，可以获得使用。使用前提必须通知tomcat进行存放，默认情况下tomcat没有存放。

配置:
1.给tomcat配置数据源（连接池），使用 <Context >标签

	方式1：%tomcat%/conf/server.xml -->  在<Host>标签下添加<Context>
	方式2：%tomcat%/conf/Catalina/localhost/day16.xml  ---> <Context>
		day16/META-INF/Context.xml  在项目的META-INF下创建Context.xml,会自动发布到“方法2”指定位置
	Context.xml中的内容:
		<Context>
			<!-- 
				name 存放进去名称
				auth 存放位置
				type 确定存放内容，tomcat将通过指定接口创建实例，底层使用DBCP
				其他都是DBCP属性设置
			 -->
		  <Resource name="jdbc/itheima" auth="Container" type="javax.sql.DataSource"
		               maxActive="100" maxIdle="30" maxWait="10000"
		               username="root" password="1234" driverClassName="com.mysql.jdbc.Driver"
		               url="jdbc:mysql://localhost:3306/day16_db"/>
		</Context>
2 从JNDI容器获取，在当前项目web.xml配置

	<!-- 给当前项目配置，从JNDI容器获得指定名称内容 -->
	<resource-ref>
	  <res-ref-name>jdbc/itheima</res-ref-name>
	  <res-type>javax.sql.DataSource</res-type>
	  <res-auth>Container</res-auth>
	</resource-ref>
3.使用

	//0 包名：javax.naming
	//1 初始化JNDI容器
	Context context = new InitialContext();
	//2 获得数据源   固定 “java:/comp/env”
	DataSource dataSource = (DataSource)context.lookup("java:/comp/env/jdbc/itheima");
