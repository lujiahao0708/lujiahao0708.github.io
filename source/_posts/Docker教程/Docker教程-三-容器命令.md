---
title: Docker教程(三)---容器命令
date: 2017-02-24 10:55:25
categories: JavaEE
tags:
- Docker
- Java
description: Docker容器命令总结
---

## Docker 容器命令

交互式操作流程
### 运行一个容器（交互式）
	docker run -i -t centos /bin/bash
	-i 表示stdin标准输入
	-t表示tty终端

第一次执行docker run命令会先从远程仓库中拉取centos的镜像
内部流程如下图

![run命令内部流程.jpg](https://ooo.0o0.ooo/2017/02/23/58aebf93e43f4.jpg)

### 退出容器
	exit

### 重命名容器
	docker rename 旧名称 新名称
	另一种方法,启动时重命名
	docker run --name crxy -t -i centos /bin/bash
	合法的容器名称正则[a-zA-Z0-9_.-]

### 查看容器列表
	docker ps [-a]  [-l]  [-n num]

### 重新启动已经停止的容器
	docker start 容器名称/容器ID
### 附着到一个容器上

	docker attach 容器名称/容器ID
		使用`exit`退出容器后,重新由`docker start`命令启动了容器
		此时可以使用这个命令重新进入到容器里面
		可以理解为从外部进入到这个容器内部
		仅仅适用于使用-it启动的容器,像下面的后台启动的是无法使用这个命令的

后台长期运行

### 创建长期运行的容器
	docker run --name lujiahao -d centos /bin/bash -c "while true;do echo hello world;sleep 1;done"

会返回该容器的长id : 55ba56913bb5666ecb352a031503b5191cc8f186558e2f22380c2adadecc7c14

我们通过`docker ps`获得的是这个长id的前十二位

docker run后面追加-d=true或者-d，那么容器将会运行在后台模式.

### 获取容器日志
	docker logs [--tail num] [-t] [-f]  容器名称/容器ID
		旧版本的日志位置/var/lib/docker/containers/ID/ID-json.log
		但是新版本的实验并没有发现这个log存在位置
		-t 是来添加时间戳的
		-f 持续输出
### 查看容器内的进程
	docker top 容器名称/容器ID
### 操作一个正在运行的容器
	在容器内部运行进程(docker1.3之后才支持)   
	docker exec -d lujiahao touch /etc/lujiahao.txt
	在后台运行的容器里面创建一个文件
	docker exec -it lujiahao /bin/bash
	连到一个在后台运行的容器,此时执行exit退出,后台容器还在运行
### 停止容器
	docker stop 容器名称/容器ID  
		向容器发送停止指令,容器自己停止,建议使用
	docker kill 容器名称/容器ID   
		直接停掉,不建议使用
### 获取容器详细信息
	docker inspect  [--format | -f] 容器名称/容器ID
		例子：docker inspect --format='{{.State.Running}}' lujiahao
		容器存储位置：/var/lib/docker/containers
### 删除容器
	docker rm 容器名称/容器ID
正在运行中的容器是无法删除的,会出现下面的提示

	Error response from daemon: You cannot remove a running container 55ba56913bb5666ecb352a031503b5191cc8f186558e2f22380c2adadecc7c14. 
	Stop the container before attempting removal or use -f
可以先停止容器,或者使用 `-f` 参数

### 批量删除正在运行中的容器
	docker rm `docker ps -a -q`
		docker ps -a -q : 这个命令用来获取所有容器的id
### 容器的导入和导出
	导出：docker export 容器ID/名称 > my_container.tar
	导入：cat mycontainer.tar | docker import - 镜像名称:标签
### 镜像的保存和加载(暂时还没讲到)
	保存：docker save 镜像ID > my_image.tar 
	加载：docker load< my_image.tar