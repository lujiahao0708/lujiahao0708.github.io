---
layout: post
title: Mac 多 Git 账户配置
date: 2020-04-20 10:40:52
categories: Mac
toc: true
tags:
- Mac
- Git

---
通常公司代码一般托管在公司自建 Gitlab 服务上，自己的代码托管在 GitHub 或者 Coding 这样的网站上。Git 账户经常切换非常不方便，这就需要配置多个 Git 账户，以向不同的网站 push 代码。本文将介绍如何在 Mac 上配置多个 Git 账户，以及快速切换管理。

<!--more-->
# 1.生成密钥
Mac 中密钥文件都保存在 `/Users/你的用户名/.ssh` 目录下，如果没有该目录手动创建即可。进入到该目录后根据邮箱生成密钥：`ssh-keygen -t rsa -C ‘邮箱账号'`

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Mac%E7%9B%B8%E5%85%B3/Mac%20%E5%A4%9A%20Git%20%E8%B4%A6%E6%88%B7%E9%85%8D%E7%BD%AE/%E7%94%9F%E6%88%90%E7%A7%98%E9%92%A5.png)

> 注意红框处 `id_rsa_testgit` ，此处是指定密钥名称，可以根据不同的账号命名，方便区分。如果不指定名称会使用默认值：`id_rsa`，再次生成密钥会覆盖上一次的密钥，因此每次都要指定不同的名称。名称输入完后一直回车直到密钥生成。

使用 `ssh-keygen` 命令，分别生成 GitHub 和 Coding 的密钥。每组密钥其中以.pub结尾的是公钥，另一个是私钥。查看生成的所有密钥：
```
~/.ssh  ls -l
rw-------  lujiahao  staff     2 KiB  Sun Apr 19 20:11:54 2020  id_rsa_coding
rw-r--r--  lujiahao  staff   570 B    Sun Apr 19 20:11:54 2020  id_rsa_coding.pub
rw-------  lujiahao  staff     1 KiB  Sun Jun 24 22:04:49 2018  id_rsa_github
rw-r--r--  lujiahao  staff   404 B    Sun Jun 24 22:04:49 2018  id_rsa_github.pub
rw-------  lujiahao  staff     1 KiB  Wed May  8 10:01:00 2019  id_rsa_gitlab
rw-r--r--  lujiahao  staff   405 B    Wed May  8 10:01:00 2019  id_rsa_gitlab.pub
```

# 2.托管网站配置公钥
## 2.1 GitHub 配置
登录 GitHub，按照顺序到达指定页面 `Settings --> SSH and GPG keys --> New SSH Key`。
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Mac%E7%9B%B8%E5%85%B3/Mac%20%E5%A4%9A%20Git%20%E8%B4%A6%E6%88%B7%E9%85%8D%E7%BD%AE/GitHub%20%E9%85%8D%E7%BD%AE%E5%85%AC%E9%92%A51.png)

在命令行中执行命令查看并复制公钥内容：`cat ~/.ssh/id_rsa_github.pub`。
在打开的页面的 Key 输入框中粘贴刚刚复制的公钥，Title 自定义填写，然后点击下方的 Add SSH key 按钮保存公钥：
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Mac%E7%9B%B8%E5%85%B3/Mac%20%E5%A4%9A%20Git%20%E8%B4%A6%E6%88%B7%E9%85%8D%E7%BD%AE/GitHub%20%E9%85%8D%E7%BD%AE%E5%85%AC%E9%92%A52.png)


## 2.2 Coding 配置
Coding 的配置与 GitHub 配置基本相似，查看并复制公钥内容：`cat ~/.ssh/id_rsa_coding.pub`。
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Mac%E7%9B%B8%E5%85%B3/Mac%20%E5%A4%9A%20Git%20%E8%B4%A6%E6%88%B7%E9%85%8D%E7%BD%AE/Coding%20%E9%85%8D%E7%BD%AE%E5%85%AC%E9%92%A51.png)

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Mac%E7%9B%B8%E5%85%B3/Mac%20%E5%A4%9A%20Git%20%E8%B4%A6%E6%88%B7%E9%85%8D%E7%BD%AE/Coding%20%E9%85%8D%E7%BD%AE%E5%85%AC%E9%92%A52.png)

> 可以勾选永久有效，避免过期失效重复操作。当然这样操作会降低一定的安全性，根据自己的情况选择即可。

# 3.本地配置私钥
> SSH 原理：在托管网站使用公钥，在本地使用私钥，通过公钥和私钥的认证，本地仓库就可以和远程仓库进行通信。

在 `.ssh` 目录下新建 `config` 文件 `vim config`，将下面内容添加到该文件中：
```
#github
Host github.com
        HostName github.com
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/id_rsa_github
#coding
Host sjzcode.coding.net
        HostName sjzcode.coding.net
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/id_rsa_coding
```
`config` 文件中各个字段的含义：

        Host : 配置别名
        HostName : git服务器地址
        IdentityFile : 私钥文件位置

> 如果你配置的 Coding ，需要注意 `HostName` 要填写你的团队的地址，Coding 现在对于新用户都是注册为团队，并为每个团队单独创建了一个二级域名，比如我的账户二级域名：`sjzcode.coding.net`。直接写 `coding.net` 无法认证成功。

# 4.验证
命令 `ssh -T git@别名` 验证是否配置成功。
```
~/.ssh ssh -T git@github.com
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.

~/.ssh ssh -T git@sjzcode.coding.net
Coding 提示: Hello xxx, You've connected to Coding.net via SSH. This is a personal key.
xx，你好，你已经通过 SSH 协议认证 Coding.net 服务，这是一个个人公钥.
公钥指纹：xxxxxxxxx
```
# 5.账号快速切换
> 如果在各个托管平台使用的用户名和邮箱都相同，请忽略此步骤。
> 这一步的原理就是通过自定义命令别名来实现快速切换用户。

在 `.bash_profile` 中增加下面内容：
```
#git change
alias cgc='git config --global user.name 用户名1 && git config --global user.email 邮箱1 && git config user.name && git config user.email'
alias cgg='git config --global user.name 用户名2 && git config --global user.email 邮箱2 && git config user.name && git config user.email'
```
切记执行 `source ~/.bash_profile` 以使修改生效。


# Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)

