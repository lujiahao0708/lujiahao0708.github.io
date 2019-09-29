---
title: JDBC笔记一
date: 2016-07-04 17:28:41
categories: JavaEE
tags:
  - JavaEE
  - 数据库
  - MySql
description: JDBC笔记第一部分,主要包括:JDBC简单应用,Junit 测试,JDBC工具类,自定义异常等.
---

## 1. JDBC简单应用

	public class FirstJDBC {
	    public static void main(String[] args) throws Exception {
	        // 0.准备变量
	        String driver = "com.mysql.jdbc.Driver";// mysql驱动实现类
	        String url = "jdbc:mysql://localhost:3306/day15_db";// 确定数据库服务器地址,端口号,使用数据库
	        String user = "root";// 登录名称
	        String password = "1234";// 登录密码
	        // 1.注册驱动
	        Class.forName(driver);
	        // 2.获得链接
	        Connection conn = DriverManager.getConnection(url, user, password);
	        // 3.获得语句执行者
	        Statement st = conn.createStatement();
	        // 4.发送sql语句,查询  结果相当于一个set集合,每一个成员表示数据库表中一条记录
	        ResultSet rs = st.executeQuery("select * from t_user ");
	        // 5.处理结果
	        rs.next();// 移动到第一行
	        // getXxx获取某一行的指定列或字段值  getXxx(int 列数),getXxx(String 字段名)
	        int id = rs.getInt("id");
	        String username = rs.getString("username");
	        String userPassword = rs.getString("password");
	        System.out.printf("id:"+id+" username:"+username+" password:"+password);
	        // 6.释放资源,优先关闭最后使用的
	        rs.close();
	        st.close();
	        conn.close();
	    }
	}

## 2. Junit 测试
1.需要导入两个jar包
`junit.jar`
`hamcrest-core.jar`
放入libs目录下即可
2.类名alt+enter,然后选择需要添加测试的方法即可

测试用例方法，公共 没有返回值 非静态  方法名自定义 没有参数列表
方法名建议：test方法名()

	public class DemoTest {
		private Demo demo;
	
		@Before  //测试方法执行前
		public void myBefore(){
			System.out.println("之前");
			demo = new Demo();
			//初始化数据
		}
		@After   //测试方法执行后
		public void myAfter(){
			System.out.println("之后");
			//方法资源
		}
		@Test(timeout=1000)  //timeout 设置测试时间，如果超时性能有问题
		public void testAdd() {
			try {
				Thread.sleep(2000);
			} catch (Exception e) {
			}
			
			int sum = demo.add(1, 2);
			//断言
			Assert.assertEquals(3, sum);
			assertEquals(3, sum);  //使用静态导入的结果
		}
		@Test
		public void testMul() {
			int sum = demo.mul(1, 2);
		}
		@BeforeClass
		public static void myBeforeClass(){
			System.out.println("类之前");
			
		}
		@AfterClass
		public static void myAfterClass(){
			System.out.println("类之后");
		}
	}

## 3. JDBC工具类

	public class JdbcUtils {
	    private static String url;
	    private static String user;
	    private static String password;
	
	    // 这些配置文件的东西只用加载一次就可以了,写在静态代码块中
	    static {
	        try {
	            // 参数配置应该放在配置文件中
	            // 1. 加载properties文件
	            // 方式1:使用ClassLoader加载资源
	            //InputStream is = JdbcUtils.class.getClassLoader().getResourceAsStream("jdbcInfo.properties");
	            // 方式2:使用Class对象加载,必须加上/,表示src
	            InputStream is = JdbcUtils.class.getResourceAsStream("/jdbcInfo.properties");
	            // 2. 解析配置文件
	            Properties properties = new Properties();
	            properties.load(is);
	            // 3. 获得配置文件中的数据
	            String driver = properties.getProperty("driver");
	            url = properties.getProperty("url");
	            user = properties.getProperty("user");
	            password = properties.getProperty("password");
	            // 4. 注册驱动
	            Class.forName(driver);
	        } catch (Exception e){
	            throw new RuntimeException(e);
	        }
	    }
	
	    /**
	     * 获得连接
	     */
	    public static Connection getConnection(){
	        try {
	            Connection conn = DriverManager.getConnection(url, user, password);
	            return conn;
	        } catch (Exception e){
	            // 将编译时异常转换成运行时异常,开发中常见运行时异常
	            // throw new RuntimeException(e);
	            // 此处可以使用自定义异常
	            // 类与类之间进行数据交换时可以使用return返回数据.
	            // 也可以使用自定义异常返回值，调用者try{} catch(e){ e.getMessage() 获得需要的数据}
	            throw new MyConnectionException(e);
	        }
	    }
	
	    /**
	     * 释放资源
	     */
	    public static void closeResource(Connection conn, Statement st, ResultSet rs){
	        try {
	            if (rs != null) {
	                rs.close();
	            }
	        } catch (Exception e){
	            throw new RuntimeException(e);
	        } finally {
	            try {
	                if (st != null) {
	                    st.close();
	                }
	            } catch (Exception e){
	                throw new RuntimeException(e);
	            } finally {
	                try {
	                    if (conn != null) {
	                        conn.close();
	                    }
	                } catch (Exception e){
	                    throw new RuntimeException(e);
	                }
	            }
	        }
	    }
	}


