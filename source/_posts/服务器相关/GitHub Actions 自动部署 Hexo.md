---
title: GitHub Actions 自动部署 Hexo
date: 2020-04-11 22:03:59
categories: Hexo
toc: true
tags:
- GitHub Actions
- Hexo
---

[Github Actions](https://github.com/features/actions) 是 GitHub 官方 CI 工具，与 GitHub 无缝集成。之前博客使用 TravisCI 实现的自动部署，现在转用 GitHub Actions 部署，本文记录部署流程。

<!-- more -->

简单介绍下 GitHub Actions 中的术语：

- workflow：表示一次持续集成的过程
- job：构建任务，一个 workflow 可以由一个或者多个 job 组成，可支持并发执行 job
- step：一个 job 由一个或多个 step 组成，按顺序依次执行
- action：每个 step 由一个或多个 action 组成，按顺序依次执行

---
接下来介绍下操作步骤：

# 1.博客工程
GitHub 博客创建步骤非本文重点，请自行搜索。
推荐使用 `master` 分支作为最终部署分支，源码分支可以根据自己喜好创建，我这里创建的是 `hexo`。

# 2.生成公私钥
源码分支中通过下面命令生成公钥和私钥。
```
 ~ cd github/lujiahao0708.github.io 
 ~ git checkout hexo
 ~ ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f github-deploy-key -N ""
```
目录中生成两个文件：
- `github-deploy-key.pub` --- 公钥文件
- `github-deploy-key` --- 私钥文件

> 公钥和私钥切记要添加到 `.gitignore` 中！！！

# 3.GitHub 添加公钥
在 GitHub 中博客工程中按照 `Settings->Deploye keys->Add deploy key` 找到对应的页面，然后进行公钥添加。该页面中 `Title` 自定义即可，`Key` 中添加 `github-deploy-key.pub` 文件中的内容。

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Hexo/GitHub%20Actions%20%E6%B7%BB%E5%8A%A0%E5%85%AC%E9%92%A5.png)

> 注意：切记不要多复制空格!!!
> 切记要勾选 `Allow write access`，否则会出现无法部署的情况。

# 4.GitHub 添加私钥
在 GitHub 中博客工程中按照 `Settings->Secrets->Add a new secrets` 找到对应的页面，然后进行私钥添加。该页面中 `Name` 自定义即可，`Value` 中添加 `github-deploy-key` 文件中的内容。

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Hexo/GitHub%20Actions%20%E6%B7%BB%E5%8A%A0%E7%A7%81%E9%92%A5.png)

> 注意：切记不要多复制空格!!!

# 5.创建编译脚本
在博客源码分支(我这里是hexo分支)中创建 `.github/workflows/HexoCI.yml` 文件，内容如下：
```yml
name: CI
on:
  push:
    branches:
      - hexo
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v1
        with:
          ref: hexo
      - name: Use Node.js ${{ matrix.node_version }}
        uses: actions/setup-node@v1
        with:
          version: ${{ matrix.node_version }}
      - name: Setup hexo
        env:
          ACTION_DEPLOY_KEY: ${{ secrets.HEXO_DEPLOY_PRI }}
        run: |
          mkdir -p ~/.ssh/
          echo "$ACTION_DEPLOY_KEY" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.email "lujiahao0708@gmail.com"
          git config --global user.name "lujiahao0708"
          npm install hexo-cli -g
          npm install
      - name: Hexo deploy
        run: |
          hexo clean
          hexo d
```

# 6.Hexo 配置
在项目根目录中修改 `_config.yml` ，增加部署相关内容：
```
deploy:
  type: git
  repo: git@github.com:lujiahao0708/lujiahao0708.github.io.git
  branch: master
```
> 这里的repo要填写ssh的形式，使用http形式可能会有问题

# 7.验证
现在 Hexo 已经和 GitHub Actions 已经集成了，接下来在博客源码分支上推送代码即可自动编译部署。具体
执行过程可以在 `Actions` 中查看：

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/Hexo/GitHub%20Actions%20%E9%83%A8%E7%BD%B2%E7%BB%93%E6%9E%9C.png)


## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！

![](https://github.com/lujiahao0708/PicRepo/raw/master/公众号二维码.jpg)
