---
title: JAVA synchronized 详解
date: 2017-02-20 10:23:48
tags: [java,并发]
categories: [java并发]
---

# 概述
Java语言的关键字，当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码。
1. 当两个并发线程访问同一个对象object中的这个synchronized(this)同步代码块时，一个时间内只能有一个线程得到执行。另一个线程必须等待当前线程执行完这个代码块以后才能执行该代码块；
2. 当一个线程访问object的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该object中的非synchronized(this)同步代码块；
3. 尤其关键的是，当一个线程访问object的一个synchronized(this)同步代码块时，其他线程对object中所有其它synchronized(this)同步代码块的访问将被阻塞；
4. 第三个例子同样适用其它同步代码块。也就是说，当一个线程访问object的一个synchronized(this)同步代码块时，它就获得了这个object的对象锁。结果，其它线程对该object对象所有同步代码部分的访问都被暂时阻塞；
5. 以上规则对其它对象锁同样适用。
# 例子代码
-  **当两个并发线程访问同一个对象object中的这个synchronized(this)同步代码块时，一个时间内只能有一个线程得到执行。另一个线程必须等待当前线程执行完这个代码块以后才能执行该代码块。**
``` java
public class Thread1 implements Runnable {
    @Override
    public void run() {
        synchronized (this) {
            for (int i = 0; i < 5; i++) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " - synchronized loop " + i);
            }
        }
    }

    public static void main(String[] args) {
        Thread1 t1 = new Thread1();
        Thread ta = new Thread(t1, "A");
        Thread tb = new Thread(t1, "B");
        ta.start();
        tb.start();

    }

}
```
```
结果：
A - synchronized loop 0
A - synchronized loop 1
A - synchronized loop 2
A - synchronized loop 3
A - synchronized loop 4
B - synchronized loop 0
B - synchronized loop 1
B - synchronized loop 2
B - synchronized loop 3
B - synchronized loop 4
```

- **当一个线程访问object的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该object中的非synchronized(this)同步代码块。**
``` java
public class Thread2 {
    public void m4t1() {
        synchronized (this) {
            int i = 5;
            while (i-- > 0) {
                System.out.println(Thread.currentThread().getName() + " : " + i);
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public void m4t2() {
        int i = 5;
        while (i-- > 0) {
            System.out.println(Thread.currentThread().getName() + " : " + i);
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        final Thread2 myt2 = new Thread2();
        Thread t1 = new Thread(myt2::m4t1, "t1");
        Thread t2 = new Thread(myt2::m4t2, "t2");
        t1.start();
        t2.start();
    }
}
```

- **当一个线程访问object的一个synchronized(this)同步代码块时，其他线程对object中所有其它synchronized(this)同步代码块的访问将被阻塞。** 
``` java
  // 修改Thread2.m4t2()方法：
  public void m4t2() {
        int i = 5;
        synchronized (this) {
            while (i-- > 0) {
                System.out.println(Thread.currentThread().getName() + " : " + i);
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```
```
结果：
t1 : 4
t1 : 3
t1 : 2
t1 : 1
t1 : 0
t2 : 4
t2 : 3
t2 : 2
t2 : 1
t2 : 0
```