## 4. 自定义异常
继承RuntimeException  覆写方法
将编译时异常转换成运行时异常,开发中常见运行时异常
throw new RuntimeException(e);
类与类之间进行数据交换时可以使用return返回数据.
也可以使用自定义异常返回值，调用者try{} catch(e){ e.getMessage() 获得需要的数据}

## 5. 模板代码

	public void demo1(){
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;

        try {
            conn = JdbcUtils.getConnection();
            // ....
        } catch (Exception e){
            throw new RuntimeException(e);
        } finally {
            // 释放资源
            JdbcUtils.closeResource(conn,st,rs);
        }
    }

## 6. Api详解
### 6.1 注册驱动
	1.0 所有的驱动实现类必须实现规范接口，java.sql.Driver接口
	1.1 原始编写方式：DriverManager.registerDriver(new Driver());
		特点：必须导包  import com.mysql.jdbc.Driver; 
			硬编程，内容写死的，无法扩展，不易于数据变迁(更换)
		源码：public class com.mysql.jdbc.Driver implements java.sql.Driver
	 
	1.2 建议编写方式：Class.forName("com.mysql.jdbc.Driver");
		特点：指定类如果有static{} 里面的内容将自动执行。
		源码：
			static {
				try {
					java.sql.DriverManager.registerDriver(new Driver());
				} catch (SQLException E) {
					throw new RuntimeException("Can't register driver!");
				}
			}
			mysql Driver实现类，将自己进行注册。
		优点：使用字符串方法加载到内容（注册），之后可以将驱动配置到配置文件中，之后只需要修改配置文件，数据库就更改。
	1.3 此行代码可省略，建议不要省略。
		mysql驱动 高版本，将自动加载驱动并注册。
			文件：mysql-connector-java-5.1.22-bin.jar/META-INF/services/java.sql.Driver
			内容：com.mysql.jdbc.Driver 
	1.4 错误总结
		* ClassNotFoundException ，是否导入jar包？驱动名称是否正确？
		* java.lang.NoClassDefFoundError   如果配置文件放置位置错误就会报这个异常
	1.5 mysql  com.mysql.jdbc.Driver 子类  org.gjt.mm.mysql.Driver
		源码：public class Driver extends com.mysql.jdbc.Driver

### 6.2 获得连接
	2.1 介绍
		使用连接接口：java.sql.Connection,只有获得连接才可以操作数据库。
		通过DriverManager 驱动管理者类获得连接。getConnection(url,user,password)
	2.2 url 确定访问数据库位置
		格式#   协议:子协议:子名称
			协议  固定值 -->  jdbc
			子协议  --> 确定数据，例如：mysql、oracle 等
			子名称  --> localhost主机，3306端口，day15_db数据库名称
			参数  --> jdbc:mysql://localhost:3306/day15_db?useUnicode=true&characterEncoding=UTF-8
				设置请求编码，安装如果指定UTF-8，不需要设置的。如果安装时设置的编码为ISO8859-1编码，中文乱码需要如上处理。
		主机默认localhost，端口默认3306
			简写方式 -->  jdbc:mysql:///day15_db
	2.3 错误总结
		* Unknown database 'day15_db2' , 表示数据库不存在。
		* Access denied for user 'root'@'localhost' (using password: YES)  ， 账号和密码不匹配

