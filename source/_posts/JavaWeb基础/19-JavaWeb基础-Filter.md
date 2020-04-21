---
title: 19-JavaWeb基础-Filter
categories: JavaEE
tags:
  - JavaEE
  - 后台
  - 过滤器
description: JavaEE中Filter过滤器的介绍/使用方法/生命周期/方法详解/案例介绍.
abbrlink: 9ce38a25
date: 2017-01-24 15:46:08
---

## 1. 介绍
当访问资源时，如果不希望此资源被访问，可以使用过滤器进行拦截。
访问一个资源，需要路径，过滤器通过路径匹配进行资源拦截。
过滤器必须实现接口：javax.servlet.Filter

	Filter
		init(FilterConfig):void		初始化
		doFilter(ServletRequest,ServletResponse,FilterChain):void	过滤方法
		destroy():void		销毁
默认情况下过滤器会拦截匹配路径和执行的资源
需要手动放行:`chain.doFilter(request,response);`
## 2.使用方法
详见`统计访问次数`的案例

## 3.生命周期
3.1 `init(FilterConfig)`  
	
	初始化方法
		执行时机：服务器启动时执行
		执行者：tomcat
	FilterConfig：当前过滤器的配置对象
		getFilterName():String	过滤名称,相当于<filter-name>标签
		getServletContext():ServletContext	ServletContext对象引用
		getInitParametes(String):String	获得过滤器初始化参数
		getInitParameterNames():Enumeration	获得过滤器初始化参数的所有名字
	
		<filter>
        	<filter-name>CountFilter</filter-name>
        	<filter-class>com.lujiahao.cms.filter.CountFilter</filter-class>
			<init-param>
				<param-name>encoding</param-name>
				<param-value>UTF-8</param-value>
    	</filter>
3.2 `doFilter(ServletRequest,ServletResponse ,FilterChain)`

	路径匹配一次执行一次。一次请求拦截一次。
3.3 `destroy()`

	执行时机：服务器正常关闭时执行
	执行者：tomcat

## 4.doFilter方法详解
参数1ServletRequest 和参数2  ServletResponse
	
	回顾：Object  			 List   			ArrayList
		  ServletRequest		HttpServletRequest		tomcat实现类
	结论：可以强转获得Http协议有关对象
		HttpServletRequest  request = （HttpServletRequest） req;
		HttpServletResponse  request = （HttpServletResponse） res;
参数3：FilterChain
	
	FilterChain 过滤器链，当请求资源时，多个过滤器都生效，tomcat将生成一个链，用于存放多个过滤器。
	第一个过滤器执行完成，并发行时，将执行第二个过滤器，。。。。。当最后一个过滤器放行时，将执行资源。
![aaaaa.png](https://ooo.0o0.ooo/2016/07/27/5798496b3b3ed.png)

过滤器链中 过滤器执行顺序 与 <filter-mapping> 映射在web.xml配置顺序一致的。web.xml 先配置先执行

## 5.`<url-pattern>`匹配路径
回顾：servlet路径配置

	1.完全匹配，必须/开头 ，例如：/a/b/c/oneServlet
	2.不完全匹配，以/开头   *结尾，例如：/a/b/*  , /*
	3.通配符匹配. *.开头，例如： *.jsp  、 *.do   、*.action 等
	4 缺省  /   以上都没有匹配将执行。
过滤器路径编写，与servlet一致的，过滤器过滤的就是指定servlet。
	
	/oneServlet  、 /twoServlet
	/*   过滤器 将匹配以上两个servlet

## 6.`dispatcher`配置
<filter-mapping><dispatcher> 用于配置过滤器在何处进行拦截。

	取值：	REQUEST、 FORWARD、 INCLUDE、ERROR
	request : 表示在请求开始时拦截。默认值。
	forward：表示在请求转发开始时拦截。及 request.getRequestDistacher(..).forward 被调用时。
	include：表示在请求包含开始时拦截。及 request.getRequestDistacher(..).include 被调用时。
	error ： 表示程序异常，显示友好页面之前拦截。

## 案例〇 : dispatcher配置友好错误页面
IE会将500的错误以他自己的方式展现出来,这就不符合需求,所以需要手动配置友好错误页面.
如果有多个友好错误界面,不能在每个jsp页面更改状态码,这就需要一个过滤器来统一过滤.

1.创建`FriendErrorFilter`实现`javax.servlet.Filter`接口

	/**
	 * 友好错误界面过滤器
	 * Created by lujiahao on 2016/7/27.
	 */
	public class FriendErrorFilter implements Filter {
	    @Override
	    public void init(FilterConfig filterConfig) throws ServletException {
	
	    }
	
	    @Override
	    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
	        HttpServletRequest request = (HttpServletRequest) servletRequest;
	        HttpServletResponse response = (HttpServletResponse) servletResponse;
	
	        // 修改状态码
	        response.setStatus(200);
	        // 放行
	        filterChain.doFilter(request,response);
	    }
	
	    @Override
	    public void destroy() {
	
	    }
	}
