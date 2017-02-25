---
title: JAVA 创建线程
date: 2017-02-24 10:54:10
tags: [java,并发,线程]
categories: [java并发]
---

Java提供了线程类Thread来创建多线程的程序。其实，创建线程与创建普通的类的对象的操作是一样的，而线程就是Thread类或其子类的实例对象。每个Thread对象描述了一个单独的线程。要产生一个线程，有两种方法：
- 需要从Java.lang.Thread类派生一个新的线程类，重写它的run()方法； 
- 实现Runnalbe接口，重载Runnalbe接口中的run()方法。

**在Java中，类仅支持单继承，也就是说，当定义一个新的类的时候，它只能扩展一个外部类.这样，如果创建自定义线程类的时候是通过扩展 Thread类的方法来实现的，那么这个自定义类就不能再去扩展其他的类，也就无法实现更加复杂的功能。因此，如果自定义类必须扩展其他的类，那么就可以使用实现Runnable接口的方法来定义该类为线程类，这样就可以避免Java单继承所带来的局限性。
还有一点最重要的就是使用实现Runnable接口的方式创建的线程可以处理同一资源，从而实现资源的共享.**
## 通过扩展Thread类来创建多线程
```java
class MyThread2 extends Thread {
    private int ticket = 5;

    public void run() {
        for (int i = 0; i < 10; i++) {
            if (ticket > 0) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " ticket = " + ticket--);
            }
        }
    }
}

public class ThreadDemo {
    public static void main(String[] args) {
        new MyThread2().start();
        new MyThread2().start();
        new MyThread2().start();
    }
}
```
```
结果：
Thread-0 ticket = 5
Thread-1 ticket = 5
Thread-2 ticket = 5
Thread-1 ticket = 4
Thread-2 ticket = 4
Thread-0 ticket = 4
Thread-2 ticket = 3
Thread-0 ticket = 3
Thread-1 ticket = 3
Thread-2 ticket = 2
Thread-0 ticket = 2
Thread-1 ticket = 2
Thread-2 ticket = 1
Thread-0 ticket = 1
Thread-1 ticket = 1
```
程序中定义一个线程类，它扩展了Thread类。利用扩展的线程类在MyThread2类的主方法中创建了三个线程对象，并通过start()方法分别将它们启动。
从结果可以看到，每个线程分别对应5张电影票，之间并无任何关系，这就说明每个线程之间是平等的，没有优先级关系，因此都有机会得到CPU的处理。但是结果
显示这三个线程并不是依次交替执行，而是在三个线程同时被执行的情况下，有的线程被分配时间片的机会多，票被提前卖完，而有的线程被分配时间片的机会比较
少，票迟一些卖完。
**可见，利用扩展Thread类创建的多个线程，虽然执行的是相同的代码，但彼此相互独立，且各自拥有自己的资源，互不干扰。**
## 通过实现Runnable接口来创建多线程
``` java
class MyThread3 implements Runnable {
    private int ticket = 5;

    public void run() {
        for (int i = 0; i < 10; i++) {
            if (ticket > 0) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " ticket = " + ticket--);
            }
        }
    }
}

public class RunnableDemo {
    public static void main(String[] args) {
        MyThread3 my1 = new MyThread3();
        MyThread3 my2 = new MyThread3();
        MyThread3 my3 = new MyThread3();
        new Thread(my1).start();
        new Thread(my2).start();
        new Thread(my3).start();
    }
}

```
```
结果：
Thread-0 ticket = 5
Thread-1 ticket = 5
Thread-2 ticket = 5
Thread-2 ticket = 4
Thread-1 ticket = 4
Thread-0 ticket = 4
Thread-2 ticket = 3
Thread-0 ticket = 3
Thread-1 ticket = 3
Thread-1 ticket = 2
Thread-0 ticket = 2
Thread-2 ticket = 2
Thread-2 ticket = 1
Thread-1 ticket = 1
Thread-0 ticket = 1
```

由于这三个线程也是彼此独立，各自拥有自己的资源，即100张电影票，因此程序输出的结果和 1 结果大同小异。均是各自线程对自己的5张票进行单独的处理，互不影响。
可见，只要现实的情况要求保证新建线程彼此相互独立，各自拥有资源，且互不干扰，采用哪个方式来创建多线程都是可以的。因为这两种方式创建的多线程程序能够实现相同的功能。
## 通过实现Runnable接口来实现线程间的资源共享
``` java
class MyThread3 implements Runnable {
    private int ticket = 5;

    public void run() {
        for (int i = 0; i < 10; i++) {
            if (ticket > 0) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " ticket = " + ticket--);
            }
        }
    }
}

public class RunnableDemo {
    public static void main(String[] args) {
        MyThread3 my = new MyThread3();
        new Thread(my).start();
        new Thread(my).start();
        new Thread(my).start();
    }
}
```
```
结果：
Thread-2 ticket = 5
Thread-1 ticket = 4
Thread-0 ticket = 3
Thread-2 ticket = 2
Thread-0 ticket = 1
Thread-1 ticket = 0
Thread-2 ticket = -1
```
序在内存中仅创建了一个资源，而新建的三个线程都是基于访问这同一资源的，并且由于每个线程上所运行的是相同的代码，因此它们执行的功能也是相同的。可见，如果现实问题中要求必须创建多个线程来执行同一任务，而且这多个线程之间还将共享同一个资源，那么就可以使用实现Runnable接口的方式来创建多线程程序。
结果出现-1的原因是多线程并发访问资源ticket时，没有多线程进行线程同步。修改后代码如下：
```java
class MyThread3 implements Runnable {
    private Integer ticket = 5;

    public void run() {
        for (int i = 0; i < 10; i++) {
            try {
//                System.out.println(Thread.currentThread().getName() + " sleep ... , i = " + i + ", ticket = " + ticket);
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (ticket) { // 使用synchronized同步
                if (ticket > 0) {
                    System.out.println(Thread.currentThread().getName() + " ticket = " + ticket--);
                }
            }
        }
    }
}

public class RunnableDemo {
    public static void main(String[] args) {
        MyThread3 my = new MyThread3();
        new Thread(my).start();
        new Thread(my).start();
        new Thread(my).start();
    }
}
```
```
结果：
Thread-1 ticket = 5
Thread-0 ticket = 4
Thread-2 ticket = 3
Thread-0 ticket = 2
Thread-2 ticket = 1
```

**实现Runnable接口相对于扩展Thread类来说，具有无可比拟的优势。这种方式不仅有利于程序的健壮性，使代码能够被多个线程共享，而且代码和数据资源相对独立，从而特别适合多个具有相同代码的线程去处理同一资源的情况。这样一来，线程、代码和数据资源三者有效分离，很好地体现了面向对象程序设计的思想。因此，几乎所有的多线程程序都是通过实现Runnable接口的方式来完成的。**



