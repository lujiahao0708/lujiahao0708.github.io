---
layout: post
title: 《Java 8 in Action》Chapter 4：引入流
categories: Java
tags:
  - Java8
  - 读书笔记
description: 《Java 8 in Action》Chapter 4：引入流
abbrlink: d8840bff
date: 2019-03-08 11:34:52
---

# 1. 流简介
流是Java API的新成员，它允许你以声明性方式处理数据集合(通过查询语句来表达，而不是临时编写一个实现)。就现在来说，你可以把它们看成遍历数据集的高级迭代器。此外，流还可以透明地并行处理。让我们来看一个实例返回低热量(<400)的菜肴名称：
```java
Java7版本：
List<Dish> lowCaloricDishes = new ArrayList<>();
// 用累加器筛选元素
for(Dish d: menu){
    if(d.getCalories() < 400){
        lowCaloricDishes.add(d);
    }
}
// 用匿名类对菜肴排序
Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
    public int compare(Dish d1, Dish d2){
        return Integer.compare(d1.getCalories(), d2.getCalories());
    }
});
// 处理排序后的菜名列表
List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish d: lowCaloricDishes){
    lowCaloricDishesName.add(d.getName());
}
Java8版本：
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;
List<String> lowCaloricDishesName = menu.stream()
                                        .filter(d -> d.getCalories() < 400)    // 选出400卡路里以下的菜肴
                                        .sorted(comparing(Dish::getCalories))    // 按照卡路里排序
                                        .map(Dish::getName)                    // 提取菜肴名称
                                        .collect(toList());                    // 将所有的名称保存在List中
利用多核架构并行执行，只需要把stream()换成parallelStream()
```

Java 8中的Stream API特性：
- 声明性——更简洁，更易读
- 可复合——更灵活 
- 可并行——性能更好

流定义：
- 元素序列——就像集合一样，流也提供了一个接口，可以访问特定元素类型的一组有序 值。
- 源——流会使用一个提供数据的源，如集合、数组或输入/输出资源。 请注意，从有序集 合生成流时会保留原有的顺序。由列表生成的流，其元素顺序与列表一致。
- 数据处理操作——流的数据处理功能支持类似于数据库的操作，以及函数式编程语言中的常用操作，如filter、map、reduce、find、match、sort等。流操作可以顺序执行，也可并行执行。
- 流水线——很多流操作本身会返回一个流，这样多个操作就可以链接起来，形成一个大的流水线。这让我们下一章中的一些优化成为可能，如延迟和短路。流水线的操作可以看作对数据源进行数据库式查询。
- 内部迭代——与使用迭代器显式迭代的集合不同，流的迭代操作是在背后进行的。

![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/04.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%204%EF%BC%9A%E5%BC%95%E5%85%A5%E6%B5%81/1.%E7%AD%9B%E9%80%89%E8%8F%9C%E8%82%B4.png)

# 2. 流与集合
集合与流之间的差异就在于什么时候进行计算。集合是一个内存中的数据结构，它包含数据结构中目前所有的值——集合中的每个元素都得先算出来才能添加到集合中。相比之下，流则是在概念上固定的数据结构(你不能添加或删除元素)，其元素则是按需计算的。集合和流的另一个关键区别在于它们遍历数据的方式。

## 2.1 只能遍历一次
和迭代器类似，流只能遍历一次。遍历完之后，我们就说这个流已经被消费掉了。以下代码会抛出一个异常，说流已被消费掉了：
```java
List<String> title = Arrays.asList(“Java8”,”In”, “Action”);
Stream<String> s = title.stream();
s.forEach(System.out::println);
s.forEach(System.out::println);

Exception in thread "main" java.lang.IllegalStateException: stream has already been operated upon or closed
	at java.util.stream.AbstractPipeline.sourceStageSpliterator(AbstractPipeline.java:279)
	at java.util.stream.ReferencePipeline$Head.forEach(ReferencePipeline.java:580)
	at com.lujiahao.learnjava8.chapter4.StreamAndCollection.main(StreamAndCollection.java:16)
```
## 2.2 外部迭代与内部迭代
使用Collection接口需要用户去做迭代(比如用for-each)，这称为外部迭代。相反,Streams库使用内部迭代
```java
集合:用for-each循环外部迭代
List<String> names = new ArrayList<>();
for(Dish d: menu){
    names.add(d.getName());
}

集合:用背后的迭代器做外部迭代
List<String> names = new ArrayList<>();
Iterator<String> iterator = menu.iterator();
while(iterator.hasNext()) {
    Dish d = iterator.next();
    names.add(d.getName());
}

流:内部迭代
List<String> names = menu.stream()
                        .map(Dish::getName)
                        .collect(toList());
```
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/04.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%204%EF%BC%9A%E5%BC%95%E5%85%A5%E6%B5%81/2.%E5%86%85%E9%83%A8%E8%BF%AD%E4%BB%A3%E4%B8%8E%E5%A4%96%E9%83%A8%E8%BF%AD%E4%BB%A3.png)

# 3. 流操作
java.util.stream.Stream中的Stream接口定义了许多操作。它们可以分为两大类。可以连接起来的流操作称为中间操作，关闭流的操作称为终端操作。
中间操作：除非流水线上触发一个终端操作，否则中间操作不会执行任何处理。
终端操作：会从流的流水线生成结果。其结果是任何不是流的值。
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/04.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%204%EF%BC%9A%E5%BC%95%E5%85%A5%E6%B5%81/3.%E4%B8%AD%E9%97%B4%E6%93%8D%E4%BD%9C%E4%B8%8E%E7%BB%88%E7%AB%AF%E6%93%8D%E4%BD%9C.png)

流的使用一般包括三件事:
- 一个数据源(如集合)来执行一个查询;
- 一个中间操作链，形成一条流的流水线;
- 一个终端操作，执行流水线，并能生成结果。

> 流的流水线背后的理念类似于构建器模式。

常见流操作：
![](https://raw.githubusercontent.com/lujiahao0708/PicRepo/master/blogPic/04.%E3%80%8AJava%208%20in%20Action%E3%80%8BChapter%204%EF%BC%9A%E5%BC%95%E5%85%A5%E6%B5%81/4.%E5%B8%B8%E8%A7%81%E6%B5%81%E6%93%8D%E4%BD%9C.png)

# 4. 小结
以下是你应从本章中学到的一些关键概念。
- 流是“从支持数据处理操作的源生成的一系列元素”。
- 流利用内部迭代:迭代通过filter、map、sorted等操作被抽象掉了。
- 流操作有两类:中间操作和终端操作。
- filter和map等中间操作会返回一个流，并可以链接在一起。可以用它们来设置一条流水线，但并不会生成任何结果
- forEach和count等终端操作会返回一个非流的值，并处理流水线以返回结果。
- 流中的元素是按需计算的。

## 资源获取
- 公众号回复 : Java8 即可获取《Java 8 in Action》中英文版!

## Tips
- 欢迎收藏和转发，感谢你的支持！(๑•̀ㅂ•́)و✧ 
- 欢迎关注我的公众号：后端小哥，专注后端开发，希望和你一起进步！