2.配置到tomcat(web.xml)

	<!--配置友好错误页面过滤器-->
    <error-page>
        <error-code>500</error-code>
        <location>/demo0/frienderror.jsp</location>
    </error-page>
    <!--友好错误页面过滤器-->
    <filter>
        <filter-name>FriendErrorFilter</filter-name>
        <filter-class>com.lujiahao.filter.FriendErrorFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>FriendErrorFilter</filter-name>
        <url-pattern>/frienderror.jsp</url-pattern>
        <dispatcher>ERROR</dispatcher>
    </filter-mapping>
3.编写友好错误界面(error.jsp)

	<html>
		<head>
		    <title>Title</title>
		</head>
		<body>
			我是友好错误界面<br/>
			服务器繁忙,请稍后重试
		</body>
	</html>
4.模拟服务器500错误

	<%
	    int i = 1/0;
	%>

## 案例一 : 统计访问次数
原理:将数据记录到ServletContext 所有用户共享

1.创建`CountFilter`实现`javax.servlet.Filter`接口
	
	/**
	 * 统计次数的过滤器
	 * Created by lujiahao on 2016/7/27.
	 */
	public class CountFilter implements Filter {
	    @Override
	    public void init(FilterConfig filterConfig) throws ServletException {
	
	    }
	
	    @Override
	    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
	        HttpServletRequest request = (HttpServletRequest) servletRequest;
	        HttpServletResponse response = (HttpServletResponse) servletResponse;
	
	        // 获得访问路径   /pages/main/main.jsp
	        String servletPath = request.getServletPath();
	        String filePath = servletPath.substring(servletPath.lastIndexOf("/")+1,servletPath.lastIndexOf("."));
	
	        // 获得servletContext获取统计数据
	        ServletContext servletContext = request.getSession().getServletContext();
	        Integer num = (Integer) servletContext.getAttribute(filePath);
	        if (num == null) {
	            num = 1;// 第一次访问
	        } else {
	            num ++;
	        }
	        // 存放到ServletContext中
	        servletContext.setAttribute(filePath,num);
	
	        // 放行
	        filterChain.doFilter(request,response);
	    }
	
	    @Override
	    public void destroy() {
	
	    }
	}
2.配置到tomcat(web.xml)
	
	<!--统计次数的过滤器-->
    <filter>
        <filter-name>CountFilter</filter-name>
        <filter-class>com.lujiahao.cms.filter.CountFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>CountFilter</filter-name>
        <url-pattern>/pages/main/main.jsp</url-pattern>
    </filter-mapping>
3.JSP中使用

	访问次数:${applicationScope.main}次<br/>

## 案例三 : 自动登录

