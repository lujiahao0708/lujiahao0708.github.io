---
layout: post
title: 《Elasticsearch技术解析与实战》Chapter 1.2 Elasticsearch安装
date: 2019-04-14 11:34:52
categories: Java
tags:
- Elasticsearch
- 读书笔记
description: 《Elasticsearch技术解析与实战》Chapter 1.2 Elasticsearch安装
---

# 1. 下载安装
## 1.1 下载
```shell
https://www.elastic.co/downloads/elasticsearch
下载 https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.7.0.tar.gz
```

## 1.2 解压
```shell
tar -zxvf elasticsearch-6.7.0.tar.gz
```

## 1.3 运行
```shell
cd elasticsearch-6.7.0
bin/elasticsearch
```

## 1.4 检验
```shell
curl http://localhost:9200  或者浏览器访问
{
    "name": "q9sdES9",
    "cluster_name": "docker-cluster",
    "cluster_uuid": "6klEi4d0Q6y0LC3YNYVXTQ",
    "version": {
        "number": "5.5.0",
        "build_hash": "260387d",
        "build_date": "2017-06-30T23:16:05.735Z",
        "build_snapshot": false,
        "lucene_version": "6.6.0"
    },
    "tagline": "You Know, for Search"
}
```

# 2. Docker部署
> 官方文档 : https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

## 2.1 拉取镜像
```shell
docker pull docker.elastic.co/elasticsearch/elasticsearch:5.5.1
```

## 2.2 启动容器
```shell
docker run -p 9200:9200 9300:9300 -e "http.host=0.0.0.0" -e “transport.host=0.0.0.0" --name elasticsearch_5.5.0 -d docker.elastic.co/elasticsearch/elasticsearch:5.5.0
```

## 2.3 修改配置
```shell
进入到容器中 : docker exec -it elasticsearch_5.5.0 /bin/bash
修改jvm配置 : vi /config/jvm.options
            -Xms2g  —>  -Xms512m
            -Xmx2g  —>  -Xmx512m
    修改小一些,服务器内存有限😂
```

## 2.4 重启容器,查看是否成功
```shell
http://服务器ip:9200
{
    "name": "q9sdES9",
    "cluster_name": "docker-cluster",
    "cluster_uuid": "6klEi4d0Q6y0LC3YNYVXTQ",
    "version": {
        "number": "5.5.0",
        "build_hash": "260387d",
        "build_date": "2017-06-30T23:16:05.735Z",
        "build_snapshot": false,
        "lucene_version": "6.6.0"
    },
    "tagline": "You Know, for Search"
}
此处集群的名称为 docker-cluster, 可以自行修改 : vi /config/elasticsearch.yml
```

## Tips
本文同步发表在公众号，欢迎大家关注！😁 
后续笔记欢迎关注获取第一时间更新！
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)