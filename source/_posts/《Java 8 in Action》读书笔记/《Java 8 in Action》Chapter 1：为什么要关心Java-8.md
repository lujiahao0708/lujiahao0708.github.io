---
layout: post
title: 《Java 8 in Action》Chapter 1：为什么要关心Java 8
date: 2019-03-02 14:11:14
categories: Java
tags:
- Java8
- 读书笔记
description: 《Java 8 in Action》Chapter 1：为什么要关心Java 8
---

自1998年 JDK 1.0(Java 1.0) 发布以来，Java 已经受到了学生、项目经理和程序员等一大批活跃用户的欢迎。这一语言极富活力，不断被用在大大小小的项目里。从 Java 1.1(1997年) 一直到 Java 7(2011年)，Java 通过增加新功能，不断得到良好的升级。Java 8 则是在2014年3月发布的。Java 8 所做的改变，在许多方面比 Java 历史上任何一次改变都深远，而且极大的提高了 Java 代码的简洁性。

## 1. lambda 表达式

本文通过筛选苹果的需求引入 Java 8 ，对 inventory 中的苹果按照重量进行排序。
Java 8 之前的版本：
```java 
Collections.sort(inventory, new Comparator<Apple>() {
        public int compare(Apple a1, Apple a2){
            return a1.getWeight().compareTo(a2.getWeight());
        }
});
```

Java 8 版本：
```java
inventory.sort(comparing(Apple::getWeight));
```

通过对比我们不难发现，使用 Java 8 可以编写更为简洁的代码，而且代码读起来更接近问题的描述。

## 2. 方法引用

在 Java 8 之前类（Class）是Java中的一等公民，Java8中将方法和lambda增加为一等公民。方法和lambda作为一等公民，是Java8中方法引用的基础。除了允许(命名)函数成为一等值外，Java 8还体现了更广义的将函数作为值的思想，包括 Lambda1(或匿名函数)。

筛选一个目录中的所有隐藏文件，Java 8 之前版本：
```java
File[] hiddenFiles = new File(“.”).listFiles(new FileFilter() {
    public boolean accept (File file) {
        return file.isHidden();
    }
}
```
 
Java 8 版本：
```java
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```

## 3. 流

在Java8之前，遍历处理集合元素，你得用for-each循环一个个去迭代元素，然后再处理元素。我们把这种数据迭代的方法称为外部迭代。相反，有了Stream API，你根本用不着操心循环的事情。数据处理完全是在库内部进行的。我们把这种思想叫作内部迭代。

Java 8 中对于大数据量的集合，用Stream API(java.util.stream)解决了：集合处理时的套路和晦涩，以及难以利用多核这两个问题。

如下展示 Java 8 中使用 Stream API 并行处理数据：
```java
import static java.util.stream.Collectors.toList;
List<Apple> heavyApples = inventory.parallelStream().filter((Apple a) -> a.getWeight() > 150) .collect(toList());
```

## 4. 默认方法
Java 8中加入默认方法主要是为了支持库设计师，让他们能够写出更容易改进的接口。同时，普通开发者也可以在接口中使用默认方法，在实现类没有实现方法时提供方法内容，进一步方便开发。
```java
List<Apple> heavyApples1 = inventory.stream().filter((Apple a) -> a.getWeight() > 150).collect(toList());
List<Apple> heavyApples2 = inventory.parallelStream().filter((Apple a) -> a.getWeight() > 150).collect(toList());
```

在Java 8之前，List<T>并没有stream或parallelStream方法，它实现 的Collection<T>接口也没有。Java 8 给接口设计者提供了一个扩充接口的方式，而不会破坏现有的代码。Java 8在接口声明 中使用新的default关键字来表示这一点。这样就实现了改变已发布的接口而不破坏已有的实现。

## 总结
本章主要总结Java 8 的主要变化(Lambda表达式、方法引用、流和默认方法)，为后面更进一步学习打下坚实基础。


## 资源获取
- 公众号回复 : Java8 即可获取《Java 8 in Action》中英文版!

## Tips
- 欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 
- 欢迎关注我的公众号：庄里程序猿，读书笔记教程资源第一时间获得！

