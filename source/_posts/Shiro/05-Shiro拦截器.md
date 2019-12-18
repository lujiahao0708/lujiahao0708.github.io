---
title: 05-Shiro拦截器
date: 2019-10-16 20:42:34
categories: Shiro
tags:
- Shiro
---

Shiro内置了很多默认的拦截器，比如身份验证、授权等相关的。

<!--more-->

## 拦截器列表

|  Filter Name   | Class  | Description  |
|  ----  | ----  | ----  |
|anon	            |org.apache.shiro.web.filter.authc.AnonymousFilter |	匿名拦截器，即不需要登录即可访问；一般用于静态资源过滤；示例/static/**=anon
|authc	            |org.apache.shiro.web.filter.authc.FormAuthenticationFilter |	基于表单的拦截器；如/**=authc，如果没有登录会跳到相应的登录页面登录
|authcBasic	        |org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter |	Basic HTTP身份验证拦截器
|logout	            |org.apache.shiro.web.filter.authc.LogoutFilter |	退出拦截器，主要属性：redirectUrl：退出成功后重定向的地址（/），示例/logout=logout
|noSessionCreation	|org.apache.shiro.web.filter.session.NoSessionCreationFilter |	不创建会话拦截器，调用subject.getSession(false)不会有什么问题，但是如果subject.getSession(true)将抛出DisabledSessionException异常
|perms	            |org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter |	权限授权拦截器，验证用户是否拥有所有权限；属性和roles一样；示例/user/**=perms["user:create"]
|port	            |org.apache.shiro.web.filter.authz.PortFilter |	端口拦截器，主要属性port(80)：可以通过的端口；示例/test= port[80]，如果用户访问该页面是非80，将自动将请求端口改为80并重定向到该80端口，其他路径/参数等都一样
|rest	            |org.apache.shiro.web.filter.authz.HttpMethodPermissionFilter |	rest风格拦截器，自动根据请求方法构建权限字符串；示例/users=rest[user]，会自动拼出user:read,user:create,user:update,user:delete权限字符串进行权限匹配（所有都得匹配，isPermittedAll）
|roles	            |org.apache.shiro.web.filter.authz.RolesAuthorizationFilter |	角色授权拦截器，验证用户是否拥有所有角色；示例/admin/**=roles[admin]
|ssl	                |org.apache.shiro.web.filter.authz.SslFilter |	SSL拦截器，只有请求协议是https才能通过；否则自动跳转会https端口443；其他和port拦截器一样；
|user	            |org.apache.shiro.web.filter.authc.UserFilter |	用户拦截器，用户已经身份验证/记住我登录的都可；示例/**=user

> anon,authcBasic,auchc,user是认证过滤器. perms,roles,ssl,rest,port是授权过滤器

## 参考资料
- [http://shiro.apache.org/10-minute-tutorial.html](http://shiro.apache.org/10-minute-tutorial.html)
- [https://mrbird.cc/tags/Shiro/](https://mrbird.cc/tags/Shiro/)

## 代码获取
[https://github.com/lujiahao0708/LearnSpring](https://github.com/lujiahao0708/LearnSpring)

## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我的公众号：庄里程序猿，读书笔记教程资源第一时间获得！

![](https://github.com/lujiahao0708/PicRepo/raw/master/公众号二维码.jpg)
