---
layout: post
title: emall商城-面向接口编程实践
date: 2017-10-22 20:43:17
categories: JavaEE
tags:
  - JavaEE
  - 商城
description: 面向接口编程实践-缓存实现
---

## 1. 面向接口编程

    在一个面向对象的系统中，系统的各种功能是由许许多多的不同对象协作完成的。在这种情况下，
    各个对象内部是如何实现自己的,对系统设计人员来讲就不那么重要了；而各个对象之间的协作关
    系则成为系统设计的关键。小到不同类之间的通信，大到各模块之间的交互，在系统设计之初都
    是要着重考虑的，这也是系统设计的主要工作内容。面向接口编程就是指按照这种思想来编程。

上述内容来自[百度百科](https://baike.baidu.com/item/%E9%9D%A2%E5%90%91%E6%8E%A5%E5%8F%A3%E7%BC%96%E7%A8%8B/6025286?fr=aladdin)(说实话我没看懂).

## 2. 需求

emall商城中有用户登录后需要将用户的token信息在服务端保存一份,实现这个功能有两个思路:

- Redis
- Guava的LoadingCache

此处就需要将存储token的这一逻辑抽象成接口,两种方式分别实现此接口,从而达到业务逻辑与底层实现分离.

## 3. 实现
### 3.1 抽象`ILocalCache`接口

```java
/**
 * 本地缓存
 * @author lujiahao
 * @version 1.0
 * @date 2017-10-20 17:24
 */
public interface ILocalCache<T> {
    /**
     * 设置缓存
     * @param key
     * @param value
     * @return
     */
    boolean setCache(String key, T value);

    /**
     * 删除缓存
     * @param key
     * @return
     */
    boolean cleanCache(String key);

    /**
     * 获取缓存
     * @param key
     * @return
     */
    Object getCache(String key);
}
```
### 3.2 Redis实现方式
```java
/**
 * Redis实现缓存
 * @author lujiahao
 * @version 1.0
 * @date 2017-10-20 17:33
 */
public class RedisCacheImpl<T> implements ILocalCache<T> {
    public static final Logger LOGGER = LoggerFactory.getLogger(RedisCacheImpl.class);

    @Autowired
    private JedisClientDao jedisClientDao;

    @Override
    public boolean setCache(String key, T value) {
        // 把用户信息写入redis
        jedisClientDao.set(Const.CACHE_TOKEN + ":" + key, JsonUtils.objectToJson(value));
        // 设置session过期时间
        jedisClientDao.expire(Const.CACHE_TOKEN + ":" + key, Const.CACHE_TOKEN_EXPIRE);
        return true;
    }

    @Override
    public boolean cleanCache(String key) {
        try {
            // 根据token从redis中查询用户信息
            String json = jedisClientDao.get(Const.CACHE_TOKEN + ":" + key);
            if (StringUtils.isNoneBlank(json)) {
                // 更新过期时间--清除key
                jedisClientDao.expire(Const.CACHE_TOKEN + ":" + key, 0);
            }
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    @Override
    public Object getCache(String key) {
        try {
            // 根据token从redis中查询用户信息
            String json = jedisClientDao.get(Const.CACHE_TOKEN + ":" + key);
            if (StringUtils.isBlank(json)) {
                return ServerResponse.build(400, "此Session已经过期,请重新登录");
            }
            // 更新过期时间
            jedisClientDao.expire(Const.CACHE_TOKEN + ":" + key, Const.CACHE_TOKEN_EXPIRE);
            // 返回用户信息
            EmallUser emallUser = JsonUtils.jsonToPojo(json, EmallUser.class);
            return ServerResponse.success(emallUser);
        } catch (Exception e) {
            return ServerResponse.error("无法获取用户信息");
        }
    }
}
```
### 3.3 Guava的LoadingCache实现方式
```java
/**
 * Guava实现缓存
 * @author lujiahao
 * @version 1.0
 * @date 2017-10-20 17:33
 */
public class GuavaCacheImpl<T> implements ILocalCache<T> {
    public static final Logger LOGGER = LoggerFactory.getLogger(GuavaCacheImpl.class);

    // LRU算法
    private static LoadingCache<String, Object> localCache = CacheBuilder.newBuilder().initialCapacity(1000)
            .maximumSize(10000).expireAfterAccess(12, TimeUnit.HOURS)
            .build(new CacheLoader<String, Object>() {
                // 默认的数据加载实现,当调用get取值是,如果没有key,就执行这个方法
                @Override
                public Object load(String s) throws Exception {
                    return null;
                }
            });

    @Override
    public boolean setCache(String key, T value) {
        try {
            localCache.put(key, value);
        } catch (Exception e) {
            return false;
        }
        return true;
    }

    @Override
    public boolean cleanCache(String key) {
        try {
            localCache.invalidate(key);
        } catch (Exception e) {
            return false;
        }
        return true;
    }

    @Override
    public Object getCache(String key) {
        try {
            return localCache.get(key);
        } catch (Exception e) {
            LOGGER.error("========== localCache get error ==========", e);
        }
        return null;
    }
}
```
### 3.4 `applicationContext.xml`中配置
```xml
<!--根据具体实现采用缓存实现方式-->
<!--<bean class="com.lujiahao.sso.utils.cache.RedisCacheImpl"/>-->
<bean class="com.lujiahao.sso.utils.cache.GuavaCacheImpl"/>
```
### 3.5 代码中使用
```java
@Service
public class IUserServiceImpl implements IUserService {
    private static final Logger LOGGER = LoggerFactory.getLogger(IUserServiceImpl.class);

    @Autowired
    private EmallUserMapper emallUserMapper;

    @Autowired
    private ILocalCache iLocalCache;

    @Override
    public ServerResponse userLogin(String username, String password) {
        try {
            String md5Password = DigestUtils.md5DigestAsHex(password.getBytes());
            EmallUser emallUser = emallUserMapper.userLogin(username, md5Password);
            if (emallUser == null) {
                // 不能返回没有此用户名 没用用户名也返回这个信息是因为防止猜测用户名
                return ServerResponse.error("用户名或密码错误");
            }
            // 保存用户信息前先把密码清除,为了安全起见
            emallUser.setPassword(StringUtils.EMPTY);
            String token = UUID.randomUUID().toString();
            // 这里采用接口编程的方式,到底用redis还是用guava
            boolean isSaveSuccess = iLocalCache.setCache(token, emallUser);
            if (isSaveSuccess) {
                return ServerResponse.success(token);
            } else {
                LOGGER.info("========== 用户信息保存缓存失败 ==========");
                return ServerResponse.error("登录失败,请重试!");
            }
        } catch (Exception e) {
            ExceptionUtil.getStackTrace(e);
            return ServerResponse.error("服务器异常");
        }
    }
}
```
在代码中使用的时候,将接口`ILocalCache`注入,在`applicationContex.xml`中根据具体要求配置不同的
bean,由此就可以实现将缓存业务与缓存实现解耦.

## 4. 总结
我理解的面向接口编程就是将业务中的需求抽取出公共的几种方式或步骤,底层由不同的类来实现这个接口,
由此达到解耦的目的.个人理解,欢迎大家拍砖^_^.