---
title: Volley学习第三篇-网络线程
categories: Android
tags:
  - Http
  - Volley
description: Volley网络线程的分析
abbrlink: ec364bdb
date: 2016-05-23 14:42:23
---

## 前文概要

其实`NetworkDispatcher`和`CacheDispatcher`是很相像的.感觉看`NetworkDispatcher`要比`CacheDispatcher`要简单.

因为它不需要进行判断缓存的过期时间和新鲜时间,仅仅是执行请求,然后在缓存响应结果,所以要简单些.

## 流程分析
> 同样的继承Thread,只看`run()`方法里面的就好了
> 
> 同样的死循环,队列无内容时阻塞


- 1.首先从队列中取出Request请求

		request = mQueue.take();

- 2.判断请求是否取消

		if (request.isCanceled()) {
            request.finish("network-discard-cancelled");
            continue;
        }

- 3.请求未取消,执行网络请求并得到NetworkResponse对象

		NetworkResponse networkResponse = mNetwork.performRequest(request);
具体的网络请求还是需要看Network接口的实现类中是如何操作的.

- 4.判断服务器返回码是否是304(资源未更新)

		if (networkResponse.notModified && request.hasHadResponseDelivered()) {
            request.finish("not-modified");
            continue;
        }

- 5.是否缓存请求的响应

        if (request.shouldCache() && response.cacheEntry != null) {
            mCache.put(request.getCacheKey(), response.cacheEntry);
            request.addMarker("network-cache-written");
        }

- 6.分发

		mDelivery.postResponse(request, response);

## 小结

其实看明白先前的缓存线程之后,理解网络线程也不是难事了.

明天详细解释下具体响应的转化和分发流程.