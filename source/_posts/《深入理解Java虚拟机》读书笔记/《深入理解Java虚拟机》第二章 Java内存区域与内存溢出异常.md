---
layout: post
title: 《深入理解Java虚拟机》第二章 Java 内存区域与内存溢出异常
date: 2019-01-11 19:22:27
categories: JVM
tags:
- JVM
- 读书笔记
description: 《深入理解Java虚拟机》读书笔记 第二章 Java 内存区域与内存溢出异常
---

> 著名数学家华罗庚先生说：“读一本书要越读越薄。”书越读越薄的过程，就是在多次重复阅读中不断删除冗余信息的过程，浓缩的主要办法是：列提纲与写梗概。前者必须在认真读的基础上，理清文章的脉络，然后逐段概括内容；后者也必须反复阅读,掌握课文要点，将内容加以高度浓缩。浓缩法就是博学反约，厚积薄发，把厚书读薄，又把薄书积厚的读书方法。

![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/深入理解Java虚拟机/2.Java%20内存区域与内存溢出异常/搞笑图片.png)

# 2.1 概述

从概念上介绍 Java 虚拟机内存的各个区域，讲解这些区域的作用、服务对象以及其中可能产生的问题，这是翻越虚拟机内存管理这堵围墙的第一步。

# 2.2 运行时数据区域

![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/深入理解Java虚拟机/2.Java%20内存区域与内存溢出异常/2.2JVM内存结构.png)

JVM 内存结构的布局和相应的控制参数 :

![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/深入理解Java虚拟机/2.Java%20内存区域与内存溢出异常/2.2JVM内存结构2.png)

## 2.2.1 程序计数器

- 一块较小的内存空间，固定宽度的整数的存储空间
- 线程私有
- 当前线程所执行的字节码的行号指示器
- Java 虚拟机规范中唯一一个没有规定 OutOfMemoryError 的区域
- 如果线程正在执行 Java 方法，存储的是正在执行的虚拟机字节码指令的地址；如果正在执行 Native 方法，其值为空(Undefined)。

