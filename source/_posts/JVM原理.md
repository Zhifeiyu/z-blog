---
title: JVM 简介
tags: java
---

1. **JVM简介**

   Java编译器和OS平台之间的虚拟处理器。它是一种利用软件方法实现的抽象的计算机基于下层的操作系统和硬件平台，可以在上面执行java的字节码程序。
   
   java编译器只要面向JVM，生成JVM能理解的代码或字节码文件。Java源文件经编译成字节码程序，通过JVM将每一条指令翻译成不同平台机器码，通过特定平台运行。

2. **Java运行过程**

    Java语言写的源程序通过Java编译器，编译成与平台无关的‘字节码程序’(.class文件，也就是0，1二进制程序)，然后在OS之上的Java解释器中解释执行。
    
![image](http://static.codeceo.com/images/2016/01/1b726d081f346f16cb0560acc4ecce54.png)

3. **JVM执行过程**
- 加载 .class文件
- 管理并分配内存
- 执行垃圾收集

JRE/JDK（java运行时环境）由JVM构造jav运行程序

![image](http://static.codeceo.com/images/2016/01/cea39daf1b007a3e53805874f4a7d12f.png)


(http://www.codeceo.com/article/jvm-stack-heap.html)