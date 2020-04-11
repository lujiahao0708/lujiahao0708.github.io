---
title: Redis学习笔记
date: 2020-04-11 17:28:08
categories: Redis
tags: Redis
toc: true
description: Redis初级学习笔记

---

Redis（Remote Dictionary Server )，即远程字典服务，是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。从2010年3月15日起，Redis的开发工作由VMware主持。从2013年5月开始，Redis的开发由Pivotal赞助。
<!-- more -->

## 1.简介
### 1.1 定义
高性能的kv缓存和数据库.
存储结构就是kv，形式如下：
![QQ截图20160708093811.png](https://ooo.0o0.ooo/2016/07/07/577f05774afbc.png)

### 1.2 应用场景
- 用来做缓存(ehcache/memcached)——redis的所有数据是放在内存中的（内存数据库）
- 可以在某些特定应用场景下替代传统数据库
- 在一些大型系统中，巧妙地实现一些特定的功能：session共享、购物车只要你有丰富的想象力，redis可以用在各种官网说明上没有提到的场景。。。。。

### 1.3 redis的特性
- redis数据访问速度快（数据在内存中）
- redis的数据有持久化
	- 持久化机制有两种：
		- 1、定期将内存数据dump到磁盘
		- 2、aof持久化机制——用记日志的方式记录每一条数据更新操作，一旦出现灾难事件，可以通过日志重放来恢复整个数据库
- redis还支持集群模式（容量可以线性扩展）
- redis相比其他缓存工具（ehcach/memcached），有一个鲜明的优势：支持丰富的数据结构

### 1.4 Redis安装

TODO:
windows版本已经总结了
linux版本的安装总结到那篇笔记中

## 2. Redis存储的数据结构

### 2.1 String
1.插入/读取数据

	set sessionid-0001 "zhangsan"
	get sessionid-0001	 

2.对string类型数据进行增减（前提是这条数据的value可以看成数字）

	DECR key	减去1
	INCR key	加上1
	
	DECRBY key decrement	减去decrement
	INCRBY key increment	加上increment

3.一次性插入或者获取多条数据

	MSET key1 value1 key2 value2 ...	插入多条数据
	MGET key1 key2	获取多条数据


### 2.2 List
 
1.从头部插入数据

	LPUSH  key  value1  value2  value3    
2.从尾部插入数据

	RPUSH  key  value1  value2  value3   
3.读取list中的values

	LRANGE  key  start   end
	lrange  task-queue  0  -1      读取整个list
4.从头部弹出一个元素

	LPOP  key
5.从尾部弹出一个元素

	RPOP  key
6.从一个list的尾部弹出一个元素插入到另一个list

	RPOPLPUSH   key1    key2
案例:
	任务调度系统(详见后面笔记)

### 2.3 Hash
Redis中的Hashes类型可以看成具有String Key和String Value的map容器
![QQ截图20160708093902.png](https://ooo.0o0.ooo/2016/07/07/577f05a581620.png)

1.插入一条hash类型的数据

	hset key field value
 
2.获取一条hash类型数据的value

	取出一条hash类型数据中所有field-value对
	hgetall key

	取出hash数据中所有fields
	hkeys value 

	取出hash数据中所有的value
	hvals key

	取出hash数据中一个指定field的值
	hget key field

3.为hash数据中指定的一个field的值进行增减

	hincrby key field increment
	hincrby user001:zhangsan xiaomi 1

4.从hash数据中删除一个字段field及其值

	hdel key field
	hdel user001:zhangsan iphone

### 2.4 Set

1.插入一条set数据

	sadd key value value value ...
	sadd frieds:zhangsan  bingbing baby fengjie furong ruhua feifei
2.显示set数据的个数

	scard key

3.获取一条set数据的所有members

	smembers frieds:zhangsan
	1) "fengjie"
	2) "baby"
	3) "furong"
	4) "bingbing"
	5) "feifei"
	6) "ruhua"	 

4.判断一个成员是否属于某条指定的set数据

	redis 127.0.0.1:6379> sismember frieds:zhangsan liuyifei     #如果不是，则返回0
	(integer) 0
	redis 127.0.0.1:6379> sismember frieds:zhangsan baby       #如果是，则返回1
	(integer) 1	 
5.求两个set数据的差集

	求差集
		redis 127.0.0.1:6379> sdiff frieds:zhangsan friends:xiaotao
		1) "furong"
		2) "fengjie"
		3) "ruhua"
		4) "feifei"
	求差集，并将结果存入到另一个set
		redis 127.0.0.1:6379> sdiffstore zhangsan-xiaotao frieds:zhangsan friends:xiaotao
		(integer) 4

6.求交集，求并集

	求交集
		redis 127.0.0.1:6379> sinterstore zhangsan^xiaotao frieds:zhangsan friends:xiaotao
		(integer) 2
	求并集
		redis 127.0.0.1:6379> sunion frieds:zhangsan friends:xiaotao
		 1) "fengjie"
		 2) "tangwei"
		 3) "liuyifei"
		 4) "bingbing"
		 5) "ruhua"
		 6) "feifei"
		 7) "baby"
		 8) "songhuiqiao"
		 9) "furong"
		10) "yangmi"	 

### 2.5 SortedSet

1.往redis库中插入一条sortedset数据

	redis 127.0.0.1:6379> zadd nanshen:yanzhi:bang  70 liudehua  90 huangbo  100 weixiaobao  250 yanggang  59 xiaotao
	(integer) 5
	 
