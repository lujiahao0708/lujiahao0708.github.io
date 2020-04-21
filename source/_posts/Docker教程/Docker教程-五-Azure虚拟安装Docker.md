---
title: Docker教程(五)---Azure虚拟安装Docker
categories: JavaEE
tags:
  - Docker
  - Java
description: 解决非root用户无法运行docker命令的问题
abbrlink: 5c43d0f4
date: 2017-03-06 16:52:39
---

老大把Azure的权限给我了,哈哈,可惜我已经准备离职了,但是工作还是要继续的.

一切进行的非常顺利,登录Azure云平台,创建虚机,然后切换root权限安装docker,退出root用户,准备docker pull centos...
无法运行,卧槽!

运行docker命令弹出如下提示:

	Cannot connect to the Docker daemon. Is the docker daemon running on this host?

docker出于安全的考虑才会这样做(http://dockone.io/article/589).

## 解决方案
1. 切换root权限

		visudo -f /etc/sudoers
2. 在 `/etc/sudoers` 中添加如下内容

		azureuser  ALL=(ALL)       NOPASSWD: /usr/bin/docker

3. 退出root用户
4. 执行 `vim ~/.bashrc` 添加

		alias docker='sudo /usr/bin/docker'

5. 保存后执行

		source ~/.bashrc

此时就可以自由的使用docker命令了.
