---
title: 淘淘商城-Nginx笔记
date: 2016-11-02 10:16:07
categories: JavaEE
tags:
  - JavaEE
  - 淘淘商城
  - FTP
  - Linux
  - Nginx
description: Nginx三个功能1.http服务器(图片服务器)2.虚拟主机3.反向代理,负载均衡
---

# Nginx
## 1.介绍
Nginx("engine x")是一款是由俄罗斯的程序设计师Igor Sysoev所开发高性能的 Web和 反向代理 服务器，也是一个 IMAP/POP3/SMTP 代理服务器。
在高连接并发的情况下，Nginx是Apache服务器不错的替代品。

## 2.安装Nginx
### 2.1 安装环境
配置Nginx的安装环境:

	yum install gcc-c++
	yum install -y pcre pcre-devel
	yum install -y zlib zlib-devel
	yum install -y openssl openssl-devel

### 2.2 安装Nginx

下载nginx

	wget http://nginx.org/download/nginx-1.8.0.tar.gz
解压

	tar -zxvf nginx-1.8.0.tar.gz
进入目录

	cd nginx-1.8.0
配置configure

	./configure \
	--prefix=/usr/local/nginx \
	--pid-path=/var/run/nginx/nginx.pid \
	--lock-path=/var/lock/nginx.lock \
	--error-log-path=/var/log/nginx/error.log \
	--http-log-path=/var/log/nginx/access.log \
	--with-http_gzip_static_module \
	--http-client-body-temp-path=/var/temp/nginx/client \
	--http-proxy-temp-path=/var/temp/nginx/proxy \
	--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
	--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
	--http-scgi-temp-path=/var/temp/nginx/scgi
<font color='red'>注意：上边将临时文件目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目录</font>

编译生成Makefile

	make
安装
	
	make install

nginx安装成功截图

