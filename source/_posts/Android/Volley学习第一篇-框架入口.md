---
title: Volley学习第一篇-框架入口.md
categories: Android
date: 2016-05-21 19:00:29
tags:
  - Http
  - Volley
description: Volley框架入口分析
---

## 前文概要
这是我的Volley学习第一篇的笔记.
看了一个星期的各类教程之后的总结,准备把Volley的整体流程都总结一遍,同时会对Volley进行一些扩展,最后会跟着大神的脚步模仿一个Volley出来.

水平有限,望多指教.

## 资源
- [郭霖大神的博客](http://blog.csdn.net/guolin_blog/article/details/17656437)
- [codeKK Volley源码解析](http://a.codekk.com/blogs/detail/54cfab086c4761e5001b2542)
- [Volley源码](https://android.googlesource.com/platform/frameworks/volley/)

## 创建全局的RequestQueue
使用Volley的时候会创建一个全局的RequestQueue,一般会在自定义的Application类里面创建.
	
	RequestQueue requestQueue = Volley.newRequestQueue(appContext);
## Volley框架的入口点
这里我们就发现原来Volley的入口是Volley类里面的一个静态方法`newRequestQueue()`

	public static RequestQueue newRequestQueue(Context context) {
        return newRequestQueue(context, null);
	}

## Volley框架的真正的入口
这是一个参数的方法,会调用两个参数的newRequestQueue方法,传入一个默认的null.

	public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
	
        File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);

        String userAgent = "volley/0";
        try {
            String packageName = context.getPackageName();
            PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
            userAgent = packageName + "/" + info.versionCode;
        } catch (NameNotFoundException e) {
        }

        if (stack == null) {
            if (Build.VERSION.SDK_INT >= 9) {
                stack = new HurlStack();
            } else {
                // Prior to Gingerbread, HttpUrlConnection was unreliable.
                // See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html
                stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
            }
        }

        Network network = new BasicNetwork(stack);

        RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
        queue.start();

        return queue;
    }
这里才是真正的创建RequestQueue的地方,因为传入的第二个参数是null,所以stack== null是成立的,这是就会根据不同的系统版本创建不同的HttpStack子类的实例对象,然后将它包装成BasicNetwork对象.

同时第一行的时候就创建了Volley的缓存目录(当然你可以根据你的需求随意更改缓存目录的位置),并将缓存目录包装成一个DiskBasedCache对象.

至此,RequestQueue需要的缓存和网络请求对象就创建成功了,然后我们的RequestQueue请求队列就创建成功了.

通过`start()`方法就启动了我们的请求队列.

## 小结
两个`newRequestQueue`方法使用的这种方式其实非常巧妙,以前我看到这种方式就叫他重载调用(不知道叫的对不对,哈哈).

这是我的Volley系列的第一篇,好的开始,加油↖(^ω^)↗!

