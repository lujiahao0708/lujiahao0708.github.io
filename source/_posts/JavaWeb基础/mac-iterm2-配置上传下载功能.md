---
layout: post
title: Mac iTerm2 配置 rz/sz 上传下载功能
date: 2019-01-31 10:16:00
categories: 工具教程
tags:
- mac
- iterm2
description: Mac iTerm2 配置 rz/sz 上传下载功能
---


使用mac的同学要进行远程服务器文件的上传下载，推荐使用 sz 和 rz 命令，下文为iTerm2配置的方法。

## 1. 安装lrzsz
```shell
brew install lrzsz
```

## 2. 下载脚本

https://github.com/mmastrac/iterm2-zmodem

## 3. 复制脚本

将脚本`iterm2-send-zmodem.sh`和`iterm2-recv-zmodem.sh`复制到`/usr/local/bin/`目录中即可

```
cp iterm2-recv-zmodem.sh iterm2-send-zmodem.sh /usr/local/bin/
```

## 4. 配置 iTerm2 
打开 iTerm2,按`⌘+,`打开Perfences，选择Profiles标签页，在Profiles标签页下选择Advanced标签页，编辑Triggers

    Regular expression: rz waiting to receive.\*\*B0100
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-send-zmodem.sh
    Instant: checked
    
    Regular expression: \*\*B00000000000000
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
    Instant: checked

图示 : 
![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/工具教程/iterm2/iterm2配置szrz1.png)
![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/工具教程/iterm2/iterm2配置szrz2.png)

## 5. 远程服务器安装lrzsz

CentOS安装方法 : `yum -y install lrzsz`

## 6. 使用方法

1. 本地上传文件到远程服务器
    1. 登录到远程服务器，在远端服务器上输入 rz ，回车
    2. 弹框中选择本地要上传的文件
    3. 确定后等待上传完成

2. 远程服务器下载文件到本地
    1. 在远程服务器输入 `sz filename filename1 ... filenameN`
    2. 弹框中选择本地的存储目录
    3. 确定后等待下载完成

至此，我们就可以使用这项黑科技了！

## Tips
本文同步发表在公众号，欢迎大家关注！😁 各位大佬点点广告，万分感谢！！！
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)