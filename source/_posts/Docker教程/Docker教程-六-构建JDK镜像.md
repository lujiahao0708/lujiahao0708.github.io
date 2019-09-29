---
title: Docker教程(六)---构建JDK镜像
date: 2017-03-06 16:54:23
categories: JavaEE
tags:
- Docker
- Java
description: Dockerfile命令实战,构建JDK1.7镜像
---

## 1.创建文件夹jdk

	因为会把当前构建目录中的内容添加到镜像里面  所以要单独创建一个文件夹

## 2.编写Dockerfile文件

	FROM centos
	MAINTAINER lujiahao
	ADD jdk-7u80-linux-x64.tar.gz /usr/local
	RUN mv /usr/local/jdk1.7.0_80 /usr/local/jdk1.7
	ENV JAVA_HOME /usr/local/jdk1.7
	ENV JRE_HOME /usr/local/jdk1.7/jre
	ENV CLASSPATH .:$JAVA_HOME/jre/lib/rt.jar  :$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
	ENV PATH $JAVA_HOME/bin:$PATH

## 3.构建镜像

	docker build -t="lujiahao/jdk1.7" .

## 4.构建完成之后启动一个临时性容器来测试

	docker run --rm -it lujiahao/jdk1.7 /bin/bash

	在运行java和javac看看,java -version看版本

## 相关资源
可以到 [`DockerfileCollection`](https://github.com/chiahaolu/DockerfileCollection) 中寻找对应的 `Dockerfile` 文件