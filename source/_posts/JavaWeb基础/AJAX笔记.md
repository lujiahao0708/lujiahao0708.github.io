---
title: AJAX笔记
categories: JavaEE
tags:
  - JavaEE
  - 后台
  - AJAX
description: JavaEE中AJAX
abbrlink: fce3647f
date: 2016-07-28 14:09:10
---

## Ajax介绍&编写流程
使用异步的方式从浏览器端发送请求，请求服务器端资源，并获得内容一种技术。
ajax 不是新技术，是多个技术整合：javascript、html、css、xml，XMLHttpRequest
![a.png](https://ooo.0o0.ooo/2016/07/28/5799ca2b5237d.png)

使用步骤:

	1.获得核心类（引擎）
	2.编写回调函数，获得响应内容
	3.建立连接
	4发送请求

同步操作 : 访问servlet地址栏改变
异步操作 : 访问servlet地址栏不改变

## Jsonlib
需要导入的jar包:

	commons-beanutils-1.8.3.jar
	commons-collections-3.2.1.jar
	commons-lang-2.6.jar
	commons-logging-1.1.1.jar
	ezmorph-1.0.6.jar
	json-lib-2.4-jdk15.jar
Java对象转json:

	JSONArray.fromObject(javaObject).toString()
Json转Java对象:

	JSONObject obj = JSONObject.fromObject(jsonStr);
	Bean bean = (Bean) JSONObject.toBean(obj, Bean.class);
注意:
	
	JsonConfig配置信息,设置排除的字段

	User user = new User("u001", "jack", "1234");
	// 配置信息 --设置排除
	JsonConfig jsonConfig = new JsonConfig();
	jsonConfig.setExcludes(new String[]{"username","password"});
	
	// java 对象  转换 json  --> {'k':'v',.....}
	String str = JSONObject.fromObject(user,jsonConfig).toString();
	System.out.println(str);// {"id":"u001"}

## XStream
需要导入的jar包:

	xmlpull-1.1.3.1.jar
	xpp3_min-1.1.4c.jar
	xstream-1.4.7.jar
Java对象转xml:

	public void demo02(){
		User user = new User("u007", "路家豪", "123");
		//1 核心类
		XStream xStream = new XStream();
		// * 设置别名,如果不设置的话会使用User类的全限定类名来作为根元素
		xStream.alias("user", User.class);
		//2 转换成xml
		String xmlData = xStream.toXML(user);
		System.out.println(xmlData);
		/*
		 * <user>
			  <id>u007</id>
			  <username>路家豪</username>
			  <password>123</password>
			</user>
		 */
	}

	设置属性的值:
	public void demo03(){
		User user = new User("u007", "路家豪", "123");
		//1 核心类
		XStream xStream = new XStream();
		// * 设置元素别名
		xStream.alias("user", User.class);
		// * 设置属性别名 , 三个参数分别标售:指定javabean/property属性名/xml中显示的属性名
		xStream.aliasAttribute(User.class, "id", "id");
		//2 转换成xml
		String xmlData = xStream.toXML(user);
		System.out.println(xmlData);
		
		/*
		 * <user id="u007">
			  <username>路家豪</username>
			  <password>123</password>
			</user>
		 */
	}

	忽略字段:
	public void demo06(){
		User user = new User("u007", "路家豪", "123");
		//1 核心类
		XStream xStream = new XStream();
		// * 设置元素别名
		xStream.alias("user", User.class);
		// * 设置属性别名 , 指定javabean property 使用其他别名
		xStream.aliasAttribute(User.class, "id", "id");
		// * 忽略指定的字段
		xStream.omitField(User.class, "username");
		
		//2 转换成xml
		String xmlData = xStream.toXML(user);
		System.out.println(xmlData);
		/*
		 * <user id="u007">
			  <password>123</password>
			</user>
		 */
	}

xml转Java对象:

	public void demo04(){
		String str = "<user id='u007'><username>路家豪</username><password>123</password></user>";
		//1 核心类
		XStream xStream = new XStream();
		// * 设置元素别名
		xStream.alias("user", User.class);
		// * 设置属性别名 , 指定javabean property 使用其他别名
		xStream.aliasAttribute(User.class, "id", "id");
		
		//2 xml转换成 javabean
		User user = (User)xStream.fromXML(str);
		System.out.println(user);
	}

## 案例一:判断用户名是否存在
html代码:

	<input id="name" type="text" name="name" onblur="sendData(this)" onfocus="document.getElementById('spanId').innerHTML=''"/>
	<span id="spanId"></span>
js代码:

    <script type="text/javascript">
        function sendData(obj) {
            // 0.获得输入的数据
            var inputValue = obj.value;
            /* 1 创建核心类
                 XMLHttpRequest ajax引擎，不同的浏览器对此对象的创建方式不同。存在浏览器兼容问题
                 * 在所有现代浏览器中（包括 IE 7）：
                    xmlhttp=new XMLHttpRequest()
                 * 在 Internet Explorer 5 和 6 中：
                    xmlhttp=new ActiveXObject("Microsoft.XMLHTTP")
             */
            var xmlhttp = null;
            if (window.XMLHttpRequest) {// code for all new browsers
                xmlhttp = new XMLHttpRequest();
            } else if (window.ActiveXObject) {// code for IE5 and IE6
                xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
            }
            /* 2 设置回调函数，通过属性设置 onreadystatechange
                 * 目的：发送请求之后可以获得服务器响应内容。
                 * 一般情况使用匿名函数实现
                 * ajax引擎在不同的阶段都会调用回调函数。
                 * 属性：readyState 和 stauts
                 2.1 readyState 返回整形数据，表示当前执行某一个阶段。
                     * 0  初始化状态。核心对象创建时默认值
                     * 1 open() 方法已调用，连接创建完成之后，ajax引擎将此状态修改1
                     * 2 send() 方法已调用，发送请求
                     * 3  接受中， 所有响应头部都已经接收到。响应体开始接收但未完成。
                     * 4  响应已经完全接收。【掌握】--服务器发送的所有数据，已经到ajax引擎内部。
                 2.2 status 表示  响应的http状态码
                     * 200 正常【掌握】
                     * 302 重定向
                     * 304 缓存
                     * 404 不存在
                     * 500 服务器异常
             */
            xmlhttp.onreadystatechange = function () {
                if (xmlhttp.readyState == 4 && xmlhttp.status == 200){
                    // 3.1 接收服务器响应的数据,获得json数据,注意json也是文本
                    var data = xmlhttp.responseText;
                    // 3.2 将字符串转化成json对象  使用格式:eval("('abc')");
                    var jsonData = eval("("+data+")");

                    // 3.3 判断  控制按钮是否可用
                    var buttonObj = document.getElementById("buttonId");
                    var spanObj = document.getElementById("spanId");
                    if (jsonData.flag) {// 可用
                        buttonObj.removeAttribute("disabled");
                        spanObj.style.color = "#3D882D";
                    } else {// 不可用
                        buttonObj.setAttribute("disabled","disabled");
                        spanObj.style.color = "#CC0000";
                    }
                    // 3.4 将信息显示到指定位置
                    spanObj.innerHTML = jsonData.msg;
                }
            };
            //3 建立连接
            xmlhttp.open("POST","${pageContext.request.contextPath}/CustomerServlet?method=checkName");

            // 4 POST请求需要设置编码
            xmlhttp.setRequestHeader("content-type", "application/x-www-form-urlencoded");

            //5 发送请求  格式: name=lujiahao&password=1234
            xmlhttp.send("name=" + inputValue);
        }
    </script>
Servlet端代码:

	/**
     * Ajax查询用户名是否存在
     */
    private void checkName(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException{
        // 0响应乱码
        response.setContentType("application/json;charset=UTF-8");
        try {
            // 1.获得数据并封装
            String name = request.getParameter("name");
            // 2.通知service查询所有
            //CustomerServeice customerServeice = new CustomerServiceImpl();
            // customerServeice.checkName(name);

            PrintWriter out = response.getWriter();
            if (name == null || "".equals(name)) {
                out.print("{'flag':false,'msg':'用户名不能为空'}");
                return;
            } else if ("lujiahao".equals(name)) {
                out.print("{'flag':false,'msg':'用户名已经存在'}");
            } else {
                out.print("{'flag':true,'msg':'用户名可用'}");
            }
        } catch (Exception e){
            // 1.打印日志
            e.printStackTrace();
        }
    }



## 案例二:省市县三级联动