2.从sortedset中查询有序结果

	#正序结果
	redis 127.0.0.1:6379> zrange nanshen:yanzhi:bang 0 4
	1) "xiaotao"
	2) "liudehua"
	3) "huangbo"
	4) "weixiaobao"
	5) "yanggang"
	#倒序结果
	redis 127.0.0.1:6379> zrevrange nanshen:yanzhi:bang 0 4
	1) "yanggang"
	2) "weixiaobao"
	3) "huangbo"
	4) "liudehua"
	5) "xiaotao"	 

3.查询某个成员的名次

	在正序榜中的名次
	redis 127.0.0.1:6379> zrank nanshen:yanzhi:bang xiaotao
	(integer) 0
	
	在倒序榜中的名次
	redis 127.0.0.1:6379> zrevrank nanshen:yanzhi:bang xiaotao
	(integer) 4	 

4.修改成员的分数

	redis 127.0.0.1:6379> zincrby nanshen:yanzhi:bang 300 xiaotao
	"359"
	redis 127.0.0.1:6379> zrevrank nanshen:yanzhi:bang xiaotao
	(integer) 0	 


## 3. 实例

### 3.1 List实例--任务调度系统
生产者不断产生任务，放入task-queue排队
消费者不断拿出任务来处理，同时放入一个tmp-queue暂存，如果任务处理成功，则清除tmp-queue，否则，将任务弹回task-queue
![QQ截图20160708094036.png](https://ooo.0o0.ooo/2016/07/07/577f06042dfdc.png)
上述机制是一个简化版，真实版的任务调度系统会更加复杂，如下所示：
（增加了一个专门用来管理暂存队列的角色，以便就算消费者程序失败退出，那些处理失败的任务依然可以被弹回task-queue）
![QQ截图20160708094023.png](https://ooo.0o0.ooo/2016/07/07/577f0604350ca.png)

代码:

	/**
	 * 使用redis模拟任务调度
	 * Created by lujiahao on 2016/7/7.
	 */
	public class TaskSchedulerSystem {
	    /**
	     * 模拟一个生产者
	     */
	    public static class TaskProducer implements Runnable {
	        Jedis jedis = new Jedis("127.0.0.1",6379);
	        @Override
	        public void run() {
	            Random random = new Random();
	            while (true) {
	                try {
	                    Thread.sleep(random.nextInt(600) + 600);
	                    // 生成一个任务
	                    UUID taskid = UUID.randomUUID();
	                    // 将任务插入任务队列:task-queue
	                    jedis.lpush("task-queue",taskid.toString());
	                    System.out.println(taskid+"任务被插入");
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        }
	    }
	
	    /**
	     * 模拟一个消费者
	     */
	    public static class TaskConsumer implements Runnable{
	        Jedis jedis = new Jedis("127.0.0.1",6379);
	        @Override
	        public void run() {
	            Random random = new Random();
	            while (true){
	                try {
	                    // 从任务队列中获取一个任务,并将任务放入暂存队列tmp-queue
	                    jedis.rpoplpush("task-queue", "tmp-queue");
	                    // 模拟处理任务--sleep---业务逻辑
	                    Thread.sleep(1000);
	                    // 模拟成功和失败的偶然情况
	                    if (random.nextInt(13) % 7 == 0){
	                        // 失败,将任务从暂存队列tmp-queue弹回到任务队列task-queue
	                        String taskid = jedis.rpoplpush("tmp-queue", "task-queue");
	                        System.out.println(taskid+"任务处理失败,弹回任务队列");
	                    } else {
	                        // 成功,将任务从暂存队列中清除
	                        String taskid = jedis.rpop("tmp-queue");
	                        System.out.println(taskid+"任务处理成功,从暂存队列中清除");
	                    }
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        }
	    }
	    public static void main(String[] args) throws InterruptedException {
	        // 启动一个生产者线程,产生模拟任务
	        new Thread(new TaskProducer()).start();
	        Thread.sleep(500);
	        // 启动一个消费者线程,模拟任务的处理
	        new Thread(new TaskConsumer()).start();
	        // 让主线程休眠
	        Thread.sleep(Integer.MAX_VALUE);
	    }
	}

### 3.2 SortedSet实例--LOL英雄数据排行榜
	/**
	 * LOL盒子英雄数据排行榜的模拟实现
	 * Created by lujiahao on 2016/7/7.
	 */
	public class LOLHerosTopN {
	    private static class PlayerTask implements Runnable {
	        String[] heros = {"压缩", "剑圣", "蛮王", "男枪", "女警", "阿狸", "猴子", "赵信", "盖伦"};
	        Jedis jedis = new Jedis("127.0.0.1", 6379);
	        @Override
	        public void run() {
	            Random random = new Random();
	            while (true) {
	                String hero = heros[random.nextInt(heros.length)];
	                try {
	                    Thread.sleep(1000);
	                    jedis.zincrby("lol:hero:rank", 1, hero);
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        }
	    }
	    private static class ViewerTask implements Runnable {
	        Jedis jedis = new Jedis("127.0.0.1", 6379);
	        int i = 1;
	        @Override
	        public void run() {
	            while (true) {
	                System.out.println("第"+i+"次获取榜单:");
	                try {
	                    Thread.sleep(2000);
	                    Set<Tuple> herosAndScore = jedis.zrevrangeWithScores("lol:hero:rank", 0, -1);
	                    for (Tuple tuple : herosAndScore) {
	                        System.out.println(tuple.getElement()+":"+tuple.getScore());
	                    }
	                    i++;
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        }
	    }
	    public static void main(String[] args) throws InterruptedException {
	        new Thread(new PlayerTask()).start();
	        Thread.sleep(1000);
	        new Thread(new ViewerTask()).start();
	        Thread.sleep(Long.MAX_VALUE);
	    }
	}