原理图:
![QQ截图20160727163940.png](https://ooo.0o0.ooo/2016/07/27/5798737cdfea1.png)

1.修改以前的登录逻辑,添加对自动登录的标记

	/**
     * 登录操作
     */
    private void login(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 1.获取数据并封装
        Customer customer = LBeanUtils.populate(Customer.class, request.getParameterMap());
        // 2.通知service进行登录
        CustomerServeice customerServeice = new CustomerServiceImpl();
        Customer loginCustomer = customerServeice.login(customer);

        // 3.处理
        if (loginCustomer != null) {
            // -------------自动登录start-------------
            // checkbox选中的话获得的值是on
            String autologin = request.getParameter("autologin");
            if (autologin != null) {
                // 勾选了记住密码,  使用cookie将用户的登录名和密码发送到浏览器,并通知浏览器保存(持久化cookie)
                // 1.创建cookie对象
                String username = URLEncoder.encode(loginCustomer.getName(), "UTF-8");
                String password = URLEncoder.encode(loginCustomer.getPwd(), "UTF-8");
                // 存入需要进行URLEndode编码,不然会报错:http://blog.csdn.net/liuxiao723846/article/details/22155393
                Cookie cookie = new Cookie("autoLoginCookie", username + "&" + password);
                // 2.设置有效时间
                cookie.setMaxAge(60 * 60);// 1小时
                // 3.设置路径
                cookie.setPath("/");
                // 4.发送cookie
                response.addCookie(cookie);
            }
            // -------------自动登录end-------------

            // 成功  -- session记录登录状态,重定向到主页面
            // * session作用域保存数据
            request.getSession().setAttribute("loginCustomer", loginCustomer);
            response.sendRedirect(request.getContextPath() + "/pages/main/main.jsp");
        } else {
            // 不成功  --- request记录当次请求提示,请求转发到login.jsp 显示数据
            request.setAttribute("msg", "用户名和密码不匹配");
            request.setAttribute("customer", customer);// 回显数据
            request.getRequestDispatcher("pages/login/login.jsp").forward(request, response);
        }
    }
2.编写自动登录的过滤器

	/**
	 * 自动登录的过滤器
	 * Created by lujiahao on 2016/7/27.
	 */
	public class AutoLoginFilter implements Filter {
	    @Override
	    public void init(FilterConfig filterConfig) throws ServletException {
	
	    }
	
	    @Override
	    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
	        HttpServletRequest request = (HttpServletRequest) servletRequest;
	        HttpServletResponse response = (HttpServletResponse) servletResponse;
	
	        Customer loginCustomer = (Customer) request.getSession().getAttribute("loginCustomer");
	        // 1.已经登录
	        if (loginCustomer != null) {
	            filterChain.doFilter(request,response);
	            return;
	        }
	
	        // 2.没有登录,获取cookie信息
	        Cookie[] allCookie = request.getCookies();
	        String cookieValue = null;
	        if (allCookie != null) {
	            for (Cookie cookie : allCookie) {
	                if ("autoLoginCookie".equals(cookie.getName())) {
	                    cookieValue = cookie.getValue();
	                    break;// 一旦查询到了就跳出循环,提升性能
	                }
	            }
	        }
	
	        // 3.没有cookie信息
	        if (cookieValue == null) {
	            filterChain.doFilter(request,response);
	            return;
	        }
	
	        // 4.查询到cookie信息,获得用户数据
	        String username = URLDecoder.decode(cookieValue.split("&")[0],"UTF-8");
	        String password = URLDecoder.decode(cookieValue.split("&")[1],"UTF-8");
	        Customer customer = new Customer(username,password);
	
	        // 5.查询用户信息
	        CustomerServeice customerServeice = new CustomerServiceImpl();
	        loginCustomer = customerServeice.login(customer);
	
	        // 6.没有查询到用户信息----例如:密码修改
	        if (loginCustomer == null) {
	            filterChain.doFilter(request,response);
	            // 应该删除cookie
	            Cookie cookie = new Cookie("autoLoginCookie","");
	            cookie.setMaxAge(0);// 删除
	            cookie.setPath("/");// 保存的时候写的路径要和这里一样
	            response.addCookie(cookie);
	            return;
	        }
	
	        // 7.查询到用户信息,执行自动登录
	        request.getSession().setAttribute("loginCustomer",loginCustomer);
	
	        // 放行
	        filterChain.doFilter(request,response);
	    }
	    @Override
	    public void destroy() {
	
	    }
	}

