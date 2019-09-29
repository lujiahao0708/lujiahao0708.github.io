---
layout: post
title: 《深入理解Java虚拟机》第六章 类文件结构 
date: 2019-01-12 19:22:27
categories: JVM
tags:
- JVM
- 读书笔记
description: 《深入理解Java虚拟机》读书笔记 第六章 类文件结构
---

# 6.1 概述

代码编译的结果从本地机器码转变为字节码，是存储格式发展的一小步，却是编程语言发展的一大步。

# 6.2 无关性的基石

Java虚拟机有两个无关性，即平台无关性和语言无关性。字节码(ByteCode) 是构成平台无关性的基石。在 Java 发展之初，设计者就曾经考虑过并实现了让其他语言运行在 Java 虚拟机之上的可能性，由此 Java 规范拆分成了 Java语言规范《The Java Language Specification》及 Java 虚拟机规范《The Java Virtual Machine Specification》。

> In the future, we will consider bounded extensions to the Java virtual machine to provide better support for the other languages.

- 语言无关性是指虚拟机并不止执行 Java程序，也考虑让其支持其他语言(Groovy/Scala/Kotlin等)的运行。

- “一次编写，到处运行”。Java的平台无关性即体现在此处，可以在多个平台上运行。



# 6.3 Class 类文件的结构

Class 文件是`一组以8位字节为基础单位的二进制流`。Class 文件采用一种类似于 C 语言结构体的伪结构来存储数据 :

- 无符号数
  - 基本的数据类型
  - u1 / u2 / u4 / u8 分别代表1个字节 / 2个字节 / 4个字节 / 8个字节
  - 可以用来描述数字 / 索引引用 / 数量值或者按照UTF-8编码构成字符串值
- 表
  - 复合数据类型
  - 由多个无符号数或者其他表作为数据项构成，习惯以“_info”结尾
  - 用于描述有层次关系的复合结构的数据，整个 Class 文件本质上就是一张表

![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/深入理解Java虚拟机/6.类文件结构/6.3_Class文件格式.png)

##  6.3.1 魔数与 Class 文件的版本

每个 Class 文件的头4个字节称为魔数(Magic Number)。唯一作用是确定这个文件是否为一个能被虚拟机接受的 Class 文件。

紧接着魔数的4个字节是 Class 文件的版本号 : 第5和第6个字节是次版本号(Minor Version)，第7和第8个字节是主版本号(Major Version)。

将HelloWorld.java编译成 Class 文件后，使用 Synalyze It 16进制编辑工具查看

![](https://github.com/lujiahao0708/PicRepo/raw/master/blogPic/深入理解Java虚拟机/6.类文件结构/6.3.1_JavaClass文件结构.png)

也可以使用命令查看 Class 文件的版本号

```
# javap -v HelloWorld.class 
Classfile /Users/lujiahao/HelloWorld.class
  Last modified 2019-1-12; size 430 bytes
  MD5 checksum f6fad4e65a952d7f01272063b971c2f8
  Compiled from "HelloWorld.java"
public class HelloWorld
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
...
```

同时查看本机 JDK 版本 :

```
# java -version  
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```

通过上述方式我们可以看到 Class 文件的前9个字节的含义。下面是 Class 文件的版本号汇总 :

| 发布版本号 | 内部版本号（十六进制） | 内部版本号（十进制） |
| ---------- | ---------------------- | -------------------- |
| 1.5        | 31                     | 49                   |
| 1.6        | 32                     | 50                   |
| 1.7        | 33                     | 51                   |
| 1.8        | 34                     | 52                   |

### Tips

- [0xCAFEBABE 的由来](https://en.wikipedia.org/wiki/Java_class_file#Magic_Number)
- [Synalyze It 16进制编辑工具](http://www.synalysis.net/)

## 6.3.2 常量池



## 6.3.3 访问标志



## 6.3.4 类索引、父类索引与接口索引集合



## 6.3.5 字段表集合



## 6.3.6 方法表集合



## 6.3.7 属性表集合


----
欢迎大家关注😁
![](https://github.com/lujiahao0708/PicRepo/raw/master/公众号二维码.jpg)