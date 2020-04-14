---
title: Docker教程(二)---安装Docker
date: 2017-02-23 14:06:49
categories: JavaEE
tags:
- Docker
- Java
description: Docker安装步骤总结
---

## Docker 核心组件

### 镜像(Image)
镜像是构建docker世界的基石,也是docker生命周期中的构建阶段.

### 仓库(Registry)
存储用户构建的镜像以及官方的镜像,分为公有和私有.
Docker公司运营的公有仓库叫做 [Docker Hub](https://hub.docker.com/)

### 容器(Container)
容器是基于镜像启动的，容器中可以运行一个或多个进程。
容器是docker生命周期中的启动或执行阶段。

> 仓库里面存储着镜像,基于镜像可以启动容器<br/>
> container = image + docker run

## Docker工作流
![dockerworkflow.jpg](https://ooo.0o0.ooo/2017/02/23/58ae7759b7e41.jpg)

## 安装Docker
> 宿主机Centos7 64位,因为 Docker 是基于64位系统构建的.

1. 验证linux内核版本`uname -a`,官方建议使用3.8版本以上
2. 检查`Device Mapper(Docker的存储驱动)`

		grep device-mapper /proc/devices
		如果不存在则安装yum install -y device-mapper
		加载dm_mod内核模块 modprobe dm_mod
3. 安装

		yum -y install docker(默认安装最新版)

4. 安装指定版本 Docker

		查看可安装的版本
			yum makecache fast(相当于更新本地缓存)
			yum list docker --showduplicates
		安装指定版本
			yum install 2:1.12.5-14.el7.centos

5. 验证
	
		启动docker
			systemctl start docker(centos7)
		添加开机启动项
			systemctl enable docker(centos7)
		验证docker是否正常
			systemctl status  docker
			docker info
出现下面图片表示安装成功了
![dockerinfo.jpg](https://ooo.0o0.ooo/2017/02/23/58ae7b4432591.jpg)

## 总结
至此,Docker已经顺利安装了,下面进入到基础命令学习阶段吧!

---


欢迎大家关注 : LF工作室

简书 : https://www.jianshu.com/u/e61935d18b09

掘金 : https://juejin.im/user/59239002570c350069c5f0bb

微信公众号 :
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)
头条号 :
![](https://github.com/lujiahao0708/PicRepo/raw/master/头条号二维码.jpg)

