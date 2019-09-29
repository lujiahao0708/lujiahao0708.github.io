---
title: Docker Compose无法部署Vue项目
date: 2019-09-12 19:04:47
categories: Docker
tags:
- CI&CD
- Docker

---

近期在进行Vue项目自动化构建，遇到一些问题，下面记录一个典型的问题。

<!--more-->

## 1. 问题出现
在使用docker部署vue项目的时候，使用了multi-stage-build来构建最终镜像，然鹅在构建时却出现了如下问题：
```jshelllanguage
Building nginx_manager
Step 1/12 : FROM node:lts-alpine as base
Service 'nginx_manager' failed to build: Error parsing reference: "node:lts-alpine as base" \
is not a valid repository/tag: invalid reference format
SSH: EXEC: completed after 1,636 ms
SSH: Disconnecting configuration [tencent_server] ...
ERROR: Exception when publishing, exception message [Exec exit status not zero. Status [1]]
Finished: UNSTABLE
```

这贴一下我最原始的Dockerfile和docker-compose.yml
```jshelllanguage
FROM node:lts-alpine as base-build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build:prod
# production stage
FROM nginx:stable-alpine as nginx-build
# 将修改的文件替换掉容器中的nginx配置
# COPY default.conf /etc/nginx/conf.d/default.conf
COPY --from=base-build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
```jshelllanguage
version: "3"
services:
  nginx:
    container_name: manager
    build: 
      context: ./manager
      dockerfile: Dockerfile
    ports:
      - 8080:80
    environment:
      HOST: 0.0.0.0
```

## 2. 查找解决方案
于是开展各种搜索，搜索到一个相关问题，推荐使用target的方式来指定具体的构建阶段

> 相关链接：https://stackoverflow.com/questions/53093487/multi-stage-build-in-docker-compose

修改后的docker-compose.yml
```jshelllanguage
version: "3.4"
services:
  nginx_manager:
    container_name: manager
    build: 
      context: ./manager
      dockerfile: Dockerfile
      target: nginx-build
    ports:
      - 8080:80
    environment:
      HOST: 0.0.0.0
```

执行后依旧是上面的错误。。。

于是又进行了搜索，在GitHub上搜索到相关问题，原因竟然在于docker版本过低！

> 相关链接：https://github.com/tootsuite/mastodon/issues/8240

GitHub上的讨论指出：
```jshelllanguage
Docker's "multi stage build" with keyword "as" is supported on Docker 17.05 or later.
```

查看服务器docker版本，确实是低于17.05，于是重装了docker。
```jshelllanguage
# 安装依赖
yum install -y yum-utils   device-mapper-persistent-data   lvm2
# 修改为阿里云的源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 删除原来的docker
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate \
        docker-logrotate docker-selinux docker-engine-selinux docker-engine
# 安装docker ce
yum install docker-ce
```

安装成功后启动docker，结果出现如下错误：

```jshelllanguage
Error response from daemon: Unknown runtime specified docker-runc
Error: failed to start containers: 7253e1be7fb3
```

真是一波未平一波又起啊，不过有解决方案：
```jshelllanguage
[root@VM_12_255_centos ~]# cd /var/lib/docker/containers/
[root@VM_12_255_centos containers]# grep -rl "docker-runc" .
./550dc5fda621f02b5079e0d70d502c01752fef6a03f0bf3dbf533f0634fc6013/hostconfig.json
./d2bc485ef863b5742c29d4e95e5de641aaf5b474657eb16c47b51bc3d679ca1a/hostconfig.json
./afa79db1bd41284e99446404afee1f43f06a7381889fd90961493d83ffb31658/hostconfig.json
./b405ade18866912841727c384c4eb71625d59865dc72676c39dfdde1592c8748/hostconfig.json
./9c291209f3e09e5cdf6f7333873d8216be531aba940d185f12160a1ff98fdc6e/hostconfig.json
./9c291209f3e09e5cdf6f7333873d8216be531aba940d185f12160a1ff98fdc6e/config.v2.json
./e5a7f498b6a8501f4e625e4e8598b819577141dbb137f945356777d3c9c0c077/hostconfig.json
./c58152c4ceeb6506aeca1eacf029c51e53cfc3848bc588120b51e97ee6463f74/hostconfig.json
./b4b59541feb03b0f102f521da336f358be4c70f23ecdc837644e3835c024535d/hostconfig.json
./b4b59541feb03b0f102f521da336f358be4c70f23ecdc837644e3835c024535d/config.v2.json
[root@VM_12_255_centos containers]# sed -i "s/docker-runc//g" ./550dc5fda621f02b5079e0d70d502c01752fef6a03f0bf3dbf533f0634fc6013/hostconfig.json
```

将上面grep搜索出来的文件全都执行sed命令替换，之后正常重启即可
> 相关链接：https://forums.docker.com/t/after-upgrade-to-docker-ce-containers-will-not-start/40243/3

## 3. 总结
至此，此次问题告一段落，版本兼容问题确实不容小觑啊！

## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我的公众号：庄里程序猿，读书笔记教程资源第一时间获得！

![](https://github.com/lujiahao0708/PicRepo/raw/master/公众号二维码.jpg)

