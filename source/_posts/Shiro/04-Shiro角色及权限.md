---
title: 04-Shiro角色及权限
date: 2019-10-15 20:42:34
categories: Shiro
tags:
- Shiro
- Spring Boot
---

授权，也叫访问控制，即在应用中控制谁能访问哪些资源(如访问页面/编辑数据/页面操作等)。

<!--more-->

## 1. 相关概念
在授权中需了解的几个关键对象：主体(Subject)、资源(Resource)、权限(Permission)、角色(Role)，其解析如下所示：

- 主体 : 即访问应用的用户，在Shiro中使用Subject代表该用户。用户只有授权后才允许访问相应的资源。
- 资源 : 在应用中用户可以访问的任何资源，比如访问JSP页面、查看/编辑某些数据、访问某个业务方法、打印文本等等用户需要授权后方可访问。
- 权限 : 安全策略中的原子授权单位,可用权限控制用户在应用中是否能访问某个资源,如访问用户列表页面,查看/新增/修改/删除用户数据(基本为CRUD式权限控制)。
- 角色 : 角色代表了操作集合,可以理解为权限的集合,一般情况下我们会赋予用户角色而不是权限,即这样用户可以拥有一组权限,不同的角色拥有一组不同的权限。
    - 隐式角色 : 即直接通过角色来验证用户有没有操作权限，即粒度是以角色为单位进行访问控制的，粒度较粗。若进行变更可能需要多处代码的修改。
    - 显示角色 : 在程序中通过权限控制谁能访问某个资源，角色聚合一组权限集合。这样若需要哪个角色不能访问某个资源，只需要从角色代表的权限集合中移除指定的访问权限即可，无须修改多处代码。

## 2. 授权方式
Shiro有三种授权方式：

### 2.1 编程方式
```java
Subject subject = SecurityUtils.getSubject();
if(subject.hasRole("admin")) {
    //有权限
} else {
    // 无权限
}
```

## 2.2 注解方式
```java
@RequiresRoles("admin")
public void hello() {
    // 有权限
}
```

## 2.3 jsp标签形式
```html
<shiro:hasRole name="admin">
// 有权限
</shiro:hasRole>
```

## 3. ini方式验证角色和权限
### 3.1 shiro-roles.ini
```
[users]
# 用户lujiahao的密码是123，拥有role1和role2两个角色
lujiahao=123,role1,role2
zhangsan=321,role3

[roles]
# 角色role1对资源user拥有create和update权限
role1=user:create,user:update
# 角色role2对资源user拥有delete权限
role2=user:delete
role3=user:update
```

配置规则：

    用户名=密码,角色1,角色2...
    角色=权限1,权限2...
    权限字符串的规则：操作:资源实例标识符 可以使用*通配符
    eg：
        用户创建权限：user:create
        用户修改实例001的权限：user:update:001

### 3.2 测试代码
```java
@Test
public void testRole() {
    // 用户登录，授权操作的前提是用户已经通过认证
    Subject subject = this.doLogin("shiro-roles.ini", "lujiahao", "123");

    System.out.println("验证用户是否拥有某个角色 true：拥有 false：没有");
    System.out.println(subject.hasRole("role1"));

    System.out.println("验证用户是否拥有所有角色 true：全部拥有 false：不全部拥有");
    System.out.println(subject.hasAllRoles(Arrays.asList("role1", "role2", "role3")));

    System.out.println("验证用户是否拥有角色，返回boolean类型数组 true：拥有 false：没有");
    System.out.println(Arrays.toString(subject.hasRoles(Arrays.asList("role1", "role2", "role3"))));


    System.out.println("验证用户是否拥有某个角色 拥有：无异常 没有：报UnauthorizedException异常");
    subject.checkRole("role1");

    System.out.println("验证用户是否拥有所有角色 拥有：无异常 没有：报UnauthorizedException异常");
    subject.checkRoles("role1", "role2");

    System.out.println("验证用户是否拥有所有角色 拥有：无异常 没有：报UnauthorizedException异常");
    subject.checkRoles(Arrays.asList("role1", "role2"));
}

@Test
public void testPermission() {
    // 用户登录，授权操作的前提是用户已经通过认证
    Subject subject = this.doLogin("shiro-roles.ini", "lujiahao", "123");


    System.out.println("验证用户是否拥有某个权限 true：拥有 false：没有");
    System.out.println(subject.isPermitted("user:list"));

    System.out.println("验证用户是否拥有所有权限 true：全部拥有 false：不全部拥有");
    System.out.println(subject.isPermittedAll("user:delete", "user:update", "user:create"));

    System.out.println("验证用户是否拥有权限，返回boolean类型数组 true：拥有 false：没有");
    System.out.println(Arrays.toString(subject.isPermitted("user:list", "user:update", "user:create")));


    System.out.println("验证用户是否拥有某个权限 拥有：无异常 没有：报UnauthorizedException异常");
    subject.checkPermission("user:update");

    System.out.println("验证用户是否拥有所有权限 拥有：无异常 没有：报UnauthorizedException异常");
    subject.checkPermissions("user:delete", "user:update", "user:create");

}
```

## 4. 自定义Realm验证角色和权限
### 4.1 shiro-permission-realm.ini
```java
# 声明一个realm
myRealm=com.lujiahao.PermissionRealm
# 指定securitymanager的realms实现
securityManager.realms=$myRealm
```

### 4.2 自定义Realm
```java
/**
 * 自定义realm实现
 *
 * @author lujiahao
 * @date 2019/10/14
 */
public class PermissionRealm extends AuthorizingRealm {

    @Override
    public String getName() {
        return "permissionRealm";
    }

    /**
     * 用户权限和角色
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        // 当前用户登录信息
        String username = (String) principalCollection.getPrimaryPrincipal();

        // 模拟数据库查询用户角色和权限
        List<String> roles = new ArrayList<>();
        List<String> permissions = new ArrayList<>();
        roles.add("role1");
        permissions.add("user:read");

        // 增加权限
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        info.addRoles(roles);
        info.addStringPermissions(permissions);

        return info;
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

### 4.3 测试代码
```java
@Test
public void testPermissionRealm() {
    // 用户登录，授权操作的前提是用户已经通过认证
    Subject subject = this.doLogin("shiro-permission-realm.ini", "lujiahao", "123");

    System.out.println(subject.isPermitted("user:read"));
    System.out.println(subject.hasRole("role1"));
}
```

## 参考资料
- [http://shiro.apache.org/10-minute-tutorial.html](http://shiro.apache.org/10-minute-tutorial.html)
- [https://mrbird.cc/tags/Shiro/](https://mrbird.cc/tags/Shiro/)

## 代码获取
[https://github.com/lujiahao0708/LearnSpring](https://github.com/lujiahao0708/LearnSpring)

## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)
