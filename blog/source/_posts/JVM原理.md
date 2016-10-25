---
title: JVM原理
date: 2016-10-25 17:15:18
tags: [java,jvm]
categories: [技术积累]
---
## JVM 原理
### 简介

Java编译器和OS平台之间的虚拟处理器。它是一种利用软件方法实现的抽象的计算机基于下层的操作系统和硬件平台，可以在上面执行java的字节码程序。
   
java编译器只要面向JVM，生成JVM能理解的代码或字节码文件。Java源文件经编译成字节码程序，通过JVM将每一条指令翻译成不同平台机器码，通过特定平台运行。

### JAVA运行过程

Java语言写的源程序通过Java编译器，编译成与平台无关的‘字节码程序’(.class文件，也就是0，1二进制程序)，然后在OS之上的Java解释器中解释执行。

![java运行过程](http://oflrm5g9z.bkt.clouddn.com/16-10-25/87103552.jpg)

### JVM执行过程
- 加载 .class文件
- 管理并分配内存
- 执行垃圾收集

![jvm运行过程](http://oflrm5g9z.bkt.clouddn.com/16-10-25/46394905.jpg)

JRE/JDK（java运行时环境）由JVM构造JAVA运行程序

[参考链接](http://www.codeceo.com/article/jvm-stack-heap.html)