![nginx安装成功.png](https://ooo.0o0.ooo/2016/09/05/57cce40bca3b6.png)
## 3.Nginx常用命令
### 3.1 启动Nginx

	cd /usr/local/nginx/sbin/
	./nginx 

查询nginx进程：
![QQ截图20160905111916.png](https://ooo.0o0.ooo/2016/09/05/57cce449293ab.png)

4780是nginx主进程的进程id，4781是nginx工作进程的进程id

注意：执行./nginx启动nginx，这里可以-c指定加载的nginx配置文件，如下：
./nginx -c /usr/local/nginx/conf/nginx.conf
如果不指定-c，nginx在启动时默认加载conf/nginx.conf文件，此文件的地址也可以在编译安装nginx时指定./configure的参数（--conf-path= 指向配置文件（nginx.conf））

### 3.2 停止nginx
方式1 : 快速停止

	cd /usr/local/nginx/sbin
	./nginx -s stop
此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程。
方式2 : 完整停止(建议使用)：

	cd /usr/local/nginx/sbin
	./nginx -s quit
此方式停止步骤是待nginx进程处理任务完毕进行停止。
### 3.3 重启nginx
方式1 : 先停止再启动（建议使用）
对nginx进行重启相当于先停止nginx再启动nginx，即先执行停止命令再执行启动命令。

	./nginx -s quit
	./nginx
方式2 : 重新加载配置文件
当nginx的配置文件nginx.conf修改后，要想让配置生效需要重启nginx，使用-s reload不用先停止nginx再启动nginx即可将配置信息在nginx中生效

	./nginx -s reload

## Nginx无法站外访问？
![QQ截图20160905112152.png](https://ooo.0o0.ooo/2016/09/05/57cce4fbe4006.png)

这是一个很尴尬的场面,传智的教程里面没有提到这一部分内容.此部分内容来自互联网和我自己的实践.

刚安装好nginx一个常见的问题是无法站外访问，本机wget、telnet都正常。而服务器之外，不管是局域网的其它主机还是互联网的主机都无法访问站点。如果用telnet的话，提示：
正在连接到192.168.0.xxx...不能打开到主机的连接， 在端口 80: 连接失败
CentOS防火墙在虚拟机的CENTOS装好APACHE不能用,郁闷,解决方法如下 

	/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT 
	/sbin/iptables -I INPUT -p tcp --dport 22 -j ACCEPT 

如果还无法访问，则需配置一下Linux防火墙:

	iptables -I INPUT 5 -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
然后保存： 
	
	/etc/rc.d/init.d/iptables save 
centos 5.3，5.4以上的版本需要用 

	service iptables save 
	来实现保存到配置文件。 
这样重启计算机后,CentOS防火墙默认已经开放了80和22端口。 
这里应该也可以不重启计算机： 

	/etc/init.d/iptables restart 
CentOS防火墙的关闭，关闭其服务即可： 

	查看CentOS防火墙信息：/etc/init.d/iptables status 
	关闭CentOS防火墙服务：/etc/init.d/iptables stop 
永久关闭？不知道怎么个永久法： 

	chkconfig –level 35 iptables off。

![QQ截图20160905112441.png](https://ooo.0o0.ooo/2016/09/05/57cce58f078cf.png)

再次访问即可使用
![QQ截图20160905112534.png](https://ooo.0o0.ooo/2016/09/05/57cce5bc7e087.png)



## 4.开机自启动nginx
### 4.1 编写shell脚本
这里使用的是编写shell脚本的方式来处理

	vi /etc/init.d/nginx
输入下面的代码

	#!/bin/bash
	# nginx Startup script for the Nginx HTTP Server
	# it is v.0.0.2 version.
	# chkconfig: - 85 15
	# description: Nginx is a high-performance web and proxy server.
	#              It has a lot of features, but it's not for everyone.
	# processname: nginx
	# pidfile: /var/run/nginx.pid
	# config: /usr/local/nginx/conf/nginx.conf
	nginxd=/usr/local/nginx/sbin/nginx
	nginx_config=/usr/local/nginx/conf/nginx.conf
	nginx_pid=/var/run/nginx.pid
	RETVAL=0
	prog="nginx"
	# Source function library.
	. /etc/rc.d/init.d/functions
	# Source networking configuration.
	. /etc/sysconfig/network
	# Check that networking is up.
	[ ${NETWORKING} = "no" ] && exit 0
	[ -x $nginxd ] || exit 0
	# Start nginx daemons functions.
	start() {
	if [ -e $nginx_pid ];then
	   echo "nginx already running...."
	   exit 1
	fi
	   echo -n $"Starting $prog: "
	   daemon $nginxd -c ${nginx_config}
	   RETVAL=$?
	   echo
	   [ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
	   return $RETVAL
	}
	# Stop nginx daemons functions.
	stop() {
	        echo -n $"Stopping $prog: "
	        killproc $nginxd
	        RETVAL=$?
	        echo
	        [ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /var/run/nginx.pid
	}
	# reload nginx service functions.
	reload() {
	    echo -n $"Reloading $prog: "
	    #kill -HUP `cat ${nginx_pid}`
	    killproc $nginxd -HUP
	    RETVAL=$?
	    echo
	}
	# See how we were called.
	case "$1" in
	start)
	        start
	        ;;
	stop)
	        stop
	        ;;
	reload)
	        reload
	        ;;
	restart)
	        stop
	        start
	        ;;
	status)
	        status $prog
	        RETVAL=$?
	        ;;
	*)
	        echo $"Usage: $prog {start|stop|restart|reload|status|help}"
	        exit 1
	esac
	exit $RETVAL

然后	:wq  保存并退出(别告诉我你不会这个)

### 4.2 设置文件的访问权限

	chmod a+x /etc/init.d/nginx   (a+x ==> all user can execute  所有用户可执行)
这样在控制台就很容易的操作nginx了：查看Nginx当前状态、启动Nginx、停止Nginx、重启Nginx…

如果修改了nginx的配置文件nginx.conf，也可以使用上面的命令重新加载新的配置文件并运行，可以将此命令加入到rc.local文件中，这样开机的时候nginx就默认启动了

### 4.3 加入到rc.local文件中

	vi /etc/rc.local

加入一行  `/etc/init.d/nginx start`    保存并退出，下次重启会生效。

	#!/bin/sh
	#
	# This script will be executed *after* all the other init scripts.
	# You can put your own initialization stuff in here if you don't
	# want to do the full Sys V style init stuff.
	
	touch /var/lock/subsys/local
	
	# nginx auto start
	/etc/init.d/nginx start

# Nginx应用-反向代理(Reverse Proxy)