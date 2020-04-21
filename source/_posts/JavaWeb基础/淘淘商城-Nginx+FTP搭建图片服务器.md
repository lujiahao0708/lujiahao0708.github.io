---
title: 淘淘商城-Nginx+FTP搭建图片服务器
categories: JavaEE
tags:
  - JavaEE
  - 淘淘商城
  - FTP
  - Linux
  - Nginx
description: Nginx+FTP搭建图片服务器
abbrlink: 5cab112c
date: 2016-11-02 12:16:07
---

# Nginx

参看[[淘淘商城-Nginx笔记](http://chiahaolu.github.io/2016/11/02/%E6%B7%98%E6%B7%98%E5%95%86%E5%9F%8E-Nginx%E7%AC%94%E8%AE%B0/)]

# FTP配置

http://blog.sina.com.cn/s/blog_a97c78020101o8fv.html

参看[[淘淘商城-搭建FTP服务器](http://chiahaolu.github.io/2016/11/02/%E6%B7%98%E6%B7%98%E5%95%86%E5%9F%8E-%E6%90%AD%E5%BB%BAFTP%E6%9C%8D%E5%8A%A1%E5%99%A8/)]

# 整合Nginx和FTP
## 1.用户登录ftp的根目录
	vi /etc/vsftpd/vsftpd.conf
添加如下内容

	# 用户登录路径
	local_root=/home/
	# 锁定用户到各自目录为其根目录
	chroot_local_user=YES
	# 用户配置目录
	user_config_dir=/etc/vsftpd/userconfig
创建`/etc/vsftpd/userconfig`

	mkdir userconfig
	cd userconfig/
配置各自用户访问根目录
	vim ftpuser
添加

	local_root=/home/ftpuser/www/
重启服务

	/etc/init.d/vsftpd restart

## 2.配置nginx root路径为ftp路径
配置Nginx主目录

	vi /usr/local/nginx/conf/nginx.conf
修改内容

	server {
	        listen       80;
	        server_name  localhost;
	
	        #charset koi8-r;
	
	        #access_log  logs/host.access.log  main;
	
	        location / {
	            root   /home/ftpuser/www/;
	            index  index.html index.htm;
	        }

此时:

	Nginx主目录:/home/ftpuser/www/

	Vsftpd.config : local_root=/home/
	Userconfig/ftpuser : local_root=/home/ftpuser/www/

由于我们使用的是自定义root路径，如果权限不够，在访问图片的时候可能会报403错误。所以我们需要在nginx.conf 里配置user root
![QQ截图20160905140758.png](https://ooo.0o0.ooo/2016/09/05/57cd0f93542d7.png)

添加如下内容

![QQ截图20160905144137.png](https://ooo.0o0.ooo/2016/09/05/57cd13bb80c3c.png)

此时上传的根目录变成了 : `/images`
资源的访问url : `http://192.168.2.11/images/a.jpg`


## 杂项
### 客户端连接FTP服务器:

![QQ截图20160905142301.png](https://ooo.0o0.ooo/2016/09/05/57cd0f681e0ef.png)
### CentOS使用命令行启动
打开/etc/inittab 文件

	vi /etc/inittab
在默认的 run level 设置中,可以看到第一行书写如:`id:5:initdefault`:(默认的 run level 等级为 5,即图形界面)

将第一行的 5 修改为 3 即可。保存文件后重启系统你就可以看见是启动的文本界面了。
