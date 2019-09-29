---
title: 解决Mac下VSCode打开zsh乱码
date: 2019-07-21 11:38:26
categories: Mac
tags:
- VSCode
- Mac技巧
---

## 1.乱码问题
iTerm2终端使用Zsh，并且配置Zsh主题，该主题主题需要安装字体来支持箭头效果，在iTerm2中设置这个字体，但是VSCode里这个箭头还是显示乱码。

<!--more-->

iTerm2展示如下:
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Mac%E7%9B%B8%E5%85%B3/%E8%A7%A3%E5%86%B3Mac%E4%B8%8BVSCode%E6%89%93%E5%BC%80zsh%E4%B9%B1%E7%A0%81/iterm%E5%B1%95%E7%8E%B0.png)

VSCode展示如下:
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Mac%E7%9B%B8%E5%85%B3/%E8%A7%A3%E5%86%B3Mac%E4%B8%8BVSCode%E6%89%93%E5%BC%80zsh%E4%B9%B1%E7%A0%81/vscode%E4%B9%B1%E7%A0%81.png)

## 2.解决方案
### 2.1 字体
在字体册查找是否已经安装`Meslo LG M for Powerline`字体，如果未安装就安装一下。

### 2.2 VSCode中配置
使用`⌘,`打开settings界面,搜索`terminal`,在`Font Family`中添加字体`Meslo LG M for Powerline`

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Mac%E7%9B%B8%E5%85%B3/%E8%A7%A3%E5%86%B3Mac%E4%B8%8BVSCode%E6%89%93%E5%BC%80zsh%E4%B9%B1%E7%A0%81/%E4%BF%AE%E6%94%B9%E5%AD%97%E4%BD%93.png)

> 也可以在VSCode的settings.json文件，加入 : "terminal.integrated.fontFamily": "Meslo LG M for Powerline",

## 3.效果展示

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Mac%E7%9B%B8%E5%85%B3/%E8%A7%A3%E5%86%B3Mac%E4%B8%8BVSCode%E6%89%93%E5%BC%80zsh%E4%B9%B1%E7%A0%81/%E4%B9%B1%E7%A0%81%E8%A7%A3%E5%86%B3.png)



----
欢迎大家关注😁
![](https://github.com/lujiahao0708/PicRepo/raw/master/公众号二维码.jpg)