- 第三个例子也可以用synchronized方法来实现。**也就是说，当一个线程访问object的一个synchronized(this)同步代码块时，它就获得了这个object的对象锁。**结果，其它线程对该object对象所有同步代码部分的访问都被暂时阻塞。（方法m4t2xx的synchronized关键字可以在public后，也可以在public前）
``` java
    //修改Thread2.m4t2()方法如下：
    public synchronized void m4t2() {
        int i = 5;
        while (i-- > 0) {
            System.out.println(Thread.currentThread().getName() + " : " + i);
            try {
                Thread.sleep(500);
            } catch (InterruptedException ie) {
            }
        }
    }
```
```
结果：
t1 : 4
t1 : 3
t1 : 2
t1 : 1
t1 : 0
t2 : 4
t2 : 3
t2 : 2
t2 : 1
t2 : 0
```
- 以上规则对其它对象锁同样适用
``` java
public class Thread3 {
    class Inner {
        private void m4t1() {
            int i = 5;
            while (i-- > 0) {
                System.out.println(Thread.currentThread().getName() + " : Inner.m4t1()=" + i);
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

        private void m4t2() {
            int i = 5;
            while (i-- > 0) {
                System.out.println(Thread.currentThread().getName() + " : Inner.m4t2()=" + i);
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private void m4t1(Inner inner) {
        synchronized (inner) { //使用对象锁
            inner.m4t1();
        }
    }

    private void m4t2(Inner inner) {
        inner.m4t2();
    }

    public static void main(String[] args) {
        final Thread3 myt3 = new Thread3();
        final Inner inner = myt3.new Inner();
        Thread t1 = new Thread(() -> myt3.m4t1(inner), "t1");
        Thread t2 = new Thread(() -> myt3.m4t2(inner), "t2");
        t1.start();
        t2.start();
    }
}
```
```
结果：
t1 : Inner.m4t1()=4
t2 : Inner.m4t2()=4
t2 : Inner.m4t2()=3
t1 : Inner.m4t1()=3
t2 : Inner.m4t2()=2
t1 : Inner.m4t1()=2
t2 : Inner.m4t2()=1
t1 : Inner.m4t1()=1
t2 : Inner.m4t2()=0
t1 : Inner.m4t1()=0
```
尽管线程t1获得了对Inner的对象锁，但由于线程t2访问的是同一个Inner中的非同步部分。所以两个线程互不干扰。
``` java
        // 现在在Inner.m4t2()前面加上synchronized：
        private synchronized void m4t2() {
            int i = 5;
            while(i-- > 0) {
                System.out.println(Thread.currentThread().getName() + " : Inner.m4t2()=" + i);
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
```
```
结果:
t1 : Inner.m4t1()=4
t1 : Inner.m4t1()=3
t1 : Inner.m4t1()=2
t1 : Inner.m4t1()=1
t1 : Inner.m4t1()=0
t2 : Inner.m4t2()=4
t2 : Inner.m4t2()=3
t2 : Inner.m4t2()=2
t2 : Inner.m4t2()=1
t2 : Inner.m4t2()=0
```
尽管线程t1与t2访问了同一个Inner对象中两个毫不相关的部分,但因为t1先获得了对Inner的对象锁，所以t2对Inner.m4t2()的访问也被阻塞，因为m4t2()是Inner中的一个同步方法。
# 用法
synchronized 关键字，它包括两种用法：synchronized 方法和 synchronized 块。  

- synchronized 方法：通过在方法声明中加入 synchronized关键字来声明 synchronized 方法。 
 
``` java
public synchronized void accessVal(int newVal);
```
synchronized 方法控制对类成员变量的访问：每个类实例对应一把锁，每个 synchronized 方法都必须获得调用该方法的类实例的锁方能执行，否则所属线程阻塞，方法一旦执行，就独占该锁，直到从该方法返回时才将锁释放，此后被阻塞的线程方能获得该锁，重新进入可执行状态。这种机制确保了同一时刻对于每一个类实例，其所有声明为 synchronized 的成员函数中至多只有一个处于可执行状态（因为至多只有一个能够获得该类实例对应的锁），从而有效避免了类成员变量的访问冲突（只要所有可能访问类成员变量的方法均被声明为 synchronized）
- synchronized 块：通过 synchronized关键字来声明synchronized 块。
``` java
synchronized(syncObject) {  
//允许访问控制的代码  
}  
```
synchronized 块是这样一个代码块，其中的代码必须获得对象 syncObject （如前所述，可以是类实例或类）的锁方能执行，具体机制同前所述。由于可以针对任意代码块，且可任意指定上锁的对象，故灵活性较高。
# 小结
总的说来，synchronized关键字可以作为函数的修饰符，也可作为函数内的语句，也就是平时说的同步方法和同步语句块。如果再细的分类，
synchronized可作用于instance变量、object reference（对象引用）、static函数和class literals(类名称字面常量)身上。
我们需要明确几点：
1. **无论synchronized关键字加在方法上还是对象上，它取得的锁都是对象，而不是把一段代码或函数当作锁――而且同步方法很可能还会被其他线程的对象访问**；
2. **每个对象只有一个锁（lock）与之相关联**；
3. **实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制**。

