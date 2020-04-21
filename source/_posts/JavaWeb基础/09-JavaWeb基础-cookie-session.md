---
title: 09.JavaWeb基础-cookie-session
tags:
  - JavaWeb基础
  - Cookie
  - Session
abbrlink: 896fbfe9
date: 2016-11-28 17:28:17
---

## 1.会话技术

> 会话可简单理解为：用户开一个浏览器，点击多个超链接（多次请求），访问服务器多个web资源，然后关闭浏览器，整个过程称之为一个会话。

会话技术：

	cookie：浏览器端会话技术
	session：服务器端会话技术
	目的：在一次会话中（多次请求）共享数据。
![aaa.png](https://ooo.0o0.ooo/2016/11/28/583bfb14174c1.png)	 

3	cookie技术
	cookie技术不局限java，其他语言也支持。例如：php、javascript等
	JavaEE 规范提供工具类对Cookie进行操作。javax.servlet.http.Cookie类
	cookie 是什么
	1. servlet创建cookie，保存少量数据，发送浏览器。
	2. 浏览器获得服务器发送的cookie数据，将自动的保存到浏览器端。
	3. 下次访问时，浏览器将自动携带cookie数据发送给服务器。
	cookie操作
	1.创建cookie：new Cookie(name,value)
	2.发送cookie到浏览器：HttpServletResponse.addCookie(Cookie)
	3.servlet接收cookie：HttpServletRequest.getCookies()  浏览器发送的所有cookie
	cookie特点
	每一个cookie文件大小：4kb ， 如果超过4kb浏览器不识别
	一个web站点（web项目）：发送20个
	一个浏览器保存总大小：300个
	cookie 不安全，可能泄露用户信息。浏览器支持禁用cookie操作。
	默认情况生命周期：与浏览器会话一样，当浏览器关闭时cookie销毁的。---临时cookie

	cookie api
	getName() 获得名称
	getValue() 获得值

	setValue(java.lang.String newValue)  设置内容
	setMaxAge(int expiry) 设置有效时间【】
	setPath(java.lang.String uri)  设置路径【】
	setDomain(java.lang.String pattern) 设置域名 , 一般无效，有浏览器自动设置，setDomain(".itheima.com")
		www.itheima.com / bbs.itheima.com 都可以访问
		a.b.itheima.com无法访问
	isHttpOnly()  是否只是http协议使用。只能servlet的通过getCookies()获得，javascript不能获得。

	setComment(java.lang.String purpose) (了解)
	setSecure(boolean flag) (了解)
	setVersion(int v) (了解)

3.1	路径
	cookie默认路径：当前访问的servlet 父路径
	例如：http://localhost:8080/day09/a/b/c/SendCookieServlet
	默认路径： /day09/a/b/c/
	通过 setPath 修改cookie访问路径，一般使用：setPath("/")  --web站点的根（没有项目名）
		设置路径与servlet访问无关。servlet 去获得cookie有关。
	如果编写一个servlet 获得cookie  GetCookiesServlet --> getCookies()
	http://localhost:8080/day09/a/b/c/SendCookieServlet1  --》/day09/a/b/c/
	http://localhost:8080/day09/a/b/SendCookieServlet2   --》/day09/a/b/
	http://localhost:8080/day09/a/SendCookieServlet3	--》/day09/a/
	http://localhost:8080/day09/d/SendCookieServlet4	--》/day09/d

	如果此获得cookie 访问路径
		http://localhost:8080/day09/a/b/c/ ，使用项目之后内容"/day09/a/b/c/".startWith(....)
				访问的servlet路径，必须与cookie本地设置路径 判断。
		通过getCookies()获得 cookie1/cookie2/cookie3
	通常使用setPath("/") 其他servlet "/day09".startWith("/") 肯定都是/开头。所以可以访问
		通途：保证在tomcat下所有的web项目可以共享相同的cookie	
		例如：tieba , wenku , beike 多个项目共享数据。例如用户名。
	当前项目访问：setPath("/day09/")
3.2	有效时间
	默认情况：cookie临时，当浏览器关闭销毁的。
	setMaxAge 可以修改cookie被浏览器保存的时间。浏览器将cookie信息将保存cookie文件，浏览器关闭后文件仍然存在，直到设置时间过期将被浏览器自动删除。单位：秒  -- 持久化cookie（保存文件）
	删除cookie：保证域名、路径和cookie名称一致情况下，设置 setMaxAge(0) 将删除。

	cookie唯一标识：域名、路径、名称
			localhost        /      demo03_cookie_key
			baidu.com		/      mk
			localhost		/day09/a/		cookie_key
3.3	发送中文数据
	cookie 使用http协议请求头和响应头，http协议不支持中文，cookie本身不支持中文的。
	如果cookie value设置中文，服务器将抛异常。
	如果cookie需要写入中文，必须手动编码（发送），将编码后结果发送浏览器，之后浏览器返回给服务器仍然编码后的结果，还需要手动解码（获取）。
	JDK提供工具，进行编码
	URLEncoder：编码
	URLDecoder：解码
//编码
		String str = URLEncoder.encode("屌中屌", "UTF-8");
		System.out.println(str);  //%E5%B1%8C%E4%B8%AD%E5%B1%8C
		//解码
		String value = URLDecoder.decode(str, "UTF-8");
		System.out.println(value);

3.4	cookie案例1：记住用户名
1. 表单可以提交（文本框、复选框） ---下次浏览时，如果曾经记录应该将记录用户名显示到文本框中。
	表单必须是servlet输出，不能是html页面，否则文本框不能显示记录数据。（除非jsp）
2. 点击提交，编写servlet处理（是否勾选）
3. 如果勾选，记录用户名（记录数据下次还可以访问，所以cookie，持久cookie setMaxAge(....)）
4.如果没有勾选，删除cookie，注意：唯一标识（域名、路径、名称），cookie必须再次发送到浏览器。

扩展：将UrlEncoder 和 UrlDecoder 结合使用（解决中文用户名）


3.5	cookie案例2：历史记录
1. 点击查询所有商品
2. 通过id查询商品详情 -- 应该记录浏览记录
3. 查询所有的浏览记录，显示记录数据

4	session
	经典应用：用户登录、JD购物车等
	一次会话中，服务器用于共享数据技术。
	默认情况：session需要基于cookie使用。及没有cookie session无效。
	当第一次调用 request.getSession() ，tomcat将创建一个session对象，并将session对象id值，以cookie方式发送给浏览器，之后浏览器再次请求时，将session id 发送服务器，服务器通过request.getSession() 就可以获取之前已经创建好的session对象。
	JavaEE规范提供接口：javax.servlet.http.HttpSession 用于描述session对象。
	获得session
	request.getSession()  获得session，如果没有将创建一个新的。等效request.getSession(true)
	request.getSession(boolean) 获得session，true：没有将创建，false：没有将返回null
	session属性操作：xxxAttribute
	session 也是 servlet 域对象。（3个servlet域对象：ServletRequest、HttpSession、ServletContext）
	session生命周期
	创建：第一次调用 getSession()时创建
	销毁：
		1. 超时，默认30分钟。
			web.xml 中配置
			<session-config>
       			 <session-timeout>30</session-timeout>   单位：分钟
    		</session-config>
		2.执行api
			invalidate() 将session对象销毁了。
			setMaxInactiveInterval(int interval) 设置有效时间，单位秒
		3.服务器非正常关闭。（自杀，将JVM马上关闭）
			如果正常关闭，session将被持久化（写入到文件中）
				D:\java\tomcat\apache-tomcat-7.0.53\work\Catalina\localhost\day09\SESSIONS.ser
			当tomcat重新启动的时候这个文件就会被删除


4.1	URL重写
	当浏览器将cookie禁用，基于cookie的session将不能正常工作，每次使用request.getSession() 都将创建一个new session。只需要将session id 传递给服务器session就可以工作的。
	通过URL将session id 传递给服务器：URL重写
	手动方式： url;jsessionid=....
	api方式：
		encodeURL(java.lang.String url) 进行所有URL重写
		encodeRedirectURL(java.lang.String url) 进行重定向 URL重写
			如果参数url为空字符不同，其他都一样。
			如果浏览器禁用cooke，api将自动追加session id ，如果没有禁用，api将不进行任何修改。
	注意：如果浏览器禁用cookie，web项目的所有url都需手动重写。否则session将不能正常工作。


5	总结
cookie
	浏览器会话技术，用于在浏览器缓存内容，达到数据共享的目的
	使用
		1.服务器创建对象，添加数据
		2.服务器将cookie发送给浏览器
		3.浏览器自动携带cookie数据，服务器获得数据。
			浏览器携带数据，根据cookie路径设置。
	api
		setMaxAge 有效时间，0删除，必须保证路径匹配
		setPath 设置访问路径
		唯一标识：域名、路径、名称
session
	服务器端会话技术，用于在服务器共享数据。
	session必须使用cookie传递session id，保证多次请求，浏览器可以共享 服务器一个session对象。
	如果浏览器禁用cookie，必须进行URL重写。
	session生命周期：

