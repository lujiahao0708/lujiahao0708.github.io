---
title: 08.JavaWeb基础-Response-Request
tags:
  - JavaWeb基础
  - Response
  - Request
abbrlink: 81aadbc5
date: 2016-11-21 11:22:52
---
# 1.Response
## 1.1 响应行

	setStatus(int status);// 设置状态码
## 1.2 响应头

	setHeader(String name, String value) 设置指定的头，一般常用。
		例如：setHeader("location","http://www.itheima.com");
	setDateHeader(String name, long date) 设置时间的
		这里使用long的原因是因为 : new Date().getTime()的返回值是long
	setIntHeader(String name, int value) 设置整型
		例如：setIntHeader("expires", 1000) 设置过期时间的(缓存页面时间)。0表示不缓存现代浏览器不好使
<!--more-->
拓展 : <font color=red><b>重定向</b></font>

	方式1：
		使用302状态码和location请求头来实现
		response.setStatus(302);
		response.setHeader("location", "1.html");
		response.setHeader("location", "/day08/1.html");
	方式2:
		response.sendRedirect("1.html");
	备注:
		使用refresh跳转也能实现和重定向相同的效果
		两次访问的状态码都是200 （有可能第二个304 读取浏览器缓存）
		格式：秒   --> 指定秒数刷新当前页面
			response.setHeader("refresh", "2");
		格式：秒;url=""  --> 指定秒之后跳转到指定的url
			response.setHeader("refresh", "0;url=1.html");

## 1.3 响应体
Response提供`字符流`和`字节流`.
一般情况，如果是流，必须关闭流。但servlet中如果使用可以不关闭
如果没有关闭流，tomcat在请求结束时，将自动的关闭
注意：<font color=red>response 的两个流不能同时使用</font>

### 1.3.1 字节流

	ServletOutputStream out = response.getOutputStream();
	out.println("1234");
	//out.print("哈哈");  //异常：不能直接发送中文
	out.write("哈哈".getBytes("UTF-8")); //有可能乱码 -- tomcat发送的字节，tomcat本身没有处理，是否乱码取决于浏览器查看方式编码

发送响应体，即发送html源码
如果println()将html源码进行回车换行，如果希望页面的显示要换行`<br/>`

tomcat默认编码使用：ISO-8859-1 （英文）

### 1.3.2 字符流
如果发送的字符，tomcat将处理字符，默认使用ISO-8859-1编码处理请求体内容。

	2.1 通知tomcat发送的数据编码  -- 注意必须在使用流之前
			response.setCharacterEncoding("UTF-8");
			注意：只通知tomcat，但不控制浏览器行为
	2.2 通过tomcat和浏览器 --强制要求浏览器查看编码
			方式1：手动方案
				response.setHeader("content-type", "text/html;charset=UTF-8");
			方式2：使用api【】
				response.setContentType("text/html;charset=UTF-8");
	2.3 通知tomcat，在使用html<meta>通知浏览器 (html源码)
			注意：<meta>建议浏览器应该使用编码，不能强制要求。

### 1.3.3 response输出缓存
response中使用流向浏览器发送数据，首先数据将写入`数据流的缓存中`，但`缓存大小8k`时，将自动刷新到浏览器。

	isCommitted()  缓存是否刷新过
	flushBuffer() 手动的刷新 输出缓存
	getBufferSize() 获得大小
	resetBuffer() 清空缓存

	// 输出缓存8kb
	System.out.println(response.isCommitted()); //输出false表示缓存没有提交
	for(int i = 0 ; i < 1024 * 8 ; i ++){
		response.getOutputStream().print("a");
	}
	System.out.println(response.isCommitted()); //输出false表示缓存没有提交 
	// 但是此时缓存已经满了
	
	response.getOutputStream().print("a");  
	System.out.println(response.isCommitted());//输出true表示缓存溢出了,提交了
	
	//手动刷新
	System.out.println(response.isCommitted());
	response.getOutputStream().print("a");
	//response.flushBuffer();  			//刷新response 输出缓存
	//response.getOutputStream().flush();	//刷新使用的流缓存,也能达到刷新缓存的效果
	response.getOutputStream().close();
	System.out.println(response.isCommitted());

# 2.Request
## 2.1 相关API

	实际访问路径   http://localhost:8080/day08/Demo05Servlet?username=jack&password=1234
		
	//1 统一资源标记符  。例如： /day08/Demo05Servlet
	String uri = request.getRequestURI();

	//2统一资源定位符：例如：http://localhost:8080/day08/Demo05Servlet
	StringBuffer url = request.getRequestURL();

	//3 协议和版本 ，  例如： HTTP/1.1
	String protocol = request.getProtocol();

	//协议，例如： http
	String scheme = request.getScheme();

	//4 主机（域名） , 例如：localhost ，如果使用ip地址，就显示ip
	String serverName = request.getServerName();

	//5 端口 , 例如：8080
	int port = request.getServerPort();

	//6 发布到tomcat下的项目名称 【】
	String contextPath = request.getContextPath();

	//7 servlet路径 , 例如：/Demo05Servlet【】
	String servletPath = request.getServletPath();

	//8 所有请求参数，及？之后所有 ,例如： username=jack&password=1234
	String queryString = request.getQueryString();

	//9远程主机的ip地址【】
	String remoteAddr = request.getRemoteAddr();

	jsp中常写的这段代码
	String path = request.getContextPath();
	String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + path + "/";

