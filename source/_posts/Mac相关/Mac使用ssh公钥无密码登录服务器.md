---
layout: post
title: Mac使用ssh公钥免密登录服务器
date: 2019-05-28 21:54:46
categories: Mac
tags:
- Mac技巧
description: 不输密码登录服务器?竟然还有这种操作!
---

> 每次登陆服务器都要输入密码,重复无用的操作让人心生厌烦。“懒人是推动社会进步的动力”,我的宗旨就是能自动的就不要手动。
下面就像大家介绍我是如何打造无密码登录服务器:

<!--more-->

## 1. 生成公钥和密钥
相信使用过git的朋友对这一部分应该不会陌生,git的公私钥配置也是这样在本地生成的,这里就不赘述了。

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Mac%E7%9B%B8%E5%85%B3/Mac%E4%BD%BF%E7%94%A8ssh%E5%85%AC%E9%92%A5%E6%97%A0%E5%AF%86%E7%A0%81%E7%99%BB%E5%BD%95%E6%9C%8D%E5%8A%A1%E5%99%A8/1.%E7%94%9F%E6%88%90%E5%85%AC%E9%92%A5%E5%92%8C%E5%AF%86%E9%92%A5.png)


## 2. 编辑ssh配置文件  
```shell
vim ~/.ssh/config
增加：
#Tencent Server
Host ts
        HostName 111.231.199.76
        User root
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/id_rsa_tencent_server
        
1. Host ts #别名，域名缩写
2. HostName 111.231.199.76 #完整的域名或ip地址
3. User root #登录该域名使用的账号名
4. PreferredAuthentications publickey #有些情况或许需要加入此句，优先验证类型ssh
5. IdentityFile ~/.ssh/id_rsa_tencent_server #本地私钥文件的路径
```

详细配置见下图:

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Mac%E7%9B%B8%E5%85%B3/Mac%E4%BD%BF%E7%94%A8ssh%E5%85%AC%E9%92%A5%E6%97%A0%E5%AF%86%E7%A0%81%E7%99%BB%E5%BD%95%E6%9C%8D%E5%8A%A1%E5%99%A8/2.%E7%BC%96%E8%BE%91ssh%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.png)

> 我的电脑里面配置了公司的gitlab和github,再加上服务器的就有三组配置了。

## 3. 放置公钥到服务器目录中
```shell
scp ~/.ssh/id_rsa_tencent_server.pub ts:~/.ssh/
```

## 4. 服务器配置公钥
```shell
mv id_rsa.pub authorized_keys
```
> 如果服务器有authorized_keys这个文件就直接覆盖。

## 5. 本地和服务器文件权限
```shell
现在为本地mac的私钥设置权限：
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa_tencent_server
设置服务器上的文件权限：
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## 6. 服务器禁用密码登录(可选)
```shell
vi /etc/ssh/sshd_config 

# 将PasswordAuthentication设置成no，
# 然后重启service sshd restart
如果你需要用密码登录，这一步也可以不设置（ssh root@111.231.199.76）
```

## 7. 登录服务器
```shell
ssh ts
```

## 8. 动图演示

下面的动图演示两种登录服务器的方法:
- ssh 用户@ip
- ssh 自定义名称

> 第一种方式需要每次手动输入密码,一旦输入错误就得重新输入,非常不方便。然而第二种方式无需每次输入密码,减少误输入的问题。通过对比不难发现第二种方式方便快捷,一劳永逸,非常推荐大家动手操作配置。


![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Mac%E7%9B%B8%E5%85%B3/Mac%E4%BD%BF%E7%94%A8ssh%E5%85%AC%E9%92%A5%E6%97%A0%E5%AF%86%E7%A0%81%E7%99%BB%E5%BD%95%E6%9C%8D%E5%8A%A1%E5%99%A8/%E4%B8%A4%E7%A7%8D%E7%99%BB%E5%BD%95%E6%96%B9%E5%BC%8F%E5%AF%B9%E6%AF%94.gif)


## 8. 参考教程

- https://bitzhi.com/2015/07/login-in-linux-servers-with-sshkey-on-osx/
- https://my.oschina.net/kmwzjs/blog/732601
- http://cssor.com/mac-ssh-auto-login-server.html

----
欢迎大家关注😁
![](https://github.com/lujiahao0708/PicRepo/raw/master/公众号二维码.jpg)