## 案例四 : GET/POST中文乱码统一处理
1.旧的处理方式:

	public class GetPostServlet extends HttpServlet {
	    @Override
	    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        /*
			String username = request.getParameter("username");
			String password = request.getParameter("password");
	
			//处理编码
			username = new String(username.getBytes("ISO8859-1"),"UTF-8");
			password = new String(password.getBytes("ISO8859-1"),"UTF-8");
	
			System.out.println("get:");
			System.out.println(username);
			System.out.println(password);
			*/
	        this.doPost(request, response);
	    }
	
	    @Override
	    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        //post请求乱码
	        //request.setCharacterEncoding("UTF-8");
	
	        String username = request.getParameter("username");
	        String password = request.getParameter("password");
	
	        System.out.println("post:");
	        System.out.println(username);
	        System.out.println(password);
	    }
	}
2.使用过滤器+装饰着模式的处理方式:

	/**
	 * 处理Get/Post请求乱码的过滤器
	 * Created by lujiahao on 2016/7/27.
	 */
	public class GetPostEncodingFilter implements Filter {
	    @Override
	    public void init(FilterConfig filterConfig) throws ServletException {
	
	    }
	
	    @Override
	    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
	        HttpServletRequest request = (HttpServletRequest) servletRequest;
	        HttpServletResponse response = (HttpServletResponse) servletResponse;
	
	        //编码设置 另一个filter编写   这个只能解决POST乱码的问题
	        request.setCharacterEncoding("UTF-8");
	
	        // 这个里面有处理GET乱码
	        // 使用装饰者模式增强的request
	        MyRequest myRequest = new MyRequest(request);
	        // 放行
	        filterChain.doFilter(myRequest,response);
	    }
	
	    @Override
	    public void destroy() {
	
	    }
	}

	/**
	 * 使用装饰者设计模式
	 * HttpServletRequestWrapper是系统为我们准备好的装饰者的父类,已经完成了相应的工作
	 * Created by lujiahao on 2016/7/27.
	 */
	public class MyRequest extends HttpServletRequestWrapper {
	    private boolean isEncoding = false;// 默认是没有编码的
	    private HttpServletRequest request;
	    public MyRequest(HttpServletRequest request) {
	        super(request);// 将tomcat的request传入父类,提供给不需要增强的方法使用
	        this.request = request;
	    }
	
	    // 需要增强的方法直接覆写父类方法就好了
	
	    // 通过名称获得第一个值
	    @Override
	    public String getParameter(String name) {
	        String[] parameterValues = this.getParameterValues(name);
	        if (parameterValues == null) {
	            return null;
	        }
	        return parameterValues[0];
	    }
	
	    // 通过名称获得所有的值
	    @Override
	    public String[] getParameterValues(String name) {
	        return this.getParameterMap().get(name);
	    }
	
	    @Override
	    public Map<String, String[]> getParameterMap() {
	        // 1.获得tomcat原始数据
	        Map<String, String[]> map = request.getParameterMap();
	        // 2.处理get请求的乱码
	        if ("GET".equals(request.getMethod()) && !isEncoding) {
	            // 3.遍历map
	            for(Map.Entry<String,String[]> entry : map.entrySet()){
	                // 4.获得所有value数据
	                String[] value = entry.getValue();
	                // 处理所有乱码
	                for (int i = 0; i < value.length; i++) {
	                    try {
	                        value[i] = new String(value[i].getBytes("ISO-8859-1"),"UTF-8");
	                    } catch (UnsupportedEncodingException e) {
	                        throw new RuntimeException(e);
	                    }
	                }
	            }
	            isEncoding = true;// 已经解决完乱码就不要再次解决了
	        }
	        return map;
	    }
	}
