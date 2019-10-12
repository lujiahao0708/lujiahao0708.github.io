---
title: 01.Shiro简介
date: 2019-10-12 11:42:34
categories: Shiro
tags:
- Shiro

---

Apache Shiro（发音为“shee-roh”，日语“堡垒（Castle）”的意思）是一个强大易用的 Java 安全框架，提供了认证、授权、加密和会话管理功能，可为任何应用提供安全保障 - 从命令行应用、移动应用到大型网络及企业应用。

<!--more-->

## 1. 权限管理
权限管理包括用户身份认证和授权两个部分,简称认证授权
- 身份认证 : 为判断用户是否为合法用户的过程,最常用的身份认证方式就是系统通过核对用户输入的用户名和口令,将其与系统中存储的该用户信息相对比,继而来判断用户身份是否正确
- 授权 : 既访问权限,控制用户访问资源的权限. 主体(subject)进行身份认证后需要分配权限方可访问系统的资源

## 2. Shiro核心概念
三个核心的概念Subject，SecurityManager和Realms，如图所示：

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Shiro/shiro-%E4%B8%89%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5.png)

- Subject : 主体,代表了当前”用户”,这个用户不一定是一个具体的人,与当前应用交互的任何东西都是Subject,如网络爬虫,机器人等,即为一个抽象概念. 所有Subject都绑定到SecurityManager,与Subject的所有交互都会委托给SecurityManager. 可以把Subject认为是一个门面,SecurityManager才是实际的执行者
- SecurityManager : 安全管理器,即所有与安全有关的操作都会与SecurityManager交互,且它管理着所有Subject. 可以看出它是Shiro的核心,它负责与后边介绍的其他组件进行交互,如果学习过SpringMVC,你可以把它看成DispatcherServlet前端控制器
- Realm : 域,Shiro从Realm获取安全数据(如用户、角色、权限),就是说SecurityManager要验证用户身份,那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法,也需要从Realm得到用户相应的角色 / 权限进行验证用户是否能进行操作. 可以把Realm看成 DataSource,即安全数据源

Shiro基本工作流程：

- 应用代码通过Subject来进行认证和授权,而Subject又将所有的互交都委托给了SecurityManager
- 我们需要给Shiro的SecurityManager注入Realm,从而让SecurityManager能得到合法的用户及其权限进行判断

> Shiro不提供维护用户 / 权限，而是通过Realm让开发人员自己注入。通过继承`AuthorizingRealm`抽象类，实现`doGetAuthorizationInfo`和`doGetAuthenticationInfo`方法。

## 3. Shiro架构

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Shiro/shiro-framework.png)

- Authentication : 身份认证 / 登录,验证用户是不是拥有相应的身份
- Authorization : 授权,即权限验证,验证某个已认证的用户是否拥有某个权限. 即判断用户是否能做事情,常见的如 : 验证某个用户是否拥有某个角色,或者细粒度的验证某个用户对某个资源是否具有某个权限
- Session Manager : 会话管理,即用户登录后就是一次会话,在没有退出之前,它的所有信息都在会话中. 会话可以是普通JavaSE环境的,也可以是如Web环境的
- Cryptography : 加密,保护数据的安全性,如密码加密存储到数据库,而不是明文存储
- Web Support : Web支持,可以非常容易的集成到Web环境中
- Caching : 缓存,比如用户登录后,其用户信息、拥有的角色 / 权限不必每次去查,这样可以提高效率
- Concurrency : shiro支持多线程应用的并发验证,即如在一个线程中开启另一个线程,能把权限自动传播过去
- Testing : 提供测试支持
- Run As : 允许一个用户假装为另一个用户(在他们允许的情况下)的身份进行访问
- Remember Me : 记住我,这个是非常常见的功能,即一次登录后,下次不用重复登录了

## 4. Shiro架构详解

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Shiro/Shiro-%E5%86%85%E9%83%A8%E7%BB%93%E6%9E%84%E5%9B%BE.png)

- Subject : 主体,可以看到主体可以是任何可以与应用交互的”用户”
- SecurityManager : 相当于SpringMVC中的DispatcherServlet或Struts2中的FilterDispatcher; 是Shiro的心脏. 所有具体的交互都通过SecurityManager进行控制,它管理着所有Subject、且负责进行认证和授权、及会话、缓存的管理
- Authenticator : 认证器,负责主体认证的,这是一个扩展点,如果用户觉得Shiro默认的不好,可以自定义实现. 其需要认证策略(Authentication Strategy),即什么情况下算用户认证通过了
- Authrizer : 授权器,或者访问控制器,用来决定主体是否有权限进行相应的操作,即控制着用户能访问应用中的哪些功能
- Realm : 可以有1个或多个Realm,可以认为是安全实体数据源,即用于获取安全实体. 可以是JDBC实现,也可以是LDAP实现,或者内存实现等,由用户提供. 注意 : Shiro不知道你的用户 / 权限存储在哪及以何种格式存储,所以我们一般在应用中都需要实现自己的Realm
- SessionManager : 如果写过Servlet就应该知道Session的概念,Session需要有人去管理它的生命周期,这个组件就是SessionManager,而Shiro并不仅仅可以用在Web环境,也可以用在如普通的JavaSE环境、EJB等环境. 所以Shiro就抽象了一个自己的Session来管理主体与应用之间交互的数据. 这样的话,比如我们在Web环境用,刚开始是一台Web服务器,接着又上了台EJB服务器,这时想把两台服务器的会话数据放到一个地方,这个时候就可以实现自己的分布式会话(如把数据放到Memcached服务器)
- SessionDAO : DAO大家都用过,数据访问对象,用于会话的CRUD,比如我们想把Session保存到数据库,那么可以实现自己的 SessionDAO,通过如JDBC写到数据库. 比如想把Session放到Memcached中,可以实现自己的Memcached SessionDAO. 另外SessionDAO中可以使用Cache进行缓存,以提高性能
- CacheManager : 缓存控制器,来管理如用户、角色、权限等的缓存的. 因为这些数据基本上很少去改变,放到缓存中后可以提高访问的性能
- Cryptography : 密码模块,Shiro提高了一些常见的加密组件用于如密码加密 / 解密

## 参考资料
- [https://www.infoq.cn/article/apache-shiro/](https://www.infoq.cn/article/apache-shiro/)
- [http://shiro.apache.org/architecture.html](http://shiro.apache.org/architecture.html)

## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我的公众号：庄里程序猿，读书笔记教程资源第一时间获得！

![](https://github.com/lujiahao0708/PicRepo/raw/master/公众号二维码.jpg)
