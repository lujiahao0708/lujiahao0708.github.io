---
title: Volley学习第二篇-缓存流程
date: 2016-05-23 11:14:42
categories: Android
tags:
  - Http
  - Volley
description: Volley缓存流程的分析
---

## 前文概要
> 上篇说道Volley初始化的时候需要创建一个RequestQueue消息队列,下面就来看看这个RequestQueue.

## RequestQueue
> 是一个队列管理器,里面维护了两个队列---`CacheQueue`和`NetowrkQueue`.

Volley类里面的newRequestQueue方法中调用了队列的start()方法,就从这个方法入手.

	public void start() {
        stop();  // Make sure any currently running dispatchers are stopped.
        // Create the cache dispatcher and start it.
        mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);
        mCacheDispatcher.start();

        // 这里默认创建4个线程
        // Create network dispatchers (and corresponding threads) up to the pool size.
        for (int i = 0; i < mDispatchers.length; i++) {
            NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork,
                    mCache, mDelivery);
            mDispatchers[i] = networkDispatcher;
            networkDispatcher.start();
        }
    }
这里创建了1个`CacheDispatcher`和4个`NetworkDispatcher`.`CacheDispatcher`和4个`NetworkDispatcher`都是继承了Thread的两个线程,一个是缓存线程,另一个是网络线程.

其中`DEFAULT_NETWORK_THREAD_POOL_SIZE`中定义了网络线程的个数,可以根据不同的cpu核数来自定义开多少个网络线程`(线程数 = cpu核数 * 2 + 1)`.

## CacheDispatcher缓存线程的流程
它继承了Thread,所以只需要看它的`run()`方法就好了.

	while (true) {
        try {
            // Get a request from the cache triage queue, blocking until
            // at least one is available.从缓存队列中不停的取出request,直到队列中只有一个请求的时候阻塞
            final Request request = mCacheQueue.take();
            request.addMarker("cache-queue-take");

            // If the request has been canceled, don't bother dispatching it.
            if (request.isCanceled()) {
                request.finish("cache-discard-canceled");
                continue;// 请求取消就不读缓存了
            }

            // Attempt to retrieve this item from cache. 从缓存中获取缓存信息的实体
            Cache.Entry entry = mCache.get(request.getCacheKey());
            if (entry == null) {
                request.addMarker("cache-miss");
                // Cache miss; send off to the network dispatcher.
                mNetworkQueue.put(request);// 缓存木有,将请求添加到网络请求队列
                continue;
            }

            // If it is completely expired 过期, just send it to the network.
            if (entry.isExpired()) {
                request.addMarker("cache-hit-expired");
                request.setCacheEntry(entry);
                mNetworkQueue.put(request);// 缓存过期,添加到网络请求队列
                continue;
            }

            // We have a cache hit; parse its data for delivery back to the request.
            request.addMarker("cache-hit");
            // 解析网络数据  这个是由请求对象request来解析的
			// 文档中说道:request对象负责请求和解析网络请求
            Response<?> response = request.parseNetworkResponse(
                    new NetworkResponse(entry.data, entry.responseHeaders));
            request.addMarker("cache-hit-parsed");

            if (!entry.refreshNeeded()) {// 缓存是否需要刷新
                // Completely unexpired cache hit. Just deliver the response.
                mDelivery.postResponse(request, response);// 无需刷新,直接分发
            } else {
                // Soft-expired cache hit. We can deliver the cached response,
                // but we need to also send the request to the network for
                // refreshing.
                request.addMarker("cache-hit-refresh-needed");
                request.setCacheEntry(entry);// 更新缓存

                // Mark the response as intermediate.// 中间  媒介
                response.intermediate = true;

                // Post the intermediate response back to the user and have
                // the delivery then forward the request along to the network.
                mDelivery.postResponse(request, response, new Runnable() {
                    @Override
                    public void run() {
                        try {
                            mNetworkQueue.put(request);
                        } catch (InterruptedException e) {
                            // Not much we can do about this.
                        }
                    }
                });
            }

        } catch (InterruptedException e) {
            // We may have been interrupted because it was time to quit.
            if (mQuit) {
                return;
            }
            continue;
        }
    }
缓存的主体流程就在这个死循环里面,Volley的dispatcher的原理和Handler里面的looper的原理非常相似.

## 读取缓存分发到主线程

	Response<?> response = request.parseNetworkResponse(new NetworkResponse(entry.data, entry.responseHeaders));
    request.addMarker("cache-hit-parsed");

    if (!entry.refreshNeeded()) {
        // Completely unexpired cache hit. Just deliver the response.
        mDelivery.postResponse(request, response);
    } else {
        // Soft-expired cache hit. We can deliver the cached response,
        // but we need to also send the request to the network for
        // refreshing.
        request.addMarker("cache-hit-refresh-needed");
        request.setCacheEntry(entry);

        // Mark the response as intermediate.// 中间  媒介
        response.intermediate = true;

        // Post the intermediate response back to the user and have
        // the delivery then forward the request along to the network.
        mDelivery.postResponse(request, response, new Runnable() {
            @Override
            public void run() {
                try {
                    mNetworkQueue.put(request);
                } catch (InterruptedException e) {
                    // Not much we can do about this.
                }
            }
        });
    }
### 详解
	request.parseNetworkResponse(new NetworkResponse(entry.data, entry.responseHeaders));`

`entry.data`是缓存的原始的`byte数组`,将`byte数组`和`响应头`封装成一个`NetworkResponse对象`(Volley里面的网络响应的统一对象).
`parseNetworkResponse()`方法将`NetworkResponse对象`解析成`Response`供各种泛型的转换.


	mDelivery.postResponse(request, response);

`mDelivery`对象时在`CacheDispatcher`的构造方法的时候赋值的,往上找找`RequestQueue`中的`start()`方法中`mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);`

继续找

	public RequestQueue(Cache cache, Network network, int threadPoolSize,ResponseDelivery delivery) {
	    mCache = cache;
	    mNetwork = network;
	    mDispatchers = new NetworkDispatcher[threadPoolSize];
	    mDelivery = delivery;
	}

还找

	public RequestQueue(Cache cache, Network network, int threadPoolSize) {
	    this(cache, network, threadPoolSize,
	            new ExecutorDelivery(new Handler(Looper.getMainLooper())));
	}
看mDelivery就是`new ExecutorDelivery(new Handler(Looper.getMainLooper()))`

ExecutorDelivery中构造方法中有一个非常重要的一行

	public ExecutorDelivery(final Handler handler) {
	    // Make an Executor that just wraps the handler.
	    mResponsePoster = new Executor() {
	        @Override
	        public void execute(Runnable command) {
	            handler.post(command);
	        }
	    };
	}

`handler`将`runnable对象`post出去,而handler是通过`Looper.getMainLooper`创建的,这样就是通过我们用主线程创建的Handler将响应发送到主线程中了.

至此,Volley缓存的读取/分发就完成了.

## 总结
![](http://img.blog.csdn.net/20140511114837375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
官方图解中的绿色的那部分---缓存线程的流程就结束了.

后面我会慢慢肥西网络线程相关的东西.