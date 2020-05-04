---
title: Redis 系列（二）Redis 安装
categories: Redis
toc: true
tags:
  - Redis
abbrlink: fe744d5a
date: 2020-05-04 20:40:52
---

Redis 全称：Remote Dictionary Server(远程字典服务器)，是完全开源免费的，用C语言编写的，遵守BSD协议，是一个高性能的(key/value)分布式内存数据库，基于内存运行并支持持久化的NoSQL数据库，是当前最热门的NoSql数据库之一，也被人们称为数据结构服务器。

<!--more-->

# 1.Redis 介绍

> 官方网站
> https://redis.io
> http://www.redis.cn

Redis 与其他 key - value 缓存产品有以下三个特点
- Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用
- Redis 不仅仅支持简单的 key-value 类型的数据，同时还提供 list，set，zset，hash 等数据结构的存储
- Redis 支持数据的备份，即 master-slave 模式的数据备份

# 2.Redis 安装
> Docker 方式安装 redis 是非常推荐的方式，易于维护。

## 2.1 下载镜像
Redis 镜像官网：https://hub.docker.com/_/redis/

执行命令：`docker pull redis`，等待镜像下载成功。
```
~ docker pull redis
Using default tag: latest
latest: Pulling from library/redis
68ced04f60ab: Pull complete
7ecc253967df: Pull complete
765957bf98d4: Pull complete
91fff01e8fef: Pull complete
76feb725b7e3: Pull complete
75797de34ea7: Pull complete
Digest: sha256:ddf831632db1a51716aa9c2e9b6a52f5035fc6fa98a8a6708f6e83033a49508d
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
```

## 2.2 创建并运行容器
```
~ docker run -itd --name local-redis -p 6379:6379 redis
b26758de6935aa9f7ab7e85d19fac064fced84ced322f42359741467c4068fdc
```

## 2.3 查看是否安装成功
```
~ docker logs -f local-redis
1:C 17 Mar 2020 10:17:22.157 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 17 Mar 2020 10:17:22.157 # Redis version=5.0.8, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 17 Mar 2020 10:17:22.157 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 5.0.8 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 1
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

1:M 17 Mar 2020 10:17:22.161 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1:M 17 Mar 2020 10:17:22.161 # Server initialized
1:M 17 Mar 2020 10:17:22.161 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1:M 17 Mar 2020 10:17:22.161 * Ready to accept connections
```

出现上述日志，即表示 Redis 安装成功。

# 3.工具推荐
## 3.1 redis-cli
redis 自带的命令行工具，所在目录`/usr/local/bin`

进入到容器控制台：
```
~ docker exec -it local-redis /bin/bash
```
链接 redis-cli 并使用 ping 命令：
```
root@b26758de6935:~# /usr/local/bin/redis-cli
127.0.0.1:6379> ping
PONG
```

## 3.2 Redis Desktop Manager
> 官方网站：https://redisdesktop.com/

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Redis/Redis%E5%9F%BA%E7%A1%80/Redis%20Desktop%20Manger.png)

> 此工具在 Mac 上会使用高性能显卡，比较费电，还有点卡，不推荐。

## 3.3 RedisInsight
> 官方网站：https://redislabs.com/redisinsight/

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Redis/Redis%E5%9F%BA%E7%A1%80/redisinsight.png)

> 推荐使用，控制台是网页，反正也要打开浏览器的，多开一个标签页也无所谓，而且颜值还不错！

# 4.其他
> Redis 默认安装目录：`/usr/local/bin`

首先查看下`/usr/local/bin`目录中的内容：
```
root@b26758de6935:~# ls -al /usr/local/bin/
total 24580
drwxr-xr-x 1 root root     4096 Mar 13 20:21 .
drwxr-xr-x 1 root root     4096 Feb 24 00:00 ..
-rwxrwxr-x 1 root root      374 Mar 13 20:20 docker-entrypoint.sh
-rwxr-xr-x 1 root root  2294944 Oct 15  2018 gosu
-rwxr-xr-x 1 root root  5940736 Mar 13 20:21 redis-benchmark
lrwxrwxrwx 1 root root       12 Mar 13 20:21 redis-check-aof -> redis-server
lrwxrwxrwx 1 root root       12 Mar 13 20:21 redis-check-rdb -> redis-server
-rwxr-xr-x 1 root root  6484736 Mar 13 20:21 redis-cli
lrwxrwxrwx 1 root root       12 Mar 13 20:21 redis-sentinel -> redis-server
-rwxr-xr-x 1 root root 10424488 Mar 13 20:21 redis-server
```
下面对主要文件进行介绍：
- docker-entrypoint.sh：docker 启动脚本
- redis-benchmark：性能测试工具
- redis-check-aof：修复有问题的 AOF 文件
- redis-check-dump：修复有问题的 dump.rdb 文件
- redis-cli：客户端链接脚本
- redis-sentinel：redis 集群脚本
- redis-server：redis 服务器启动脚本

# Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我：后端小哥，专注后端开发，希望和你一起进步！

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)

