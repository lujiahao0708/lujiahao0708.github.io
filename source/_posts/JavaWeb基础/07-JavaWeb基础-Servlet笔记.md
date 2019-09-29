---
title: 07.JavaWeb基础_Servlet笔记
date: 2016-11-20 09:52:43
tags:
  - JavaWeb基础
  - Servlet
description: Servlet基础总结
---

## Servlet基础
是用Java编写的服务器端程序

## 类关系
![Servlet类关系图.bmp](https://ooo.0o0.ooo/2016/11/19/58306becf3784.bmp)

## 根据生命周期打断点

![启动Tomcat调试断点.gif](https://ooo.0o0.ooo/2016/11/19/5830713e6ac1a.gif)

## 生命周期执行流程

### init()方法
当我们首次访问`HelloServlet`时,会在tomcat中创建该Servlet,此时会执行`init()`方法,Java中方法执行是首先从当前类中找是否有此方法,如果没有的话就会往该类的父类里面找.
这就到了HttpServlet方法,发现里面也没有`init()`方法,继续往上找.
而GenericServlet中有相关的东西

	public void init(ServletConfig config) throws ServletException {
        this.config = config;
        this.init();
    }

    public void init() throws ServletException {
    }
第一个是生命周期的方法,而下面的方法是用来被子类覆写的初始化方法,这样的写法既可以实现初始化`Servlet`,又可以避免子类覆写父类的方法造成父类方法无法执行的问题.

### service()方法
`service()`方法和这个是类似的流程.首先会从`HelloServlet`中寻找`service()`方法,没有,进入到`HttpServlet`中寻找,找到

	public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        HttpServletRequest request;
        HttpServletResponse response;
        try {
            request = (HttpServletRequest)req;
            response = (HttpServletResponse)res;
        } catch (ClassCastException var6) {
            throw new ServletException("non-HTTP request or response");
        }

        this.service(request, response);
    }
这里会调用一个`service()`的重载方法,这个重载方法里面有执行其中操作的具体方法,以`doGet()`为例,虽然`HttpServlet`中有`doGet()`的实现,但是我们的`HelloServlet`中覆写了`doGet()`方法,具体的实现就看我们的覆写的方法中的操作了.

### destory()方法
这个没啥好写的,就是销毁

## Servlet总结
- Servlet是单例，一生只有一个实例。
- 多个线程同时执行doGet方法，存在线程并发问题（安全问题）
解决方案：不要使用成员变量
- 初始化操作：复写init()
- 业务操作：在`doGet()`和`doPost()`
- 所有的servlet都必须在web.xml配置

>注意：配置当前使用的servlet（父类HttpServlet和爷类GenericServlet不要配置的）
