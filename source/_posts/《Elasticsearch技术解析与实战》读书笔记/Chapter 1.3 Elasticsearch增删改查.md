---
layout: post
title: 《Elasticsearch技术解析与实战》Chapter 1.3 Elasticsearch增删改查
date: 2019-04-15 11:34:52
categories: Java
tags:
- Elasticsearch
- 读书笔记
description: 《Elasticsearch技术解析与实战》Chapter 1.3 Elasticsearch增删改查
---

# 1. 新增文档，建立索引
语法格式:

    PUT /index/type/id
    {
      "json数据"
    }

输入:

    PUT /person/chinese/1
    {
      "id":12345,
      "name":"lujiahao",
      "age":18
    }

输出:
    
    {
      "_index": "person",
      "_type": "chinese",
      "_id": "1",
      "_version": 1,
      "result": "created",
      "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
      },
      "created": true
    }

> es会自动建立index和type，不需要提前创建，而且es默认会对document每个field都建立倒排索引，让其可以被搜索。

# 2. 检索文档
格式:

    GET /index/type/id

输入:

    GET /person/chinese/1
输出:

    {
      "_index": "person",
      "_type": "chinese",
      "_id": "1",
      "_version": 1,
      "found": true,
      "_source": {
        "id": 12345,
        "name": "lujiahao",
        "age": 18
      }
    }
    
# 3. 更新文档
## 3.1 替换方式
格式:

    PUT /index/type/id
    {
        "json数据"
    }
输入:

    PUT /person/chinese/1
    {
      "name":"lujiahao123"
    }
输出:

    {
      "_index": "person",
      "_type": "chinese",
      "_id": "1",
      "_version": 2,
      "result": "updated",
      "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
      },
      "created": false
    }

查询:

    {
      "_index": "person",
      "_type": "chinese",
      "_id": "1",
      "_version": 2,
      "found": true,
      "_source": {
        "name": "lujiahao123"
      }
    }

> 替换方式更新文档时，必须带上所有的field，才能去进行信息的修改；如果缺少field就会丢失部分数据。其原理时替换，因此需要全部字段。不推荐此种方式更新文档。

## 3.1 更新方式
格式:

    POST /index/type/id/_update
    {
        "doc":{
            "json数据"
        }
    }
输入:

    POST /person/chinese/1/_update
    {
      "doc":{
        "name":"lujiahao10010"
      }
    }
输出:

    {
      "_index": "person",
      "_type": "chinese",
      "_id": "1",
      "_version": 4,
      "result": "updated",
      "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
      },
      "created": false
    }
再次查询：

    {
      "_index": "person",
      "_type": "chinese",
      "_id": "1",
      "_version": 6,
      "found": true,
      "_source": {
        "id": 12345,
        "name": "lujiahao10010",
        "age": 18
      }
    }
# 4. 删除文档
格式:

    DELETE /index/type/id/_update
    {
        "doc":{
            "json数据"
        }
    }
输入:

    DELETE /person/chinese/1
输出:

    {
      "found": true,
      "_index": "person",
      "_type": "chinese",
      "_id": "1",
      "_version": 7,
      "result": "deleted",
      "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
      }
    }
再次查询:
    
    {
      "_index": "person",
      "_type": "chinese",
      "_id": "1",
      "found": false
    }

# 5. 小结
本文所有操作都是在kibana的Dev tools中进行的，相较于Elasticsearch-Heade插件，kibana中更加方便与美观（个人观点），推荐大家使用。

## Tips
本文同步发表在公众号，欢迎大家关注！😁 
后续笔记欢迎关注获取第一时间更新！
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)