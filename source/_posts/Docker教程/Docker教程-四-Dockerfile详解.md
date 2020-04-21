---
title: Docker教程(四)---Dockerfile命令详解
categories: JavaEE
tags:
  - Docker
  - Java
description: Dockerfile命令总结
abbrlink: b603c996
date: 2017-02-28 20:55:40
---

## 深入docker镜像
拉取镜像文件到本地

	docker pull centos
查看本地镜像：

	docker images 
查找镜像

	docker search [镜像名称]
构建镜像
	
	1. 使用docker commit 命令	  用户名:lujiahao
		docker commit 容器名称/ID lujiahao/centos-tomcat
		docker commit -m="describe" --author="lujiahao" 容器名称/ID  lujiahao/centos-tomcat:test
		docker inspect lujiahao/centos-tomcat:test
	2. 使用docker build命令和Dockerfile文件(推荐)
		docker build  -t=“用户名/镜像名称" .
		docker build  -t="crxy/centos" .
		点表示会在当前目录中找Dockerfile文件
		建议创建镜像的时候单独一个目录,然后在写Dockerfile,这个目录叫构建目录.如果目录下面有其他文件,会把目录下所有文件发送给docker的守护进程
查看构建的步骤和层级

	docker history 用户ID/镜像名

## 执行流程
- dockerfile中的指令会按照从上到下执行
- docker首先从基础镜像运行一个容器
- 执行一条指令，对容器进行修改
- 执行类似docker commit的操作，提交一个新的镜像层
- docker再基于刚提交的镜像运行一个新容器
- 然后执行dockerfile中的下一条指令，直到所有指令都执行完毕

## Dockerfile指令详解
- 所有指令都必须为大写字母
- 以#开头的都认为是注释

### FROM

	第一条指令必须是`FROM`(指定使用的基础镜像)
### MAINTAINER

	指定镜像的作者信息
### RUN

	指定镜像被构建时要运行的命令,默认会使用/bin/bash -c 执行RUN 后面指定的命令
	也可以使用RUN  ["mkdir","a.txt"]
### EXPOSE

	告诉docker该容器需要对外开放的端口
### CMD
	
	指定一个容器运行时要执行的命令 
	docker run -i -t lujiahao/cenos-tomcat /bin/bash
	等于在Dockerfile中添加 CMD ["/bin/ps"]
	注意：使用 docker run 命令可以覆盖CMD指令
### ENTRYPOINT

	与CMD类似，但是ENTRYPOINT指定的命令不会被docker run指定的命令覆盖，
	它会把docker run命令行中指定的参数传递给ENTRYPOINT指令中指定的命令当作参数。
	ENTRYPOINT ["/bin/ps"]
	docker run -t -i lujiahao/centos  -l
	ENTRYPOINT 还可以和CMD组合一块来用，可以实现，当用户不指定参数的时候使用默认的参数执行，如果用户指定的话就使用用户提供的参数.
	ENTRYPOINT ["/bin/ps"]
	CMD ["-h"]
	注意：如果确实要覆盖默认的参数的话，可以在docker run中指定--entrypoint进行覆盖
### WORKDIR
	WORKDIR /etc
	RUN touch a.txt
	WORKDIR /usr/local
	RUN touch b.txt
	注意：可以在docker run命令中使用-w覆盖工作目录，但是不会影响之前在工作目录中执行的命令

### ENV
	设置环境变量，在这设置的环境变量可以在后面的指令中使用，使用$引用
	并且这个环境变量也会被持久保存到从我们的镜像创建的任何容器中。可以使用env命令查看
	也可以使用docker run 命令行的-e参数来传递环境变量，这些变量只在运行时有效 -e JAVA_HOME=/usr/local
### USER(了解)
	可以指定以什么用户去运行这个容器
	USER ftp（用户信息：查看/etc/group）
	还可以使用其他组合方式
	USER 用户名/UID:组/GID（用户名/UID和组/GID可以单独指定或者组合）
	不指定的话默认就是root用户
### ADD
	ADD指令用来将构建目录下的指定文件复制到镜像中(注意只能是构建目录下面的文件)
	ADD a.txt /etc/a.txt
	ADD 在添加压缩文件的时候会把压缩文件自动解压。(gzip bzip2 xz),如果目的位置已经存在了和压缩文件同名的文件或者目录，那么这些文件将会被覆盖。
### COPY
	COPY和ADD类似，但是COPY命令不会对压缩文件进行解压，只负责复制。
	如果目的位置不存在，docker会自动创建所有需要的目录，就像使用make -p一样
	注意：使用COPY和ADD的命令的时候，复制的源文件必须在当前构建环境之内，也就是和Dockerfile同一目录，否则会找不到。
