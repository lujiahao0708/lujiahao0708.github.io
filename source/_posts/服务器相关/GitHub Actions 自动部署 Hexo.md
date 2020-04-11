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

# 1. 生成公钥私钥
```
ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f github-deploy-key -N ""
```
之后会生成两个文件 `github-deploy-key.pub` 和 `github-deploy-key` ,这两个文件分别是公钥和私钥（公钥和私钥切记添加 `.gitignore`，以防万一）。

# 2. 配置公钥私钥
在 GitHub 中博客工程的 `Settings->Deploye keys->Add deploy key` 中添加`github-deploy-key.pub`中的内容

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9B%B8%E5%85%B3/GitHub%20Actions%20%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2%20Hexo/deploy_keys.png)

在 GitHub 中博客工程的 `Settings->Secrets->Add a new secrets` 中添加 `github-deploy-key` 中的内容

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9B%B8%E5%85%B3/GitHub%20Actions%20%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2%20Hexo/add_secret.png)

> 注意：切记不要多复制空格!!!

# 3. 创建编译脚本
在博客源码分支(我这里是hexo分支)中创建`.github/workflows/HexoCI.yml`文件。
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

# 4. Hexo 配置
在项目根目录中修改 `_config.yml` 配置文件
```
deploy:
  type: git
  repo: git@github.com:lujiahao0708/lujiahao0708.github.io.git
  branch: master
```
> 这里的repo要填写ssh的形式，使用http形式可能会有问题

# 5. 验证
现在 Hexo 已经和 GitHub Actions 已经集成了,接下来在博客源码分支上推送代码即可自动编译部署.
执行过程可以在`Actions`中查看:
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9B%B8%E5%85%B3/GitHub%20Actions%20%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2%20Hexo/actions_result.png)

# 6. 参考链接
https://segmentfault.com/a/1190000020873860

## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！

![](https://github.com/lujiahao0708/PicRepo/raw/master/公众号二维码.jpg)
