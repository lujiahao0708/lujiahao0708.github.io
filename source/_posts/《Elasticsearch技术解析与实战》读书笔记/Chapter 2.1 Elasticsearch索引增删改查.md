---
layout: post
title: 《Elasticsearch技术解析与实战》Chapter 2.1 Elasticsearch索引增删改查
date: 2019-04-17 11:34:52
categories: Java
tags:
- Elasticsearch
- 读书笔记
description: 《Elasticsearch技术解析与实战》Chapter 2.1 Elasticsearch索引增删改查
---

# 1. 创建索引
```
PUT /lujiahao123
{
  "acknowledged": true,
  "shards_acknowledged": true
}
索引默认的主分片数量是5，每个主分片的副本数量是1。
```

<!--more-->

创建自定义字段类型的索引：
```
PUT /order
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 2
  },
  "mappings": {
    "carpoolType":{
      "properties": {
        "orderNo":{
          "type": "string",
          "index": "not_analyzed"
        }
      }
    }
  }
}
#! Deprecation: The [string] field is deprecated, please use [text] or [keyword] instead on [orderNo]
{
  "acknowledged": true,
  "shards_acknowledged": true
}
ES5.0版本中string类型被拆分为text和keyword类型: https://www.elastic.co/blog/strings-are-dead-long-live-strings
```

# 2. 修改索引
```
PUT /order/_settings
{
  "number_of_replicas": 2
}
```

更新分词器
```
POST /order/_close
PUT /order/_settings
{
  "analysis": {
    "analyzer": {
      "content":{
        "type":"customer",
        "tokenizer":"whitespace"
      }
    }
  }
}
POST /order/_open
添加分析器之前必须先关闭索引，添加之后再打开索引。
```

尝试修改索引主分片数：
```
PUT /order/_settings
{
  "number_of_shards": 2
}
```
错误提示：
```
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "Can't update non dynamic settings [[index.number_of_shards]] for open indices [[order/2cOJ6Ga7THCyW10idoPPig]]"
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "Can't update non dynamic settings [[index.number_of_shards]] for open indices [[order/2cOJ6Ga7THCyW10idoPPig]]"
  },
  "status": 400
}

主分片是无法修改的，主分片个数一旦建立就无法修改，但是看错误原因里面说因为这个是open的索引，那是不是关闭的索引就可以修改主分片个数了呢？这个我们后面再解答。
```

# 3. 删除索引
```
删除单个索引：
DELETE /order
删除索引需要指定索引名称，别名或者通配符
```

```
删除多个索引：
DELETE order,order1,order2
DELETE order*
```

```
删除全部索引：
DELETE _all
DELETE *
```
使用_all 和通配符删除全部索引
为了防止误删除，设置config/elasticsearch.yml属性action.destructive_requires_name=true，以禁用通配符或_all删除索引。

# 4. 查询索引
```
GET /order
{
  "order": {
    "aliases": {},
    "mappings": {
      "carpoolType": {
        "properties": {
          "orderNo": {
            "type": "keyword"
          }
        }
      }
    },
    "settings": {
      "index": {
        "creation_date": "1555054886098",
        "number_of_shards": "3",
        "number_of_replicas": "2",
        "uuid": "uSh9K26CS_q1uZKaos7NRQ",
        "version": {
          "created": "5050099"
        },
        "provided_name": "order"
      }
    }
  }
}
```

自定义返回结果的属性：
```
GET /order/_settings,_aliases
{
  "order": {
    "settings": {
      "index": {
        "creation_date": "1555054886098",
        "number_of_shards": "3",
        "number_of_replicas": "2",
        "uuid": "uSh9K26CS_q1uZKaos7NRQ",
        "version": {
          "created": "5050099"
        },
        "provided_name": "order"
      }
    },
    "aliases": {}
  }
}
```

# 5. 打开/关闭索引
索引关闭只能显示索引元数据信息，不能够进行读写。
```
POST /order/_close
POST /order/_open
health status index   uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   .kibana 8n5wnGEjRJ-aVa54l_jTjA   1   1          1            0      3.1kb          3.1kb
yellow open   order   uSh9K26CS_q1uZKaos7NRQ   3   2          0            0       486b           486b
```
可以同时打开或关闭多个索引。如果指向不存在的索引会抛出错误，可通过配置ignore_unavailable=true 关闭异常提示。
禁止使用关闭索引功能，配置settingscluster.indices.close.enable为false，默认值是ture。

# 6. 关闭索引后尝试修改主分片个数
```
PUT /order/_settings
{
  "number_of_shards": 2
}
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "final order setting [index.number_of_shards], not updateable"
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "final order setting [index.number_of_shards], not updateable"
  },
  "status": 400
}
```
解答了之前的问题，索引一旦建立，主分片数量不可改变。


## Tips
本文同步发表在公众号，欢迎大家关注！😁 
后续笔记欢迎关注获取第一时间更新！
![](https://github.com/lujiahao0708/PicRepo/raw/master/公众号二维码.jpg)