### ONBUILD
	这个指令能为镜像添加触发器，当一个镜像被用作其他镜像的基础镜像时，
	该镜像中的触发器将会执行。这个触发器所指定的命令会在FROM指令之后执行。
	ONBUILD ADD . /usr/local
	先在一个镜像中添加这个触发器，再使用这个镜像作为基础镜像创建一个镜像，
	会发现在FROM指令之后就开始执行基础镜像中设置的触发器中的指令了。启动一个容器可以查看效果。
	注意：触发器只能被继承一次。
	有一些命令是不能在ONBUILD中指定的。包括FROM,MAINTAINER,ONBUILD,这样是为了防止在构建过程中产生递归调用的问题。
### .dockerignore
	在执行build的时候忽略掉不需要的文件。
	在Dockerfile同目录下创建.dockerignore,将要忽略的文件填到里面即可.

### VOLUME
	可以理解为设置(共享目录，磁盘挂载),添加到容器中,并没有真正提交到镜像文件中
	这个指令用来向基于镜像创建的容器添加卷，一个卷可以存在于一个或者多个容器内的特定的目录。
	卷的特点
		1：卷可以在容器间共享和重用
		2：对卷的修改是立刻生效的
		3：对卷的修改不会对更新镜像产生影响
		4：卷会一直存在直到没有任何容器再使用它。
	卷功能可以让我们将一些数据添加到镜像中而不是将将这个内容提交到镜像中。并且允许我们在多个容器间共享这些内容。
		格式：VOLUME  ["/usr/projrct"]
		还可以一次指定多个VOLUME ["/usr/project","/data"]
	创建数据卷
		VOLUME  ["/usr/projrct"]
		或者在启动容器的时候创建docker run -it -v /webapp centos /bin/bash
		注意：默认情况下，如果只是声明数据卷而没有映射到宿主机上的具体目录，docker会在/var/lib/docker/vfs/dir/
			目录下分配一个具有唯一名字的目录给该数据卷。通过docker inspect 命令可以查看具体路径
	指定宿主主机目录作为数据卷
							宿主机目录:容器目录
		docker run -it -v /usr/local/webapp:/webapp centos /bin/bash
		注意：如果容器内部已经存在webapp目录，那么目录中的文件将会被覆盖。
	默认情况下，数据卷是具有读写权限，rw权限，可以改为只读权限,ro
		docker run -it -v /usr/local/webapp:/webapp:ro centos /bin/bash
	使用场景:
		容器里面的tomcat/webapps/目录和本地宿主机的目录对应上,本地修改代码后,
		直接重启容器即可(如果是html连重启都不用了),相当于一个动态部署.


## 镜像操作

## docker的构建缓存
> 由于每一步的构建过程都会将结果提交为镜像，所以docker的构建镜像过程就显得非常聪明，它会将之前的镜像层看作缓存。再进行重新构建时会从第一条发生了变化的指令开始，前面的都使用缓存

不使用构建缓存的两种方式:

- 可以使用 `--no-cache` 参数来指定不使用缓存功能 
- 一般会在dockerfile中使用ENV指令设置一个时间(ENV CREATE_TIME 2015-01-01),写在`FROM`下面,这样只要一改了这个就不会使用构建缓存了

## 容器端口映射
默认情况下docker不会自动打开这些端口，必须在运行容器的时候指定-P(大写)参数，这样就可以打开expose指定的所有端口。 会随机映射到宿主机的一个端口上

EXPOSE指令可以同时指定一个或多个需要对外开放的端口
	
	expose 80 8080 6379

对外开放端口的两种方式

	-P(大写) 这样的话会将dockerfile文件中EXPOSE 指令指定的所有端口一并公开
	-p(小写)可以手工指定需要对外公开的端口

如果没有使用expose指定需要对外开放的端口，还可以在启动容器的时候动态指定，使用-p(小写)参数
	
	-p 80(docker会在宿主机上随机选择一个位于49000~49900内未被使用的端口来映射到容器中的80端口上)
	-p 8080:8080(把宿主机上的8080端口映射到容器的8080端口) 前面的表示宿主机
	-p 80:80 -p 8080:8080 同时指定多个
使用`docker ps` 可以查看容器的端口分配情况 或者使用`docker port 容器ID/名称`查看

### 推送镜像到Docker Hub(需要先登录：docker login)
	docker push 用户ID/镜像名(docker 1.6以下版本这样使用)
	docker1.6需要使用下面命令才能推送
	docker tag image_id docker.io/login_name/image_name
	docker push docker.io/login_name/image_name
	推送可能会报错，cdn-registry-1.docker.io: no such host 是因为网络问题、重试几次。

### 删除镜像
	docker rmi 用户ID/镜像名
	或者docker rmi `docker images -a -q`
	删除所有未打标签的镜像
	 docker rmi -f $(docker images --filter 'dangling=true')




