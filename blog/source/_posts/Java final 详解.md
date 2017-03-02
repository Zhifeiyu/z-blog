---
title: Java final 详解
date: 2017-02-20 10:23:48
tags: [java,final]
categories: [java]
---
> final在Java中是一个保留的关键字，可以声明成员变量、方法、类以及本地变量。
> 一旦你将引用声明作final，你将不能改变这个引用了，编译器会检查代码，如果你试图将变量再次初始化的话，编译器会报编译错误。
> **<font color="red">使用final修饰方法参数的目的是防止修改这个参数的值，同时也是一种声明和约定，强调这个参数是不可变的</font>**

# 基本类型 & 引用类型
- 基本类型：它的值就是一个数字，一个字符或一个布尔值；
- 引用类型：是一个对象类型，值是什么呢？**它的值是指向内存空间的引用**，就是地址，所指向的内存中保存着变量所表示的一个值或一组值。 


``` java
     int a;
     a = 250; //声明变量a的同时，系统给a分配了空间。
```
引用类型就不是了，只给变量分配了引用空间，数据空间没有分配，因为谁都不知道数据是什么，整数，字符？我们看一个错误的例子：
``` java
     MyDate today;
     today.day = 4;   // 发生错误，因为today对象的数据空间未分配
```
那我们怎么给它赋值？引用类型变量在声明后必须通过实例化开辟数据空间，才能对变量所指向的对象进行访问。如下：
``` java
    MyDate today;           // 将变量分配一个保存引用的空间
    today = new MyDate();   // 这句话是2步，首先执行new MyDate（），给today变量开辟数据空间，然后再执行赋值操作
    // 引用变量赋值
    MyDate a，b;            // 在内存开辟两个引用空间
    a = new MyDate();       // 开辟MyDate对象的数据空间，并把该空间的首地址赋给a
    b = a;                  // 将a存储空间中的地址写到b的存储空间中
```
内存模型如下图：
![](http://p1.bqimg.com/567571/0c171661e31446bc.png)
# 引用传递 & 值传递
- 引用传递：除了在函数传值的时候是"引用传递"，在任何用"＝"向对象变量赋值的时候都是"引用传递"。
- 值传递：基本类型的传递都属于值传递，和C语言一样，当把Java的基本数据类型（如 int，char，double等）作为入口参数传给函数体的时候，传入的参数在函数体内部变成了局部变量，这个局部变量是输入参数的一个拷贝，所有的函 数体内部的操作都是针对这个拷贝的操作，函数执行结束后，这个局部变量也就完成了它的使命，它影响不到作为输入参数的变量。这种方式的参数传递被称为"值 传递"。


# 基本用法
## final 变量
对于一个final变量，如果是**基本数据类型的变量，则其数值一旦在初始化之后便不能更改**；如果是**引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象**。
当用final作用于类的成员变量时，成员变量（注意是类的成员变量，局部变量只需要保证在使用之前被初始化赋值即可）必须在定义时或者构造器中进行初始化赋值，而且final变量一旦被初始化赋值之后，就不能再被赋值了。
在上面提到被final修饰的引用变量一旦初始化赋值之后就不能再指向其他的对象，那么该引用变量指向的对象的内容可变吗？看下面这个例子：
``` java
public class Test {
    public static void main(String[] args)  {
        final MyClass myClass = new MyClass();
        System.out.println(++myClass.i);
 
    }
}
 
class MyClass {
    public int i = 0;
}
```
这段代码可以顺利编译通过并且有输出结果，输出结果为1。**这说明引用变量被final修饰之后，虽然不能再指向其他对象，但是它指向的对象的内容是可变的。**
## final 方法
使用final方法的原因有两个。第一个原因是**把方法锁定，以防任何继承类修改它的含义**；第二个原因是**效率**。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升。在最近的Java版本中，不需要使用final方法进行这些优化了。
因此，**<font color=red>只有在想明确禁止该方法在子类中被覆盖的情况下才将方法设置为final的</font>**。
***注：类的private方法会隐式地被指定为final方法。***
## final 类
当用final修饰一个类时，表明这个类不能被继承。也就是说，如果一个类你永远不会让他被继承，就可以用final进行修饰。final类中的成员变量可以根据需要设为final，但是要注意final类中的所有成员方法都会被隐式地指定为final方法。
**在使用final修饰类的时候，要注意谨慎选择，除非这个类真的在以后不会用来继承或者出于安全的考虑，尽量不要将类设计为final类。**


# final & static
很多时候会容易把static和final关键字混淆，**<font color="red">static作用于成员变量用来表示只保存一份副本，而final的作用是用来保证变量不可变。</font>**
``` java
public class Test {
    public static void main(String[] args)  {
        MyClass myClass1 = new MyClass();
        MyClass myClass2 = new MyClass();
        System.out.println(myClass1.i);
        System.out.println(myClass1.j);
        System.out.println(myClass2.i);
        System.out.println(myClass2.j);
 
    }
}
 
class MyClass {
    public final double i = Math.random();
    public static double j = Math.random();
}
```
运行这段代码就会发现，每次打印的两个j值都是一样的，而i的值却是不同的。从这里就可以知道final和static变量的区别了。

# final 参数
**在方法参数前面加final关键字就是为了防止数据在方法体中被修改。**
- **用final修饰基本数据类型**：这时参数的值在方法体内是不能被修改的，即不能被重新赋值。否则编译就通不过。
- **用final修饰引用类型**：这时参数变量所引用的对象是不能被改变的。作为引用的拷贝，参数在方法体里面不能再引用新的对象。否则编译通不过。


