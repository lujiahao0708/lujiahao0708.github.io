---
title: 11-JavaWeb基础-EL-JSTL
date: 2016-12-12 12:37:52
categories: JavaEE
tags:
  - JavaEE
  - 后台
  - EL
  - JSTL

---

## EL表达式

### EL介绍

EL（Expression Language） 目的：为了使JSP写起来更加简单。表达式语言的灵感来自于 ECMAScript 和 XPath 表达式语言，它提供了在 JSP 中简化表达式的方法，让Jsp的代码更加简化。

<!--more-->

### EL基本语法

- el格式： ${  表达式 }
- 作用：
	
		获得自定义数据：自定义数据必须在作用域中
		执行运算
		获得servlet的api
		调用java方法

- el内置对象：11个
	
		4个作用域：pageScope、requestScope、sessionScope、applicationScope
		pageContext ，表示的就是jsp PageContext对象
		
		param 一个请求参数  ${param.username}   request.getParameter("username");
		paramValues 一组 ${paramValues.loves}    request.getParameterValues("loves");
	
		header   一个头  ${header.referer}  request.getHeader("referer");
		headerValues 一组头  ${header.cookie}   request.getHeaders("cookie");
	
		cookie 获得cookie对象
	
		initParam web项目初始化参数， servletContext.getInitParameter("xxx");