> 经典问题扩展 : [Java 程序计数器为什么不规定 OutOfMemoryError ？](https://www.zhihu.com/question/49597071)

## 2.2.2 Java 虚拟机栈(Java Virtual Machine Stacks)

- 线程私有，生命周期与线程相同
- 运行 Java 方法( 字节码 ) 服务
- 描述的是 Java 方法执行的内存模型 : 栈帧(Stack Frame)，包含：局部变量表、操作数栈、动态链接、方法出口等
- StackOverflowError 异常：如果线程请求的栈深度大于虚拟机所允许的深度，抛出此异常
- OutOfMemoryError 异常：如果虚拟机栈可以动态扩展，如果扩展时无法申请到足够的内存，抛出此异常

## 2.2.3 本地方法栈(Native Method Stack)

- 线程私有，生命周期与线程相同
- 运行 Native方法服务
- 与Java虚拟机栈相似，也会抛出StackOverflowError异常和OutOfMemoryError异常

## 2.2.4 Java 堆(Java Heap)
- JVM 所管理的内存中最大的一块，垃圾回收的主要操作区域
- 所有线程共享，虚拟机启动时创建
- 所有对象实例以及数组都要在堆上分配(非绝对，JIT 和逃逸分析技术发展)
- 物理上不连续的内存空间，逻辑上是连续的
- 划分为：新生代( Eden空间、From Survivor空间、To Survivor空间 (分配比例 8:1:1) )和老年代
- 控制参数
  - -Xms 设置堆的最小空间大小
  - -Xmx 设置堆的最大空间大小
  - -XX:NewSize 设置新生代最小空间大小
  - -XX:MaxNewSize 设置新生代最小空间大小。
- OutOfMemoryError异常：如果堆中没有内存完成实例分配，并且堆也无法扩展，抛出此异常

## 2.2.5 方法区(Method Area)

- 所有线程共享
- Java 虚拟机规范把方法区描述为堆的一个逻辑部分，别名非堆 Non-Heap，包含：类信息、常量、静态变量、即时编译器编译后的代码等数据
- HotSpot 虚拟机称为“永久代”(Permanent Generation)
- 回收效率并不高
- OutOfMemoryError异常：当方法区无法满足内存分配需求时，抛出此异常

## 2.2.6 运行时常量池(Runtime Constant Pool)

- 方法区一部分，所有线程共享
- 存储编译期生成的各种字面量和符号引用
- OutOfMemoryError 异常：方法区一部分，受到方法区内存限制，当常量池无法再申请到内存时，抛出此异常
- 扩展 [深入解析String#intern](https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html)

## 2.2.7 直接内存(Direct Memory)

- 并不是虚拟机运行时数据区的一部分，也不是 Java 虚拟机规范中定义的内存区域
- OutOfMemoryError 异常：配置虚拟机参数时，使得各个内存区域总和大于物理内存限制，从而导致动态扩展时出现异常

# 2.3 HotSpot 虚拟机对象探秘

## 2.3.1 对象的创建
1. 类加载检查：new 类名，根据 new 的参数在常量池中定位一个类的符号引用，如果没有找到这个符号引用,说明类还未加载，则进行类的加载/解析和初始化。
2. 虚拟机为对象分配内存(位于堆中)
3. 将分配的内存初始化为零值(不包括对象头)，如果使用 TLAB ，这一过程可以提前至 TLAB 分配时进行
4. 调用对象的`<init>`方法

堆内存分配两种方式：指针碰撞(Bump the Pointer) : Java 堆中的内存是规整的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，分配内存也就是把指针向空闲空间那边移动一段与内存大小相等的距离。例如：Serial、ParNew 等收集器。空闲列表(Free List) : Java 堆中的内存不是规整的，已使用的内存和空闲的内存相互交错，就没有办法简单的进行指针碰撞了。虚拟机必须维护一张列表，记录哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录。例如：CMS 这种基于 Mark-Sweep 算法的收集器。

堆内存分配并发解决方案：对分配内存空间的动作进行同步处理，实际上虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。本地线程分配缓冲 TLAB (Thread Local Allocation Buffer)，把内存分配的动作按照线程划分为在不同的空间之中进行，即每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲(TLAB)。哪个线程要分配内存，就在哪个线程的TLAB上分配。只有TLAB用完并分配新的TLAB时，才需要同步锁定。
## 2.3.2 对象的内存布局

- 对象头( Header )
  - 对象自身运行时数据 ( Mark Word )，包含：哈希码 / GC分代年龄 / 锁状态标志 / 线程持有的锁 / 偏向线程ID / 偏向时间戳
  - 类型指针：对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例
- 实例数据( Instance Data )
  - 对象真正存储的有效信息
  - 程序代码中定义的各种类型的字段内容
  - HotSpot 虚拟机默认分配策略：longs/doubles/ints/shorts/chars/bytes/booleans/oops(Oridinary Object Pointers)
- 对齐填充( Padding ),并不是必然存在的，仅仅起着占位符的作用。

## 2.3.2 对象的访问定位
- 使用句柄：Java 堆中分配一块内存,reference 中存储的就是对象句柄地址,使用句柄来访问的最大好处就是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中实例数据地址，reference 本身不用改变。如下图所示：![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/深入理解Java虚拟机/2.Java%20内存区域与内存溢出异常/2.3.3通过句柄访问对象.png)
- 直接指针：Java 堆中分配一块内存，reference 中存储的就是对象实例地址，HostSpot 使用此种方式，节省了一次指针定位的时间开销，提升了速度。如下图所示：![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/深入理解Java虚拟机/2.Java%20内存区域与内存溢出异常/2.3.3通过直接指针访问对象.png)

# 2.4 实战：OutOfMemoryError 异常

- MacBook Pro Retina, 2.6 GHz Intel Core i7, 16 GB 2133 MHz LPDDR3, OS X Yosemite
- JDK 1.8

## 2.4.1 Java 堆溢出

```java
/**
 * Java 堆内存溢出
 * VM Args: -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
 * @author lujiahao
 * @date 2018-12-21 14:47
 */
public class HeapOOM {
    static class OOMObject {

    }

    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<>();
        while (true) {
            list.add(new OOMObject());
            System.out.println(list.size());
        }
    }
}
```

```shell
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid1439.hprof ...
Heap dump file created [28440089 bytes in 0.115 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:265)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:239)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:231)
	at java.util.ArrayList.add(ArrayList.java:462)
	at OutOfMemory.HeapOOM.main(HeapOOM.java:20)

Process finished with exit code 1
```



## 2.4.2 虚拟机栈和本地方法栈溢出

```java
/**
 * 虚拟机栈和本地方法栈溢出
 * VM Args: -Xss128k
 * @author lujiahao
 * @date 2018-12-21 17:02
 */
public class JavaVMStackSOF {
    private int stackLength = 1;

    public void stackLeak() {
        stackLength++;
        stackLeak();
    }

    public static void main(String[] args) {
        JavaVMStackSOF oom = new JavaVMStackSOF();
        try {
            oom.stackLeak();
        } catch (Throwable e) {
            System.out.println("stack length:" + oom.stackLength);
            throw e;
        }
    }
}
```

```shell
设置128k,启动会报下面的问题
The stack size specified is too small, Specify at least 160k
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
```

```shell
修改为161k，实现效果
stack length:7738
Exception in thread "main" java.lang.StackOverflowError
	at OutOfMemory.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:22)
	at OutOfMemory.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:22)
```

```java
/**
 * 创建线程导致内存溢出   危险!!! 可能导致死机,我就不轻易尝试了
 * VM Args: -Xss2M
 * @author lujiahao
 * @date 2018-12-21 17:11
 */
public class JavaVMStackOOM {
    private void dontStop() {
        while (true) {

        }
    }

    public void stackLeakByThread() {
        while (true) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    dontStop();
                }
            });
            thread.start();
        }
    }

    public static void main(String[] args) {
        JavaVMStackOOM oom = new JavaVMStackOOM();
        oom.stackLeakByThread();
    }
}
```



## 2.4.3 方法区和运行时常量池溢出

```java
/**
 * 运行时常量池导致内存溢出
 * VM Args: -XX:PermSize=10m -XX:MaxPermSize=10M
 * jdk1.6
 *
 * 使用新版本的jdk会输出:
 * Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=10m; support was removed in 8.0
 * Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=10M; support was removed in 8.0
 *
 * @author lujiahao
 * @date 2018-12-21 17:33
 */
public class RuntimeConstantPoolOOM {
    public static void main(String[] args) {
        // 使用List保持炸常量池引用,避免Full GC回收常量池行为
        List<String> list = new ArrayList<>();
        // 10MB的PermSize在Integer范围内足够产生OOM了
        int i = 0;
        while (true) {
            list.add(String.valueOf(i++).intern());
        }
    }
}
```

```java
/**
 * 借助CGLib使方法区出现内存溢出
 * VM Args: -XX:PermSize=10M -XX:MaxPermSize=10M
 * @author lujiahao
 * @date 2018-12-21 17:40
 */
public class JavaMethodAreaOOM {
    static class OOMObject{}
    public static void main(String[] args) {
        while (true) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(OOMObject.class);
            enhancer.setUseCache(false);
            enhancer.setCallback(new MethodInterceptor() {
                @Override
                public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                    return methodProxy.invokeSuper(objects, args);
                }
            });
            enhancer.create();
        }
    }
}
```



## 2.4.4 本机直接内存溢出

```java
/**
 * 本机直接内存溢出(使用unsafe分配本机内存)
 * VM Args: -Xmx20M -XX:MaxDirectMemorySize=10M
 * @author lujiahao
 * @date 2018-12-21 17:51
 */
public class DirectMemoryOOM {
    public static final int _1MB = 1024 * 1024;

    public static void main(String[] args) throws Exception{
        Field unsafeField = Unsafe.class.getDeclaredFields()[0];
        unsafeField.setAccessible(true);
        Unsafe unsafe = (Unsafe) unsafeField.get(null);
        while (true) {
            unsafe.allocateMemory(_1MB);
        }
    }
}
```

----
欢迎大家关注😁
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg)

