---
title: Travis CI持续集成GitHub个人博客
categories: 持续集成
tags:
  - 持续集成
  - 博客系统
  - hexo
description: 'Travis CI持续集成GitHub个人博客,Github代码push触发自动部署个人博客'
abbrlink: b23c159
date: 2018-06-27 22:53:59
---

# 什么是Travis CI
> Travis CI 是目前新兴的开源持续集成构建项目，它与jenkins，GO的很明显的特别在于采用yaml格式，同时他是在在线的服务，
不像jenkins需要你本地打架服务器，简洁清新独树一帜。目前大多数的github项目都已经移入到Travis CI的构建队列中，
据说Travis CI每天运行超过4000次完整构建。对于做开源项目或者github的使用者，快将你的项目加入Travis CI构建队列吧!

# 目标
使用Hexo搭建托管在Github上的个人博客,每次推送新博客到Github,Travis CI 自动构建并推送到博客项目的master分支上.
由于GitPages服务规定网页文件必须在master分支上,所以博客源码内容在项目的hexo-source分支.

![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/TravisCI/blog-master.png)
![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/TravisCI/blog-source.png)

# 步骤
## 1.TravisCI创建账户
最好使用Github账户直接登录,登录后界面如下,勾选个人博客项目即可.

![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/TravisCI/0.png)

## 2.生成并配置Access Token
在GitHub生成Travis CI 的token

![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/TravisCI/1.png)
![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/TravisCI/2.png)
![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/TravisCI/3.png)
![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/TravisCI/4.png)
生成之后一定要保存好，因为只会出现一次，丢失了就只能再重新生成了。

之后将生成的token配置到Travis CI中

![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/TravisCI/5.png)

## 3.创建.travis.yml配置文件
在项目的hexo-source分支中,项目的根目录下创建.travis.yml配置文件 : 
```yml
language: node_js
node_js: 6

# S: Build Lifecycle
install:
  - npm install

#before_script:
 # - npm install -g gulp

script:
  - hexo g

after_script:
  - cd ./public
  - git init
  - git config user.name "lujiahao0708"
  - git config user.email "lujiahao0708@gmail.com"
  - git add .
  - git commit -m "Update docs"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master

# E: Build LifeCycle
branches:
  only:
    - hexo-source
env:
 global:
   - GH_REF: github.com/lujiahao0708/lujiahao0708.github.io.git
```

替换git config信息为你自己的,GH_REF的值更改为你的仓库地址.

## 4.发布新博客
将博客内容推送到hexo-source分支上,就会触发Travis CI 的自动构建.

![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/TravisCI/6.png)

# Q&A
## 1.Travis CI编译错误
    我参照的教程中.travis.yml配置文件的node_js版本使用`stable`,但是会出现错误.
    解决方案 : 
        使用低版本的NodeJS版本
        https://segmentfault.com/q/1010000011317783

## 2.自定义域名无法跳转
    CNAME文件直接放到了工程的根目录下,将无法打包进去
    解决方案 : 
        将CNAME文件放到source目录下

# 参考教程
https://blog.csdn.net/woblog/article/details/51319364
https://www.jianshu.com/p/5691815b81b6


----
欢迎大家关注😁
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)

