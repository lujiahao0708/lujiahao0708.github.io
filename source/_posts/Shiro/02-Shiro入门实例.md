---
title: 02.Shiro入门实例
categories: Shiro
tags:
  - Shiro
  - Spring Boot
abbrlink: 9144b96d
date: 2019-10-13 20:42:34
---

本篇为Shiro入门案例，使用Spring Boot集成。

<!--more-->

## 1. 引入坐标依赖
有两种方式引入jar包
- SpringBoot starter方式（推荐）
- shiro-spring

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-starter</artifactId>
    <version>1.4.0</version>
</dependency>
```

或者
```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.4.0</version>
</dependency>
```

## 2. 创建ini文件
Shiro最基础的配置方式，存储用户身份信息（即用户名和密码）
```ini
[users]
lujiahao=123
```

## 3. 主体代码编写
这里我有点偷懒了，直接在Application的main方法写了，你可以使用单元测试的写法，这样更加专业些。

```java
@SpringBootApplication
public class ShiroIniApplication {

    public static void main(String[] args) {
        SpringApplication.run(ShiroIniApplication.class, args);

        // 1.加载配置文件，创建SecurityManger工厂
        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        // 2.获得SecurityManager实例对象
        SecurityManager securityManager = factory.getInstance();
        // 3.将SecurityManger绑定到当前运行环境中
        SecurityUtils.setSecurityManager(securityManager);
        // 4.创建当前登录主体
        Subject currentUser = SecurityUtils.getSubject();
        // 5.绑定主体登录的身份凭证（即账户密码）
        UsernamePasswordToken token = new UsernamePasswordToken("lujiahao", "123");
        // 6.主体登录
        try {
            currentUser.login(token);
            System.out.println("用户身份验证：" + currentUser.isAuthenticated());
        } catch (UnknownAccountException e) {
            System.out.println("账户信息错误：无此账户信息");
        } catch (IncorrectCredentialsException e) {
            System.out.println("密码错误！");
        } catch (Exception e) {
            e.printStackTrace();
        }
        // 7.注销登录
        currentUser.logout();
        System.out.println("用户身份验证：" + currentUser.isAuthenticated());
    }
}
```

## 4. 运行结果
正常执行：
```
用户身份验证：true
用户身份验证：false
```

账户不存在：
```
账户信息错误：无此账户信息
用户身份验证：false
```
> 修改代码中的用户名，与ini文件中不一致即可

密码错误：
```
密码错误！
用户身份验证：false
```
> 修改代码中的密码，与ini文件中不一致即可

## 参考资料
- [http://shiro.apache.org/10-minute-tutorial.html](http://shiro.apache.org/10-minute-tutorial.html)
- [https://juejin.im/post/5cff0cfc5188250d28510681](https://juejin.im/post/5cff0cfc5188250d28510681)
- [https://mrbird.cc/tags/Shiro/](https://mrbird.cc/tags/Shiro/)

## 代码获取
[https://github.com/lujiahao0708/LearnSpring](https://github.com/lujiahao0708/LearnSpring)

## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我：后端小哥，专注后端开发，希望和你一起进步！

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)
