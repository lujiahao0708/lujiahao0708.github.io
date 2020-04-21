---
title: 10-JavaWeb基础-JSP笔记
categories: JavaEE
tags:
  - JavaEE
  - 后台
  - JSP
abbrlink: fd25bde8
date: 2016-12-05 17:52:14
---

## 什么是JSP
- tomcat安装目录下的`work`目录就是tomcat处理jsp的工作目录
- JSP是以Java语言为基础的动态网页开发技术(Servlet也是动态网页开发技术)
- 二者特点
	- servlet特点:在java代码中嵌入html代码
	- jsp特点:在html代码中嵌入java代码

<!--more-->

## JSP本质上就是servlet
1. tomcat获得jsp文件后,将jsp转化成servlet,会产生`xxx_jsp.java`(servlet源码)
		
		D:\java\tomcat\apache-tomcat-7.0.53\work\Catalina\localhost\day10\org\apache\jsp\b_005fdemo_jsp.java
				tomcat安装目录				     引擎	      主机	   项目   固定包名		jsp文件对应servle

2. tomcat将java文件编译产生class文件
3. tomcat运行class文件,并将结果输出到浏览器

源码分析:
	
	public final class xxx_jsp extends org.apache.jasper.runtime.HttpJspBase
	public abstract class HttpJspBase extends HttpServlet

> 注意:jsp生成java代码,默认是第一次生成,之后直接执行,除非内容修改

## JSP脚本元素
- 表达式:`<%= 表达式 %>`
	- 将表达式结果输出到浏览器,底层解析使用`out.print(表达式);`
- 代码片段:`<% java代码 %>`
	- 将"java代码"完整复制到service方法体中
- 脚本声明:`<%! 声明内容 %>`
	- 将"声明内容"完整复制到class类中(成员变量or成员方法)

> 注意:脚本不能嵌套

## JSP注释
JSP注释将jsp源码内容注释掉,tomcat生成java源码中将没有注释掉的内容

	<%-- 注释内容  --%>
## JSP指令
	配置给tomcat,tomcat将jsp生成java源码(servlet)依据.
	格式:`<%@ 指令名称 属性=值 属性=值 ...%>`
	指令包括:page/include/taglib
### page指令

1. 编码[*]
	
		pageEncoding:当前页面的编码
		contentType:jsp生成servlet响应给浏览器编码
		注意:一般一致
		对比:
			如果只有pageEncoding,设置当前页面编码,也可以设置响应编码
			如果只有contentType,可以设置响应编码,也可以设置当前页面编码
2. jsp缓存机制
	
		buffer:设置缓存大小,默认8Kb
		autoFlash:缓存如果溢出,将自动刷新
			如果设置为false可能异常:java.io.IOException:Error:JSP Buffer overflow
3. jsp错误处理机制
	
		errorPage:设置错误页面. 当前jsp页面出现异常,将显示错误界面(友好界面).
		isErrorPage:指定当前页面发生错误时的错误处理页,可以捕获异常信息.
		思考：当页面过多时如何配置错误页面? 
			使用统一管理，给整个web项目配置友好页面。web.xml
			Element : error-page  需要使用<error-page>标签指定
			Content Model : ((error-code | exception-type), location)
				<error-code> 错误码（状态码）：500、404等
				<exception-type> java异常类型：java.lang.RuntimeException
				<location> 出现指定错误时，指定的显示页面
4. 常用
	
		session:表示当前jsp页面是否可以使用session内置对象.
				true:可以在jsp脚本(表达式/代码块)中使用session
		import:jsp使用其他类，进行导包。【】
			分别导入：java.util.List
			一次性导入：java.util.List,java.util.ArrayList
			星号：java.util.*
		language :表示jsp支持嵌入语言(java)
		info : 使用在servlet接口第5个方法，getServletInfo()

5. 指令的使用注意
	
		一个指令可以编写多个属性
		一个指令可以多次使用
		指令可以使用在jsp页面任何位置
		大部分指令只能使用一次,但是`import`可以使用多次.