- 语法列表

	代码详细 : [https://github.com/chiahaolu/JavaEEDemo/blob/master/day11_el_jstl_ums/web/el_base.jsp](https://github.com/chiahaolu/JavaEEDemo/blob/master/day11_el_jstl_ums/web/el_base.jsp)

## EL函数

### 在JSP页面中引用系统函数库
> <font color='red'>`<%@taglib uri="http://java.sun.com/jsp/jstl/functions" prefix="fn" %>`</font>

代码 : [https://github.com/chiahaolu/JavaEEDemo/blob/master/day11_el_jstl_ums/web/el_function.jsp](https://github.com/chiahaolu/JavaEEDemo/blob/master/day11_el_jstl_ums/web/el_function.jsp)

### 自定义EL函数

#### 1.确定实现类

		public class MyFunctions {//如果大于指定长度，截取并显示分隔符
			public static String mysub(String str,int length , String split){
				if(str == null){
					return "";
				}
				if(str.length() <= length){
					return str;
				}
				return str.substring(0, length) + split;
			}
		}
#### 2.编写配置文件(WEB-INF目录下编写myfn.tld)

		<?xml version="1.0" encoding="UTF-8" ?>
		<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
		  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd"
		  version="2.0">
		    
		  <tlib-version>1.0</tlib-version>
		  <short-name>myfn</short-name>
		  <uri>http://www.itheima.com/lt/myfn</uri>
		  
		  <!-- 定义函数 -->
		  <function>
		  	<name>sub</name><!-- 访问的名称 -->
		  	<!-- 实现类 -->
		  	<function-class>com.itheima.b_fn.MyFunctions</function-class>
		  	<!-- 函数签名 
		  			String mysub(String str,int length , String split)
		  			签名格式： 返回值  方法名(参数列表)-->
		  	<function-signature>java.lang.String mysub(java.lang.String,int,java.lang.String)</function-signature>
		  </function>
		</taglib>
#### 3.jsp页面中引用自定义的函数库

		<%@taglib uri="http://www.itheima.com/lt/myfn" prefix="myfn" %>

## JSTL

### JSTL基本介绍

>JavaServer Pages Standard Tag Library(JSP标准标签库)，是jsp规范一部分。
>sun定义规范及接口,apache 对规范进行实现。

- 导入jar包

		myeclipse在发布项目到tomcat时，自动添加jstl.jar包.
		eclipse需要手动导入jar包.
		IntellJ将`jstl.jar`和`standard.jar`导入到WEB-INF目录下的lib包下.

- 使用格式：<前缀:标签名  属性=值  />
- 标签库分类:

	![QQ截图20160623172734.png](https://ooo.0o0.ooo/2016/06/23/576bacee58638.png)

- 导入方式:
	
	`<%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>`

### 标签介绍

#### 核心库标签

- out标签

		将制定内容输出到浏览器,默认支持EL表达式.
		主要用途:将html源码进行转义.
		escapeXml:表示是否转义,默认值:true
	
		<c:out value="abc"></c:out><br/>
		<c:out value="${empty user }"></c:out><br/>
		<c:out value="<a>你敢点我</a>" escapeXml="false"></c:out><br/>

- set标签

		给指定的作用域设置内容
			value 设置值
			var 属性名称
			scope 作用域 (page/request/session/application 这里的名字和el表达式的不一样)
		相当于 : pageContext.setAttribute(var,value,scope)

		<c:set value="request_屌" var="dzd" scope="request" ></c:set>
		<c:remove var="dzd" scope="page"/> <%--移除作用域内容 如果没写scope就依次移除 --%>
		${dzd}  <%--依次从四个作用域中获取 page、request、session、application --%> <br/>
		${requestScope.dzd } <br/>

- if标签

		if(test){
			标签体
		}

		登录的案例改进:
	
		<c:set var="loginUser" value="xxx" scope="session"></c:set>
		<c:if test="${not empty sessionScope.loginUser }">
			欢迎，xxx
		</c:if>
		<c:if test="${empty sessionScope.loginUser }">
			<a href="">登录</a>
		</c:if>

- choos标签

		<c:choose >相当于 switch
		<c:when> 相当于 case
		<c:otherwise> 相当于 default

- forTokens标签(不常用)

		将自定字符串，安装指定符号进行分割，并遍历。
		<c:forTokens items="www.itheima.com" delims="." var="s">
			${s} , <br/>
		</c:forTokens>

- catch标签(不常用)

		<c:catch>  等效 try{}catch{}

- url标签(常用)

		<%-- <c:url valur="" var="" scope=""> value设置url，如果/开头，自动添加项目名。
			如果没有设置var，将直接输出到浏览器，如果设置var，将保存到指定(scope)的作用域，默认page
		--%>
		<a href="${pageContext.request.contextPath}/c_core3.jsp">当前页面</a> <br/>
		<a href="<c:url value='/c_core3.jsp'/>">当前页面</a> <br/>
		
		<c:url value='/c_core3.jsp' var="baseUrl"  >
			<c:param name="username" value="凤儿"></c:param> <%--自动进行url编码 --%>
		</c:url>
		<a href="${baseUrl}">当前页面</a> <br/>

- redirect标签
		
		<%--重定向 ，url设置重定向位置
			如果/开头，不需要项目名称
			<c:redirect url="/index.jsp"></c:redirect>
		--%>

- forEach标签
		items 用于设置需要遍历的内容
				如果字符串，将直接输出
				如果数组和List，遍历每一项
				如果Map，遍历每一项Map.Entry
		begin/end 这和items只能存在一组
		var 用于存放遍历每一项内容,存放page作用域，只能在循环体中使用。

#### fmt标签

	<%--格式化 时间 --%>
	<%
		pageContext.setAttribute("date", new Date());
		//new SimpleDateFormat()
	%>
	<fmt:formatDate value="${date}" pattern="yyyy-MM-dd hh:mm:ss:SSS"/> <br/>
	<fmt:formatNumber value="3.1415926" pattern="#.##"></fmt:formatNumber> <br/>
	<fmt:formatNumber value="3" pattern="#.##"></fmt:formatNumber> <br/>
	<fmt:formatNumber value="3" pattern="0.00"></fmt:formatNumber> <br/>

### 自定义标签

#### 1.编写实现类
	
	要求：自定义的类需要实现接口`SimpleTag`或者继承类`SimpleTagSupport`

	传统标签：实现接口Tag，继承类TagSupport，功能更强大，但实现繁琐。
	简单标签：接口SimpleTag，继承SimpleTagSupport ，在jsp2.0之后提供
	提供标记接口父类：JspTag

	一般实现类是以Tag结尾的,表明它是一个标签类  例如:MyDateTag

	public class MyContentTag extends SimpleTagSupport {
		
		private boolean show;
		public void setShow(boolean show) {
			this.show = show;
		}
	
		@Override
		public void doTag() throws JspException, IOException {
			if(show){
				this.getJspBody().invoke(null);//执行标签体，并输出到浏览器
			}
			//throw new SkipPageException();//当前标签之后的内容将不再执行
		}
	}

#### 2.编写tld配置文件

	tld是taglib description 的缩写 ， 标签类库的描述文件，是jsp解析程序可以通过配置文件获得相应的实现类
	tld文件基于xml文件，扩展名为tld，内容xml
	tld位置：
		1.WEB-INF目录，及子目录，但 classes、lib目录除外
		2.WEB-INF/lib/*.jar/META-INF/目录下，jar文件内

	<?xml version="1.0" encoding="UTF-8" ?>
	<taglib xmlns="http://java.sun.com/xml/ns/javaee"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-jsptaglibrary_2_1.xsd"
	    version="2.1">
	  <!-- 标签库的版本 -->
	  <tlib-version>1.0</tlib-version>
	  <!-- 建议使用前缀名称 -->
	  <short-name>my</short-name>
	  <!-- 引用标签库时，使用名称，建议使用：url作为名称，可以任意 -->
	  <uri>http://www.itheima.com/lt/mytag</uri>
	  <!-- 配置标签 -->
	  <tag>
		<!-- 标签访问名称 -->
		<name>date</name>
		<!-- 标签的实现类 -->
		<tag-class>com.itheima.e_tag.MyDateTag</tag-class>
		<!-- 标签体内容设置 -->
		<body-content>empty</body-content>
		<!-- empty 没有标签体 -->
	  </tag>
	   <tag>
	   	<name>date2</name>
	   	<tag-class>com.itheima.e_tag.MyDateTag2</tag-class>
	   	<body-content>empty</body-content>
	  	<!-- <attribute> : 用于设置属性，确定实现类执行的setter方法
	  			<name> ： 属性名称-->
	   	<attribute>
	   		<name>pattern</name>
	   	</attribute>
	   </tag>
	   <tag>
	   	<name>content</name>
	   	<tag-class>com.itheima.e_tag.MyContentTag</tag-class>
	   	<body-content>scriptless</body-content>
		<!-- scriptless：不支持jsp脚本<%  %>,支持el函数和其他标签 -->
	   	<attribute>
	   		<name>show</name>
			<!-- 用于配置标签是否支持运行时表达式的。及el表达式 -->
	   		<rtexprvalue>true</rtexprvalue>
	   	</attribute>
	   </tag>
	</taglib>

#### 3.在jsp中引用配置文件,使用标签

	<%@taglib uri="http://www.itheima.com/lt/mytag" prefix="my" %>

### 简单标签的生命周期
1. 创建标签实例
2. 设置PageContext，执行setJspContext()
3. 每一个属性setter方法，执行 setPattern()
4. 如果有标签体，执行setJspBody()
5. 执行doTag()

执行标签体
	this.getJspBody().invoke(null) ;  输出到浏览器。

在doTag()方法体中，throw SkipPageException() 异常 可以阻止当前标签之后内容的输出。
