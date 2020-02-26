---
title: 第一章 快速认识线程
date: 2019-11-20 17:54:41
categories: 并发
tags:
- Java
- 并发
---

对于计算机而言每个任务就是一个进程,在每个进程内部至少要有一个线程,因此有时线程也称为轻量级进程.
线程是程序执行的一个路径,每一个线程都有自己的局部变量表,程序计数器以及各自的生命周期.

## 1.创建线程

### 1.1 演示并行运行
```java
public class TryConcurrencyThread {
    public static void main(String[] args) {
        // 通过匿名内部类的方式创建线程,并重写其中的run方法
        new Thread() {
            @Override
            public void run() {
                enjoyMusic();
            }
        }.start();

        browseNews();
    }

    private static void browseNews() {
        for (; ; ) {
            System.out.println("Uh-huh, the good news.");
            sleep(1);
        }
    }

    private static void enjoyMusic() {
        for (; ; ) {
            System.out.println("Uh-huh, the nice music.");
            sleep(1);
        }
    }

    /**
     * 模拟等待并忽略异常
     */
    private static void sleep(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

输出结果:
```
Uh-huh, the good news.
Uh-huh, the nice music.
Uh-huh, the good news.
Uh-huh, the nice music.
```

> 上面创建线程的方法可以使用 Java 8 Lambda优化 `new Thread(TryConcurrencyThreadLambda::enjoyMusic).start();`

### 1.2 使用Jconsole观察线程
Jconsole是JDK自身提供的监测工具.可以在控制台启动jconsole
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/%E3%80%8AJava%20%E9%AB%98%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E8%AF%A6%E8%A7%A3-%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B8%8E%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1%E3%80%8B%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/01.%E5%BF%AB%E9%80%9F%E8%AE%A4%E8%AF%86%E7%BA%BF%E7%A8%8B/jconsole.png)

图中Thread-0就是我们创建的新线程.

## 2.线程生命周期
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/%E3%80%8AJava%20%E9%AB%98%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E8%AF%A6%E8%A7%A3-%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B8%8E%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1%E3%80%8B%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/01.%E5%BF%AB%E9%80%9F%E8%AE%A4%E8%AF%86%E7%BA%BF%E7%A8%8B/%E7%BA%BF%E7%A8%8B%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

线程生命周期:
- NEW
- RUNNABLE
- RUNNING
- BLOCKED
- TERMINATED

### 2.1 线程NEW状态
当我们用关键字 new 创建一个 Thread 对象时,此时它并不处于执行状态,因为没有调用 start 方法启动该线程,那么线程的状态为 NEW 状态,准确的说,它只是 Thread 对象的状态,因为在没有 start 之前,该线程根本不存在,与你用关键字 new 创建一个普通的 Java 对象没有什么区别.

NEW 状态通过 start 方法进入 RUNNABLE 状态.

### 2.2 线程RUNNABLE状态
线程对象进入 RUNNABLE 状态必须调用 start 方法,那么此时才是真正的在 JVM 进程中创建了一个线程,线程启动后是不会立即执行的,需要听令于 CPU 的调度,此时称此状态为可执行状态(RUNNABLE),即线程具备执行资格,但是并没有真正执行,在等待 CPU 调度.

由于存在 running 状态,所以不会直接进入 BLOCKED 状态和 TERMINATED 状态,即使是在线程的执行逻辑中调用 wait sleep 或者其他 block 的IO 操作等,也不许先获得 CPU 的调度执行权才可以,严格来讲, RUNNABLE 的线程只能意外终止或者进入 RUNNING 状态.

### 2.3 线程RUNNING状态
一旦 CPU 通过轮询或者其他方式从任务可执行队列中选中了线程,那么此时它才能真正的执行自己的逻辑代码,需要说明的一点是一个正在 RUNNING 状态的线程事实上也是 RUNNABLE 的,反之则不成立.

此状态下线程的状态可以发生如下转换:
- 直接进入 TERMINATED 状态,比如调用 JDK 已经不推荐的 stop 方法或者判断某个逻辑标识
- 进入 BLOCKED 状态,比如调用 sleep 或者wait 方法而加入 waitSet中
- 进行某个阻塞 IO 操作,比如因网络的读写而进入 BLOCKED 状态
- 获取某个锁资源,从而加入到该锁的阻塞队列中而进入 BLOCKED 状态
- 由于 CPU 的调度器轮询使该线程放弃执行,进入 RUNNABLE 状态
- 线程主动调用 yield 方法,放弃 CPU 执行权,进入 RUNNABLE 状态


### 2.4 线程BLOCKED状态 
从上节即可获得线程进入 BLOCKED 状态的原因,接下来介绍线程在 BLOCKED 状态中可以切换至如下几个状态:
- 直接进入 TERMINATED 状态,比如调用 JDK 已经不推荐的 stop 方法或者意外死亡(JVM Crash)
- 线程阻塞操作结束,比如读取了想要的数据字节进入到 RUNNABLE 状态
- 线程完成了指定时间的休眠,进入到 RUNNABLE 状态
- Wait 中的线程被其他线程 notify/notifyall 唤醒,进入到RUNNABLE 状态
- 线程获取到了某个锁资源,进入到 RUNNABLE 状态
- 线程在阻塞过程中被打断,比如其他线程调用了 interrupt 方法,进入 RUNNABLE 状态

### 2.5 线程TERMINATED状态
TERMINATED 是一个线程的最终途状态,在该状态中线程将不会切换到其他任何状态,线程进入 TERMINATED 状态,意味着线程的整个生命周期结束了,下列情况将会使线程进入到 TERMINATED 状态:
- 线程运行正常结束,结束生命周期
- 线程运行出错意外结束
- JVM Crash,导致所有的线程都结束

## 3.线程start方法剖析和模板设计模式应用

### 3.1 start方法源码解析
如果直接调用线程的run方法,线程是不会启动的,这相当于调用了一个类的属性方法.只有调用start方法才会启动线程,接下来我们剖析start方法原理:
```java
public synchronized void start() {
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
        }
    }
}
```

start方法源码很简单,核心方法是start0的本地方法.start 方法中会调用 start0方法,而start0方法会调用线程的 run方法,这一点在源码中有注释:
```
Causes this thread to begin execution; the Java Virtual Machine calls the run method of this thread.
```

总结下源码中获得的知识点:
- Thread 被构造后的 NEW状态,事实上 threadStatus 这个内部属性是0
- 不能两次启动 Thread,否则就会出现 IllegalThreadStateException
- 线程启动后会被加入到一个 ThreadGroup 中
- 线程生命周期结束,即 TERMINATED 状态,再次调用 start 方法是不允许的,即 TERMINATED 状态无法回到 RUNNABLE/RUNNING 状态



## 4.Runnable接口


## 参考资料
- [《JAVA与模式》之模板方法模式](https://www.cnblogs.com/java-my-life/archive/2012/05/14/2495235.html)
- [Java设计模式：策略模式](http://ifeve.com/java-example-of-strategy-pattern/)

## 代码获取
[https://github.com/lujiahao0708/LearnJavaConcurrent](https://github.com/lujiahao0708/LearnJavaConcurrent)

## Tips
欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 

欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！！

![](https://github.com/lujiahao0708/PicRepo/raw/master/公众号二维码.jpg)