假设P1、P2是同一个类的不同对象，这个类中定义了以下几种情况的同步块或同步方法，P1、P2就都可以调用它们。
- 把synchronized当作函数修饰符时，示例代码如下：
```java
public synchronized void methodAAA()
{
    //….
}
```
这也就是同步方法，那这时synchronized锁定的是哪个对象呢？它锁定的是调用这个同步方法对象。也就是说，当一个对象P1在不同的线程中执行这个同步方法时，它们之间会形成互斥，达到同步的效果。但是这个对象所属的Class所产生的另一对象P2却可以任意调用这个被加了synchronized关键字的方法。
上边的示例代码等同于如下代码：
``` java
    public void methodAAA() {
        synchronized (this)      // (1)
        {
            //…..
        }
    }
```
(1)处的this指的是什么呢？它指的就是调用这个方法的对象，如P1。**可见同步方法实质是将synchronized作用于object reference**。那个拿到了P1对象锁的线程，才可以调用P1的同步方法，而对P2而言，P1这个锁与它毫不相干，程序也可能在这种情形下摆脱同步机制的控制，造成数据混乱。

- 同步块，示例代码如下：
``` java
    public void method3(SomeObject so)
    {
        synchronized (so)
        {
            //….. 
        }
    }
```
这时，锁就是so这个对象，谁拿到这个锁谁就可以运行它所控制的那段代码。当有一个明确的对象作为锁时，就可以这样写程序，但当没有明确的对象作为锁，只是想让一段代码同步时，可以创建一个特殊的instance变量（它得是一个对象）来充当锁：
``` java 
    class Foo implements Runnable {
        private byte[] lock = new byte[0]; // 特殊的instance变量
        public void methodA() {
            synchronized (lock) {
                //…
            }
        }
        //…..
    }
```
**注：零长度的byte数组对象创建起来将比任何对象都经济――查看编译后的字节码：生成零长度的byte[]对象只需3条操作码，而Object lock =  new Object()则需要7行操作码**。

- 将synchronized作用于static 函数，示例代码如下：
```java
class Foo {
    public synchronized static void methodAAA()   // 同步的static 函数
    {
        //….
    }
    public void methodBBB() {
        synchronized (Foo.class)   // class literal(类名称字面常量)
    }
}
```
代码中的methodBBB()方法是把class literal作为锁的情况，它和同步的static函数产生的效果是一样的，取得的锁很特别，是当前调用这个方法的对象所属的类（Class，而不再是由这个Class产生的某个具体对象了）。
***可以推断：如果一个类中定义了一个synchronized的static函数A，也定义了一个synchronized 的instance函数B，那么这个类的同一对象Obj在多线程中分别访问A和B两个方法时，不会构成同步，因为它们的锁都不一样。A方法的锁是Obj这个对象，而B的锁是Obj所属的那个Class。***
还有一些技巧可以让我们对共享资源的同步访问更加安全：
1. 定义private 的instance变量+它的 get方法，而不要定义public/protected的instance变量。如果将变量定义为public，对象在外界可以绕过同步方法的控制而直接取得它，并改动它。这也是JavaBean的标准实现方式之一。
2. 如果instance变量是一个对象，如数组或ArrayList什么的，那上述方法仍然不安全，因为当外界对象通过get方法拿到这个instance对象的引用后，又将其指向另一个对象，那么这private变量也就变了，岂不是很危险。 这个时候就需要将get方法也加上synchronized同步，并且，只返回这个private对象的clone()――这样，调用端得到的就是对象副本的引用了。


　




