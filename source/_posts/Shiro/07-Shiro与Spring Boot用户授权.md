---
title: 07-Shiro与Spring Boot用户授权.md
date: 2019-10-18 20:42:34
categories: Shiro
tags:
- Shiro
- Spring Boot
---

Spring Boot与Shiro整合案例，用户授权部分。

<!--more-->

Spring Boot集成Shiro主要配置两个类： `ShiroConfig`类及继承`AuthorizingRealm的Realm`类。
- ShiroConfig：是Shiro的配置，相当于Spring中的xml配置。包括：包括过滤器(ShiroFilter)、安全事务管理器(SecurityManager)、密码凭证匹配器(CredentialsMatcher)、缓冲管理器(EhCacheManager)、AOP注解支持(authorizationAttributeSourceAdvisor)、等等
- UserRealm：自定义的UserRealm继承自AuthorizingRealm，重写`doGetAuthorizationInfo`(授权认证)和`doGetAuthenticationInfo`(登陆认证)这两个方法。

## 1. 依赖
```xml
<dependencies>
    <!-- MyBatis -->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.0.1</version>
    </dependency>

    <!-- thymeleaf -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring-boot-web-starter</artifactId>
        <version>1.4.0</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.15</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

## 2. ShiroConfig
```java
@Configuration
public class ShiroConfig {
    /**
     * 过滤规则
     */
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        // 设置securityManager
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 登录的url
        shiroFilterFactoryBean.setLoginUrl("/login");
        // 登录成功后跳转的url
        shiroFilterFactoryBean.setSuccessUrl("/index");
        // 未授权url
        shiroFilterFactoryBean.setUnauthorizedUrl("/403");

        // LinkedHashMap是有序的，shiro会按照顺序依次匹配，匹配成功就不验证了，因此下面的url是较为宽松的
        // 上面的url是比较严格的   "/**"一定要放在最后
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();

        // 定义filterChain，静态资源不拦截
        filterChainDefinitionMap.put("/css/**", "anon");
        filterChainDefinitionMap.put("/js/**", "anon");
        filterChainDefinitionMap.put("/fonts/**", "anon");
        filterChainDefinitionMap.put("/img/**", "anon");

        // 配置退出过滤器，其中具体的退出代码Shiro已经替我们实现了
        filterChainDefinitionMap.put("/logout", "logout");

        // 除上以外所有url都必须认证通过才可以访问，未通过认证自动访问LoginUrl
        filterChainDefinitionMap.put("/**", "authc");

        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }

    /**
     * 安全管理器
     * 注：使用shiro-spring-boot-starter 1.4时，返回类型是SecurityManager会报错，直接引用shiro-spring则不报错
     */
    @Bean
    public DefaultWebSecurityManager securityManager(){
        DefaultWebSecurityManager securityManager =  new DefaultWebSecurityManager();
        securityManager.setRealm(userRealm());
        return securityManager;
    }

    /**
     * 自定义realm
     */
    @Bean
    public UserRealm userRealm(){
        UserRealm userRealm = new UserRealm();
        return userRealm;
    }
}
```

## 3. 自定义Realm
```java
@Slf4j
public class UserRealm extends AuthorizingRealm {

    @Autowired
    private UserMapper userMapper;

    /**
     * 获取用户角色和权限
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principal) {
        return null;
    }

    /**
     * 登录认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

        // 获取用户输入的用户名和密码
        String userName = (String) token.getPrincipal();
        String password = new String((char[]) token.getCredentials());

        log.info("用户:{} 执行登录认证操作", userName);

        // 通过用户名到数据库查询用户信息
        User user = userMapper.findByUserName(userName);

        if (user == null) {
            throw new UnknownAccountException("用户名或密码错误！");
        }
        if (!password.equals(user.getPassword())) {
            throw new IncorrectCredentialsException("用户名或密码错误！");
        }

        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, password, getName());
        return info;
    }
}
```

## 4. Service/Mapper/SQL
> 参考工程中的`data.sql`

## 5. html
登录界面：
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>登录</title>
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
</head>
<body>
<div class="login-page">
    <div class="form">
        <input type="text" placeholder="用户名" name="username" required="required"/><br/>
        <input type="password" placeholder="密码" name="password" required="required"/><br/>
        <button onclick="login()">登录</button>
    </div>
</div>
</body>
<script th:inline="javascript">
        function login() {
            var username = $("input[name='username']").val();
            var password = $("input[name='password']").val();
            $.ajax({
                type: "post",
                url: "/login",
                data: {"username": username,"password": password},
                dataType: "json",
                success: function (r) {
                    if (r.code == 0) {
                        location.href = '/index';
                    } else {
                        alert(r.msg);
                    }
                }
            });
        }
</script>
</html>
```

首页：
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
    <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
</head>
<body>
<p>你好！[[${user.username}]]</p>
<a th:href="@{/logout}">注销</a>
</body>
</html>
```


## 参考资料
- [http://shiro.apache.org/10-minute-tutorial.html](http://shiro.apache.org/10-minute-tutorial.html)
- [https://mrbird.cc/tags/Shiro/](https://mrbird.cc/tags/Shiro/)

## 代码获取
[https://github.com/lujiahao0708/LearnSpring](https://github.com/lujiahao0708/LearnSpring)

## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！！

![](https://github.com/lujiahao0708/PicRepo/raw/master/公众号二维码.jpg)