## 案例五 : 页面静态化 & 全栈压缩
页面静态化原理图:
![QQ截图20160728133730.png](https://ooo.0o0.ooo/2016/07/28/57999a3a4ed25.png)
1.装饰者模式增强HttpServletResponse

	public class MyResponse extends HttpServletResponseWrapper{
	
		private HttpServletResponse response;
		
		//提供自定义缓存
		private ByteArrayOutputStream baos = new ByteArrayOutputStream();
	
		private PrintWriter pw = null;
		public MyResponse(HttpServletResponse response) {
			super(response);
			this.response = response;
		}
		
		@Override
		public PrintWriter getWriter() throws IOException {
			if(pw == null){
				pw = new PrintWriter(new OutputStreamWriter(baos , "UTF-8" ));
				System.out.println("执行" + pw);
			}
			return pw;
		}
		
		/**
		 * 获得自定义缓存内容
		 * @return
		 */
		public byte[] getData(){
			return baos.toByteArray();
		}
	
	}
2.过滤器写法

	public class PageStaticFilter implements Filter {
	
		@Override
		public void init(FilterConfig filterConfig) throws ServletException {
		}
	
		@Override
		public void doFilter(ServletRequest req, ServletResponse resp,
				FilterChain chain) throws IOException, ServletException {
			HttpServletRequest request = (HttpServletRequest) req;
			HttpServletResponse response = (HttpServletResponse) resp;
			
			//1 获得页面位置
			//D:\java\tomcat\apache-tomcat-7.0.53\webapps\day19_demo\demo04\1.html
			ServletContext sc = request.getSession().getServletContext();
			String id= request.getParameter("id");
			String path = "/demo04/"+id+".html";// 静态页面存放的位置
			String htmlPath = sc.getRealPath(path);
	
			//2 文件是否存在
			File htmlFile = new File(htmlPath);
			if(htmlFile.exists()){
				//存在就显示
				request.getRequestDispatcher(path).forward(request, response);
				return;
			}
			
			// 自定义response 提供缓存，用于存放 发送到浏览器所有内容
			MyResponse myResponse = new MyResponse(response);
			//放行
			chain.doFilter(request, myResponse);
			
			//内容已经写入到自定义缓存
			byte[] arrayByte = myResponse.getData();
	
			/***写入到文件 start**/
			//将指定的内容内容到文件
			FileOutputStream out = new FileOutputStream(htmlFile);
			out.write(arrayByte);
			out.close();
			
			//手动写入到浏览器
			response.getOutputStream().write(arrayByte);
			/***写入到文件 end**/
			
			/**全站压缩 ， 将需要发送的数据，压缩之后再发给浏览器，节省流量
				全站压缩和页面静态化是冲突的,需要一个一个来测试**/
			/*
			// 设置发送的数据位压缩数据
			response.setHeader("content-encoding", "gzip");
			ByteArrayOutputStream baos = new ByteArrayOutputStream(); //存放压缩后的结果
			GZIPOutputStream gzipOut = new GZIPOutputStream(baos); //压缩的位置
			gzipOut.write(arrayByte); //需要压缩的原始数据
			gzipOut.close();
			
			response.getOutputStream().write(baos.toByteArray());
			*/
		}
	
		@Override
		public void destroy() {
		}
	}





## 扩展:装饰者设计模式
乱码解决的十几分钟的时候讲了这个东西

## jar包和war包的区别
java项目 压缩 ：jar包
web项目 压缩：war包 。 当war添加到tomcat/webapps下将自动解压
	jar包和war包压缩格式都是 zip

## 升级订制版HttpFilter
这个非常好,用到的思想和Servlet中的service()方法非常类似,好好总结一下
http://www.cnblogs.com/jianjianyang/p/5001471.html