## 2.2 Request获取参数

	// 获取单个参数
	String username = request.getParameter("username");
	// 获取一组参数
	String[] loves = request.getParameterValues("loves");
	// 获取所有的参数
	Map<String, String[]> allDataMap = request.getParameterMap();
## 2.3 乱码处理

	public class Demo07Servlet extends HttpServlet {
	
		public void doGet(HttpServletRequest request, HttpServletResponse response)
				throws ServletException, IOException {
			//get请求
			
			//1 获得内容
			String username =request.getParameter("username");
			String userpwd = request.getParameter("userpwd");
			
			//2 get请求乱码处理
			username = new String(username.getBytes("ISO-8859-1"),"UTF-8");
			userpwd = new String(userpwd.getBytes("ISO-8859-1"),"UTF-8");
			
			
			System.out.println(username);
			System.out.println(userpwd);
		}
	
		public void doPost(HttpServletRequest request, HttpServletResponse response)
				throws ServletException, IOException {
			//0处理post请求 乱码
			request.setCharacterEncoding("UTF-8");
			
			//1 获得内容
			String username =request.getParameter("username");
			System.out.println(username);
			String userpwd = request.getParameter("userpwd");
			System.out.println(userpwd);
		}
	
	}
这里的乱码处理方式还是非常小儿科的,后面会学习Filter来全局统一处理GET和POST乱码了
## 2.4 RequestDispatcher 请求调度对象
### 2.4.1 请求转发
	forward(ServletRequest request, ServletResponse response) 
		A 转发 B，只输出B内容到浏览器。（如果A没有数据发送 response.isComitted = false，将清空缓存）
		请求转发只输出最后一个servlet内容
		如果 isCommitted = true ，在进行forward将抛异常。一般情况如果输出少量的数据，认为isCommitted=false
### 2.4.2 请求包含
	include(ServletRequest request, ServletResponse response)  
		A 包含 B，先输出A内容到浏览器，在输出B的内容到浏览器。
		请求包含，输出所有servlet 汇总后的内容。
## 2.4 请求转发和重定向对比
浏览器发送请求，可能涉及多个页面。
示意图 : 

![请求转发和重定向.png](https://ooo.0o0.ooo/2016/11/27/583a642bf1b82.png)

- 请求次数

		转发：1次
		重定向：2次
- 浏览器地址栏是否改变

		转发：不改变
		重定向：改变
- request作用域数据是否共享

		转发：共享（两个request对象，但数据共享的（克隆））
		重定向：不共享（创建两个新的request对象，且没有关系的）
- api使用

		转发：request.getRequestDispatcher("....").forward(request,response);  -- 让当次请求继续延续下去。
		重定向：请求通过浏览器改变方法。response.sendRedirect("location")
- 使用范围

		转发：只能在当前web项目中转发，不能转发到另一个web项目中，/表示web项目根。
			例如：D:\java\tomcat\apache-tomcat-7.0.53\webapps\day08
				request.getRequestDispatcher("/a/b/oneServlet").forward()
						http://localhost:8080/day08/a/b/oneServlet
		重定向：改变方向任意。如果重定向到当前tomcat下，/表示web站点
			例如：D:\java\tomcat\apache-tomcat-7.0.53\webapps\
				response.sendRedirect("/a/b/oneServlet");
						http://localhost:8080/a/b/oneServlet
				这里如果想要达到和上面的一样的访问路径就需要加上项目名称`/day08/a/b/oneServlet`

# 3.总结
	1.web项目根：(context root)
		开发：G:\Workspaces\hei16\day08\WebRoot\
		运行：D:\java\tomcat\apache-tomcat-7.0.53\webapps\day08\
	2. web站点根：
		运行：D:\java\tomcat\apache-tomcat-7.0.53\webapps\
	3.编码
		页面（html、jsp）、servlet 请求和响应 编码 必须保持一致。
		一般情况下：servlet开始2段内容
		request.setCharacterEncoding("UTF-8");
		response.setContentType("text/html;charset=UTF-8");
	4.请求转发特点
		* 请求路径没有改变，但可以涉及服务器端多个资源（你和媳妇）
		* 可以在一次请求中，共享request作用域的数据。
		* 一次请求，使用请求转发，tomcat将创建两个request，和一个response对象
			两个request对象数据相同的（可以理解成对象被克隆了）