### 6.3 Connection提供的不同的操作对象

	3.1 操作对象
			* Statement ， 语句执行者，获得方式 conn.createStatement()【】
			* PreparedStatement , 预处理对象，获得方法  conn.prepareStatement(sql)【】
			* 必须先提供sql语句，进行预先处理。
			* CallableStatement 存储过程，  prepareCall(String sql)，将sql语句编写数据库，相当于数据库函数，执行时	只需要传递实际参数
	3.2 Statement
		int executeUpdate(sql) 执行DDL/DML语句，返回影响的行数。【】
		ResultSet  executeQuery(sql) 执行DQL语句，返回结果集（相当于一个表）【】
		boolean execute(sql) 
			返回值true，表示执行DQL语句，必须通过  getResultSet() 获得结果集
			返回值false，表示执行DML/DDL语句，必须通过  getUpdateCount() 获得影响行数 
	3.3 滚动结果集 （了解）
		* jdbc规范 默认结果集forward ，只能向前的结果集。
		* 滚动结果集：可以向前，也可以向后。mysql 直接支持滚动
		* ResultSet api
			next() 下一个（向前）
			previous() 上一个（向后）
		* 结果集ResultSet要滚动，前提Statement必须设置支持的。 
		* 使用方法：createStatement(int resultSetType, int resultSetConcurrency)
			resultSetType - 结果集类型，
					ResultSet.TYPE_FORWARD_ONLY，仅仅只能向前
					ResultSet.TYPE_SCROLL_INSENSITIVE，可以滚动，不敏感。滚动中，数据库发生改变，结果集内容不改变的。
						数据库的数据是否同步到结果集ResultSet中。
					ResultSet.TYPE_SCROLL_SENSITIVE，可以滚动，敏感。滚动中，数据库数据发生改变，结果集内容改变。
			resultSetConcurrency - 并发类型
				ResultSet.CONCUR_READ_ONLY ，结果集只能读，不能改。
				ResultSet.CONCUR_UPDATABLE，结果集可以更新，数据一并更改。
					结果集数据同步到数据库
		* 操作
			获得  getXxx(int)获得指定列号的内容，  getXxx(String)获得指定列名(字段)的内容
			例如：getString(4) 获得第4列，  getString("username")  获得字段名称username的值

## 7.SQL注入
sql注入 ：用户输入实际参数，作为了sql语句语法的一部分，数据库编译在执行时生效的。
	select * from t_user where username = 'jack' or 1=1 or 1='' and password = '12345'
	select * from t_user where username = 'jack' --' and password = '12345'
解决方案：
	1. 手动方式：\转义单引号
	2. 使用预处理对象，防止sql注入
		1) 先提供sql语句，将实际参数使用占位符?替换。
			例如：select * from t_user where username = ? and password = ?
		2) 获得预处理对象，获得对象时必须提供sql语句，让sql语句预先进行编译。
		3) 设置实际参数
			PreparedStatement 提供 setXxx(int , Object)
				参数1：int表示 ?位置，从1开始。
				参数2：Object具体类型，如果字符串String等。
	3. PreparedStatement 接口 是 Statement接口的子接口。
		但是execute(sql) 父类方法，子类使用抛异常。使用的没有参数。

## 8.预处理对象PreparedStatement
	特点：
		sql 易于编写，更佳清晰
		提高性能，编译一次，可以执行多次。
	编写步骤：
		1.提供sql语句，并将实际参数使用？占位符
		2.获得预处理对象，注意提供sql语句
		3.设置实际参数，将?替换回来
		4.执行，注意：不能设置sql语句。
	Statement  和 PreparedStatement  对比：
		一般情况使用PreparedStatement，之后学习框架DbUtils底层使用PreparedStatement，hibernate底层使用也是。
		如果使用Statement，必须保证sql都是自己编写，实际参数都是自己传递的。
	PreparedStatement 应用场景
		1.防sql注入
		2.大数据
			大数据类型：blob 字节 、text 字符
		3.批处理



