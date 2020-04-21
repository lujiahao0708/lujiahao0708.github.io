---
title: 03-Shiro自定义Realm及加密
categories: Shiro
tags:
  - Shiro
  - Spring Boot
abbrlink: 696c7113
date: 2019-10-14 20:42:34
---

Shiro的ini方式配置用户名密码，灵活性差，安全性不佳。本文介绍自定义Realm及加密，可以安全的获取数据认证。

<!--more-->

Shiro从Realm获取安全数据(如用户、角色、权限)，就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法。也需要从Realm得到用户相应的角色 / 权限进行验证用户是否能进行操作，可以把Realm看成 DataSource，即安全数据源。

# 1. 自定义Realm
## 1.1 新建MyRealm类

新建`MyRealm`并继承`AuthorizingRealm`，重写`getName`，`doGetAuthorizationInfo`，`doGetAuthenticationInfo`三个方法。
```java
/**
 * 自定义realm实现
 *
 * @author lujiahao
 * @date 2019/10/14
 */
public class MyRealm extends AuthorizingRealm {

    @Override
    public String getName() {
        return "myRealm";
    }

    /**
     * 用户权限和角色
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    /**
     * 用户认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        // 获取用户信息
        String username = (String) token.getPrincipal();

        // 模拟从数据库获取用户名和密码
        String dbUsername = "lujiahao";
        String dbPassword = "123";
        // 验证用户
        if (!dbUsername.equals(username)) {
            return null;
        }
        // 参数列表依次是： 用户名  密码  当前realm名字
        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(username, dbPassword, getName());
        return info;
    }
}
```

## 1.2 配置文件

新建`shiro-realm.ini`配置文件
```ini
# 声明一个realm
myRealm=com.lujiahao.realms.MyRealm
# 指定securitymanager的realms实现
securityManager.realms=$myRealm
```

## 1.3 测试类
```java
/**
 * 测试自定义realm
 */
@Test
public void testCustomRealm() {
    // 1.加载配置文件，创建SecurityManger工厂
    Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro-realm.ini");
    // 2.获得SecurityManager实例对象
    SecurityManager securityManager = factory.getInstance();
    // 3.将SecurityManger绑定到当前运行环境中
    SecurityUtils.setSecurityManager(securityManager);
    // 4.创建当前登录主体
    Subject currentUser = SecurityUtils.getSubject();
    // 5.绑定主体登录的身份凭证（即账户密码）
    UsernamePasswordToken token = new UsernamePasswordToken("lujiahao222", "123");
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
```


# 2. 加密

Shiro提供了base64和16进制字符串编码 / 解码的API支持，方便一些编码解码操作。Shiro内部的一些数据的存储及表示都使用 base64和16进制字符串。
散列算法是一种不可逆的算法，一般用于存储密码之类的数据，常见的散列算法如MD5、SHA等等。

## 2.1 新建EncryRealm类

套路同上😏
```java
/**
 * 加密
 * @author lujiahao
 * @date 2019/10/14
 */
public class EncryRealm extends AuthorizingRealm {
    @Override
    public String getName() {
        return "encryRealm";
    }

    /**
     * 用户权限和角色
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    /**
     * 用户认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        // 获取用户信息
        String username = (String) token.getPrincipal();

        // 模拟从数据库获取用户名和密码
        String dbUsername = "lujiahao";
        // 模拟db中的密码：密码+盐+散列次数
        String dbPassword = "6b74111749b5409fa32e05b02816b760";
        // 验证用户
        if (!dbUsername.equals(username)) {
            return null;
        }

        // 参数列表依次是： 用户名  密码  盐  当前realm名字
        // 这里传入的密码一定是经过md5的，不然会报异常
        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(username, dbPassword, ByteSource.Util.bytes(username), getName());
        return info;
    }
}
```

## 2.2 配置文件
新建`shiro-encry-realm.ini`配置文件
```ini
[main]
#定义凭证匹配器
credentialsMatcher = org.apache.shiro.authc.credential.HashedCredentialsMatcher
#散列算法
credentialsMatcher.hashAlgorithmName = md5
#散列次数
credentialsMatcher.hashIterations = 2

# 声明一个realm
myRealm=com.lujiahao.realms.EncryRealm
#将凭证匹配器设置到realm
myRealm.credentialsMatcher = $credentialsMatcher
# 指定securitymanager的realms实现
securityManager.realms=$myRealm
```

## 2.3 测试类
```java
/**
 * 测试自定义加密realm
 */
@Test
public void testCustomEncryRealm() {
    // 1.加载配置文件，创建SecurityManger工厂
    Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro-encry-realm.ini");
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
```

## Tips
```java
SimpleAuthenticationInfo info = new SimpleAuthenticationInfo
(username, dbPassword, ByteSource.Util.bytes(username), getName());
```
中的第二个参数需要是经过md5加密的字符串，不然会报异常，异常信息如下：
```java
org.apache.shiro.authc.AuthenticationException: Authentication failed for token submission [org.apache.shiro.authc.UsernamePasswordToken - lujiahao, rememberMe=false].  Possible unexpected error? (Typical or expected login exceptions should extend from AuthenticationException).
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
    at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
    at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
    at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
    at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
    at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
    at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
    at org.apache.shiro.authc.AbstractAuthenticator.authenticate(AbstractAuthenticator.java:214)
    at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
    at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
    at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
    at org.apache.shiro.mgt.AuthenticatingSecurityManager.authenticate(AuthenticatingSecurityManager.java:106)
    at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
    at org.apache.shiro.mgt.DefaultSecurityManager.login(DefaultSecurityManager.java:274)
...
Caused by: java.lang.IllegalArgumentException: Odd number of characters.
    at org.apache.shiro.codec.Hex.decode(Hex.java:128)
    at org.apache.shiro.codec.Hex.decode(Hex.java:107)
    at org.apache.shiro.codec.Hex.decode(Hex.java:95)
    at org.apache.shiro.authc.credential.HashedCredentialsMatcher.getCredentials(HashedCredentialsMatcher.java:353)
    at org.apache.shiro.authc.credential.HashedCredentialsMatcher.doCredentialsMatch(HashedCredentialsMatcher.java:380)
    at org.apache.shiro.realm.AuthenticatingRealm.assertCredentialsMatch(AuthenticatingRealm.java:600)
    at org.apache.shiro.realm.AuthenticatingRealm.getAuthenticationInfo(AuthenticatingRealm.java:581)
    at org.apache.shiro.authc.pam.ModularRealmAuthenticator.doSingleRealmAuthentication(ModularRealmAuthenticator.java:180)
    at org.apache.shiro.authc.pam.ModularRealmAuthenticator.doAuthenticate(ModularRealmAuthenticator.java:267)
    at org.apache.shiro.authc.AbstractAuthenticator.authenticate(AbstractAuthenticator.java:198)
    ... 26 more
```

## 参考资料
- [http://shiro.apache.org/10-minute-tutorial.html](http://shiro.apache.org/10-minute-tutorial.html)
- [https://blog.csdn.net/weixin_38278878/article/details/81054672](https://blog.csdn.net/weixin_38278878/article/details/81054672)
- [https://blog.csdn.net/qq_35981283/article/details/78559375](https://blog.csdn.net/qq_35981283/article/details/78559375)
- [https://mrbird.cc/tags/Shiro/](https://mrbird.cc/tags/Shiro/)

## 代码获取
[https://github.com/lujiahao0708/LearnSpring](https://github.com/lujiahao0708/LearnSpring)

## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)
