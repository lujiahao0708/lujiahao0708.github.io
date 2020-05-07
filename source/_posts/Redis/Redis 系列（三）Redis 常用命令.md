---
title: Redis 系列（三）Redis 常用命令
categories: Redis
toc: true
tags:
  - Redis
abbrlink: 43930bbd
date: 2020-05-07 20:40:52
---

本篇主要介绍下 Redis 常用的命令，包含数据库操作和 key 相关。

<!-- more -->

# 1.常用数据库命令
## 1.1 select
切换数据库命令
```
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]>
```
> [1] 表示切换后是 1 号库。Redis 默认 16 个数据库，默认使用零号库

## 1.2 dbsize
查看当前数据库 key 的数量
```
127.0.0.1:6379> dbsize
(integer) 2
```
> 表名当前零号数据库中有 2 个 key

## 1.3 flushdb
清空当前库

```
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> dbsize
(integer) 0
```

## 1.3 flushall
清空所有库（16 个都清空）

```
127.0.0.1:6379[1]> flushall
OK
127.0.0.1:6379[1]> dbsize
(integer) 0
```

# 2.常用 key 命令
## 2.1 keys *
查看当前库中所有的 key
```
127.0.0.1:6379[1]> keys *
1) "a1"
2) "k1"
3) "a2"
```
> 生产环境不要使用，会造成服务器负载过高！

## 2.2 exists key
判断某个key是否存在，key 存在返回 1；key 不存在返回 0。

```
127.0.0.1:6379> exists k1
(integer) 1
127.0.0.1:6379> exists k2
(integer) 0
```
## 2.3 move key db
移动 key 到指定的数据库
```
127.0.0.1:6379> exists k1
(integer) 1
127.0.0.1:6379> move k1 1
(integer) 1
127.0.0.1:6379> exists k1
(integer) 0
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> exists k1
(integer) 1
```
> 上述例子中将 k1 从 0 号库移动到 1 号库

## 2.4 expire key second
为给定的key设置过期时间，单位秒。设置成功返回 1，设置失败返回 0。
```
127.0.0.1:6379[1]> expire k1 10
(integer) 1
127.0.0.1:6379[1]> expire k2 10
(integer) 0
```
## 2.5 ttl key
查看 key 还有多少秒过期，-1表示永不过期，-2表示已过期
```
127.0.0.1:6379[1]> expire k1 10
(integer) 1
127.0.0.1:6379[1]> ttl k1
(integer) 7
127.0.0.1:6379[1]> ttl k1
(integer) -2
```

## 2.6 type key
查看 key 是什么类型（Redis 中支持的5 种数据类型），如果 key 不存在，则返回 none。
```
127.0.0.1:6379[1]> type k1
none
127.0.0.1:6379[1]> set k1 v1
OK
127.0.0.1:6379[1]> type k1
string
```

# Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我：后端小哥，专注后端开发，希望和你一起进步！

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)