### include指令

格式:`<%@include file=""%>`

分类:

![QQ截图20160622105838.png](https://ooo.0o0.ooo/2016/06/21/576a00c1b0513.png)

### taglib指令

具体请参看EL&JSTL笔记.


## JSP九大内置对象[***]
可以在jsp表达式脚本`<%= &>`和代码片段脚本`<% %>`可以使用的变量.都在`service()`方法中,及`service()`方法提供的变量.

- page -----> 表示当前页面.就是this引用
- config -----> 表示servlet配置.类型是:ServletConfig
- application -----> 表示web应用的上下文.类型是:ServeltContext
- request -----> 表示一次请求.类型:HttpServletRequest
- response -----> 表示一次响应.类型:HttpServletResponse
- session -----> 表示一次会话.类型:HttpSession
- out -----> 表示输出响应体.类型:JspWriter
- exception -----> 表示异常.类型:Throwable
- pageContext -----> 表示jsp页面的上下文(jsp管理者).类型:PageContext

### JSP四个作用域

- page -----> 当前页面(一个页面)
- request -----> 一次请求(默认就一个页面,如果使用请求转发可以多个页面)
- session -----> 一次会话(可以有多次请求)
- application -----> 一个应用(可以多次会话)

### JSP输出
- 输出类型:JspWriter
- jsp输出底层使用的是`response.getWriter()`

		private void initOut() throws IOException {
			if(out == null){
				out = response.getWriter();
			}
		}
- 下面代码输出情况:
		
		<%
			out.print("aaa");
			response.getWriter().print("bbb");
			out.print("ccc");
		%>
		输出结果:bbbaaaccc

		原理解析:jsp和servlet各自都有自己的缓存,jsp首先写入到jsp自己的缓存中(JspWriter),
		然后再刷新到servlet缓存(response.getWriter();)
	![新建位图图像.jpg](https://ooo.0o0.ooo/2016/06/22/576a357f616e9.jpg)

> 小知识点:
> 如果在jsp片段中使用`response.getOutputStream().print("abc");`的话会报错`java.lang.IllegalStateException`
> 原因是因为jsp底层使用了`getWriter()`,不能和`getOutputStream()`混合使用.


## JSP动作标签
JSP 内置标签，可以取代 jsp脚本。

格式： `<jsp:标签名 />`

	<jsp:include page="" />  动态包含
	<jsp:forward />  转发
		<jsp:param/>  处理请求参数的 ， 可以将中文内容进行 URL编码，
	                       类似  <form enctype="application/x-www-form-urlencoded">
		
		<jsp:include page="/error.jsp">
			<jsp:param value="屌丝" name="username"/>
		</jsp:include>
	
		<jsp:useBean id="user" class="com.itheima.User"></jsp:useBean>  
		<%-- User user = new User();  pageContext.setAttribute("user",user)--%>
		
		<jsp:setProperty property="username" name="user" value="jack"/>
		<%--  user.setUsername("jack") --%>
		
		<jsp:getProperty property="username" name="user"/>
		<%-- user.getUsername() --%>

## JavaBean属性
	public class User{
		private String id2;// 字段(Field)
		public String getId(){
			return id2;
		}
		public void setId(String id){
			this.id2 = id;
		}
		// javabean属性(property) --> 由setter或getter方法获得  --> setId --> Id  --> id
		// 这个bean的属性叫id
	}



## 用户登录案例

	// 登录处理
    if (userLogin != null) {
        // 登录成功   session保存用户信息 重定向到成功界面
        request.getSession().setAttribute("userLogin",userLogin);
        // 重定向
        String url = request.getContextPath() + "/login_success.jsp";
        response.sendRedirect(url);
    } else {
        // 登录失败   request设置错误,请求转发,在登录页面显示提示信息
        // request作用域保存错误信息
        request.setAttribute("msg","用户名和密码不匹配");
        request.setAttribute("username",username);
        // 使用请求转发(一次请求涉及到多个页面)
        request.getRequestDispatcher("/login.jsp").forward(request,response);
    }