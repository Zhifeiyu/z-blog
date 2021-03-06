---
title: JAVA 线程安全
date: 2017-02-19 10:23:48
tags: [java,并发,线程]
categories: [java并发]
---

# java内存模型
java的多线程并发问题最终都会反映在java的内存模型上，所谓线程安全无非是要控制多个线程对某个资源的有序访问或修改。我们都知道计算机有高速缓存的存在，处理器并不是每次处理数据都是取内存的。JVM定义了自己的内存模型，屏蔽了底层平台内存管理细节，对于java开发人员，要清楚在jvm内存模型的基础上，如果解决多线程的可见性和有序性。
## 可见性
多个线程之间是不能互相传递数据通信的，它们之间的沟通只能通过共享变量来进行。Java内存模型（JMM）规定了jvm有主内存，主内存是多个线程共享的。当new一个对象的时候，也是被分配在主内存中，每个线程都有自己的工作内存，工作内存存储了主存的某些对象的副本，当然线程的工作内存大小是有限制的。当线程操作某个对象时，执行顺序如下：
- 从主存复制变量到当前工作内存 (read and load)
-  执行代码，改变共享变量值 (use and assign)
-  用工作内存数据刷新主存相关内容 (store and write)
JVM规范定义了线程对主存的操作指令：read，load，use，assign，store，write。当一个共享变量在多个线程的工作内存中都有副本时，如果一个线程修改了这个共享变量，那么其他线程应该能够看到这个被修改后的值，这就是多线程的可见性问题。

## 有序性
线程在引用变量时不能直接从主内存中引用,如果线程工作内存中没有该变量,则会从主内存中拷贝一个副本到工作内存中,这个过程为read-load,完成后线程会引用该副本。当同一线程再度引用该字段时,有可能重新从主存中获取变量副本(read-load-use),也有可能直接引用原来的副本 (use),也就是说 read,load,use顺序可以由JVM实现系统决定。
 线程不能直接为主存中中字段赋值，它会将值指定给工作内存中的变量副本(assign),完成后这个变量副本会同步到主存储区(store- write)，至于何时同步过去，根据JVM实现系统决定.有该字段,则会从主内存中将该字段赋值到工作内存中,这个过程为read-load,完成后线程会引用该变量副本，当同一线程多次重复对字段赋值时,比如：
``` java
for (int i = 0; i < 10; i++)
    a++;
```
线程有可能只对工作内存中的副本进行赋值,只到最后一次赋值后才同步到主存储区，所以assign,store,weite顺序可以由JVM实现系统决定。假设有一个共享变量x，线程a执行x=x+1。从上面的描述中可以知道x=x+1并不是一个原子操作，它的执行过程如下：
1. `从主存中读取变量x副本到工作内存
2. 给x加1
3.  将x加1后的值写回主 存
如果另外一个线程b执行x=x-1，执行过程如下：
1. 从主存中读取变量x副本到工作内存
2. 给x减1
3.  将x减1后的值写回主存 
那么显然，最终的x的值是不可靠的。假设x现在为10，线程a加1，线程b减1，从表面上看，似乎最终x还是为10，但是多线程情况下会有这种情况发生：
1. 线程a从主存读取x副本到工作内存，工作内存中x值为10
2. 线程b从主存读取x副本到工作内存，工作内存中x值为10
3. 线程a将工作内存中x加1，工作内存中x值为11
4. 线程a将x提交主存中，主存中x为11
5. 线程b将工作内存中x值减1，工作内存中x值为9
6. 线程b将x提交到中主存中，主存中x为9 
同样，x有可能为11，如果x是一个银行账户，线程a存款，线程b扣款，显然这样是有严重问题的，要解决这个问题，必须保证线程a和线程b是有序执行的，并且每个线程执行的加1或减1是一个原子操作。

# 线程同步机制
## synchronized关键字
java用synchronized关键字做为多线程并发环境的执行有序性的保证手段之一。当一段代码会修改共享变量，这一段代码成为互斥区或临界区，为了保证共享变量的正确性，synchronized标示了临界区。典型的用法如下：
```java
synchronized(锁){   
     临界区代码   
}   
```
## volatile关键字 
 volatile是java提供的一种同步手段，只不过它是轻量级的同步，为什么这么说，因为**volatile只能保证多线程的内存可见性，不能保证多线程的执行有序性。而最彻底的同步要保证有序性和可见性，例如synchronized**。任何被volatile修饰的变量，都不拷贝副本到工作内存，任何修改都及时写在主存。因此对于Valatile修饰的变量的修改，所有线程马上就能看到，但是volatile不能保证对变量的修改是有序的。
