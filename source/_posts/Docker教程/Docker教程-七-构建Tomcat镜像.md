---
title: Docker教程(七)---构建Tomcat镜像
categories: JavaEE
tags:
  - Docker
  - Java
description: 'Dockerfile命令实战,构建Tomcat镜像'
abbrlink: ca4ed89c
date: 2017-03-07 16:01:45
---

这次镜像构建是基于上篇的JDK镜像构建的.

## Dockerfile

	FROM lujiahao/jdk1.7
	MAINTAINER lujiahao
	ADD apache-tomcat-7.0.75.tar.gz /usr/local/
	RUN mv /usr/local/apache-tomcat-7.0.75 /usr/local/tomcat7
	WORKDIR /usr/local/tomcat7/bin
	EXPOSE 8080

## 构建命令

	docker build -t="lujiahao/tomcat7" .

## 启动容器测试

	docker run --name tomcat7 -p 8080:8080 -d lujiahao/tomcat7 ./catalina.sh run

>注意：在运行装有tomcat的容器的时候使用catalina.sh run (调试模式，在前台运行)来启动，如果使用startup.sh 的话会在后台运行，容器会认为进程down掉，容器也会自动停止。

然后在浏览器中查看能否访问tomcat主页