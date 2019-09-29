title: Android网络层二次封装
categories: Android
tags:
  - Http
description: Android网络层二次封装
date: 2016-5-18 15:40:12
---
## 项目介绍
对Volley进行二次封装，方便使用和扩展。主要是学习封装的思想。
> 该示例主体代码来自传智的某位Android讲师，具体不清楚。

## 网络层封装示意图
![网络层二次封装示意图.png](https://ooo.0o0.ooo/2016/05/18/573c1e3de1315.png)
![网络层二次封装具体流程.png](https://ooo.0o0.ooo/2016/05/18/573c1e3def867.png)

<hr/>

## 接下来前期准备
	public class App extends Application {
	    public static Context application;
	    public static HttpLoader HL;
	
	    @Override
	    public void onCreate() {
	        super.onCreate();
	        application = this;
	
	        // 初始化网络请求相关的核心类
	        HL = HttpLoader.getInstance(this);
	    }
	}
不用我介绍了吧，记得在AndroidManiFest文件中添加name属性&&&&&&&<font color=red>联网权限</font>

### 1.Activity实现HttpListener接口
HttpListener是我们连接网络层和UI层的桥梁，记得重写接口中的成功和失败的方法。
### 2.创建Protocol类发起网络请求

	new MainProtocol(MainActivity.this,"15613566958").doRequest(this);
这样我们就发起了一个网络请求，感觉简单了好多。

具体`MainProtocol`的具体实现

	public class MainProtocol extends BaseProtrocol {
	    private String phone;
	    private Context actContext;
		// 传入必备参数
	    public MainProtocol(Context actContext,String phone) {
	        this.actContext = actContext;
	        this.phone = phone;
	    }
	
	    @Override
	    public void doRequest(HttpLoader loader, HttpLoader.HttpListener listener) {
	        HttpParams params = new HttpParams().put("phone", phone);
			// 发起GET请求
	        loader.get(actContext, AppConstants.URL_COUPONS, params, AppConstants.REQUEST_CODE_COUPONS, listener);
	    }
	}
### 3.HttpLoader--网络请求类
单例类，不要问我为什么，这是网络请求的类啊，能不单利吗！！！！

其中`mRequestQueue`是保存请求队列的，`mInFlightRequests`是用来保存已经等待的请求，同时具有过滤重复请求的功能。

`get`方法会调用`request`方法，这里才是请求的主体，`request`方法在发起请求前会通过`tryLoadCacheResponse`方法首先读取缓存，然后在进行访问网络的操作。然后根据返回结果的不同，分别调用HttpListener的不同的回调方法，这样就把服务器的结果返回到UI层了。

	/**
     * ResponseListener，封装了Volley的错误和成功的回调监听，并执行一些默认处理，同时会将事件通过HttpListener分发到UI层
     */
    private class ResponseListener implements Response.ErrorListener, Response.Listener<String> {
        private HttpListener listener;
        private int requestCode;

        public ResponseListener(int requestCode, HttpListener listener) {
            this.listener = listener;
            this.requestCode = requestCode;
        }

        @Override
        public void onErrorResponse(VolleyError volleyError) {
            LLog.w("Request error from network!");
            volleyError.printStackTrace();
            mInFlightRequests.remove(requestCode);// 请求错误，从请求集合中删除该请求
            if (listener != null) {
                listener.onGetResponseError(requestCode, volleyError);
            }
        }

        @Override
        public void onResponse(String response) {
            mInFlightRequests.remove(requestCode);// 请求成功，从请求集合中删除该请求
            if (response != null) {
                //SystemClock.sleep(2000);
                LLog.i("Request success from network!");
                try {
                    JSONObject jsonObject = new JSONObject(response);// 处理分发数据---解析json
                    String status = jsonObject.getString("status");
                    if (listener != null && response != null) {// 分发数据
                        if (("success".equals(status) || "ok".equals(status))) {
                            String data = null;
                            try {
                                data = jsonObject.getString("data");
                                listener.onGetResponseSuccess(requestCode, data);
                            } catch (Exception e) {
                                // 不是标准格式的时候就把原始数据传回去
                                listener.onGetResponseSuccess(requestCode, response);
                            }
                        } else if (("fail".equals(status) || "error".equals(status))) {
                            String errMsg = null;
                            try {
                                errMsg = jsonObject.getString("msg");
                            } catch (Exception e) {
                                errMsg = jsonObject.getString("message");
                            } finally {
                                listener.onGetResponseError(requestCode, new VolleyError(errMsg));
                            }
                        }
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    listener.onGetResponseError(requestCode, new VolleyError("解析问题"));
                }
            }
        }
    }

这之中ResponseListener是关键，他关联了三方框架的回调方法和HttpListener的回调方法。然后不要问我为嘛onResponse里面写了这么多，你是为了健壮性更好吗？

![](http://7xjzdd.com1.z0.glb.clouddn.com/i/2016-05-13-7615e8bba1d0d8199c492a1eecbcfde0.png)我能骂街吗？
后台返回数据是乱七八糟的好不？![](http://7xjzdd.com1.z0.glb.clouddn.com/i/2016-05-18-61e99217174057ace5ae7b498d6804cd.gif)

## 小结
好了，到了这里你肯定还没有看懂，废话，我才写了几篇文章，哪有那么快就写的让你看懂。
如果你还没有看懂的话就看我的代码去吧，[我不是链接](https://github.com/chiahaolu/Arsenal/tree/master/volleydemo)

还是看不懂？
![](http://7xjzdd.com1.z0.glb.clouddn.com/i/2016-05-11-5b3011df1e27d254986292b1233eca38.png)

找我要视频啊~

看完了再看这几篇文章就好了  多搞搞就好了

![](http://7xjzdd.com1.z0.glb.clouddn.com/i/2016-04-20-eb05adccee2519e22b635dce85669ce7.jpg)