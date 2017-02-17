---
title: Java集合类
date: 2017-02-17 15:50:22
tags: [java,collection]
categories: [java]
---
# 基本概念
集合类主要负责保存、盛装其他数据，因此集合类也被称为容器类。所以的集合类都位于java.util包下，后来为了处理多线程环境下的并发安全问题，java5还在java.util.concurrent包下提供了一些多线程支持的集合类。
Java容器类类库的用途是"保存对象"，并将其划分为两个不同的概念：
-  **Collection**：  一组"对立"的元素，通常这些元素都服从某种规则
 -  **List** 必须保持元素特定的顺序
 -  **Set** 不能有重复元素
 - **Queue** 保持一个队列(先进先出)的顺序
- **Map**： 一组成对的"键值对"对象

Collection和Map的区别在于容器中每个位置保存的元素个数:
 - Collection 每个位置只能保存一个元素(对象)
 - Map保存的是"键值对"，就像一个小型数据库。我们可以通过"键"找到该键对应的"值"

# 架构层次关系
```
java.util.Collection [I]
+--java.util.List [I]
   +--java.util.ArrayList [C]
   +--java.util.LinkedList [C]
   +--java.util.Vector [C]
      +--java.util.Stack [C]
+--java.util.Set [I]
   +--java.util.HashSet [C]
   +--java.util.SortedSet [I]
      +--java.util.TreeSet [C]

java.util.Map [I]
+--java.util.SortedMap [I]
   +--java.util.TreeMap [C]
+--java.util.Hashtable [C]
+--java.util.HashMap [C]
+--java.util.LinkedHashMap [C]
+--java.util.WeakHashMap [C]
 
[I]：接口
[C]：类
```
## Collection
Collection是最基本的集合接口，一个Collection代表一组Object的集合，这些Object被称作Collection的元素。
所有实现Collection接口的类都必须提供两个标准的构造函数：无参数的构造函数用于创建一个空的Collection，有一个Collection参数的构造函数用于创建一个新的Collection，这 个新的Collection与传入的Collection有相同的元素。后一个构造函数允许用户复制一个Collection。
如何遍历Collection中的每一个元素？不论Collection的实际类型如何，它都支持一个iterator()的方法，该方法返回一个迭代子，使用该迭代子即可逐一访问Collection中每一个元素。典型的用法如下：
```
Iterator it = collection.iterator(); // 获得一个迭代器
while(it.hasNext()) { 
　　Object obj = it.next(); // 得到下一个元素 
}
```

### List
List继承自Collection接口。List是有序的Collection，使用此接口能够精确的控制每个元素插入的位置。用户能够使用索引（元素在List中的位置，类似于数组下标）来访问List中的元素，这类似于Java的数组。
跟Set集合不同的是，List允许有重复元素。对于满足e1.equals(e2)条件的e1与e2对象元素，可以同时存在于List集合中。当然，也有List的实现类不允许重复元素的存在。
除了具有Collection接口必备的iterator()方法外，List还提供一个listIterator()方法，返回一个 ListIterator接口，和标准的Iterator接口相比，ListIterator多了一些add()之类的方法，允许添加，删除，设定元素， 还能向前或向后遍历。
实现List接口的常用类有LinkedList，ArrayList，Vector和Stack。
#### LinkedList
LinkedList实现了List接口，允许null元素。此外LinkedList提供额外的get，remove，insert方法在 LinkedList的首部或尾部。这些操作使LinkedList可被用作堆栈（stack），队列（queue）或双向队列（deque）。
注意LinkedList没有同步方法。如果多个线程同时访问一个List，则必须自己实现访问同步。一种解决方法是在创建List时构造一个同步的List：
```
List list = Collections.synchronizedList(new LinkedList(...)); 
```
#### ArrayList 
ArrayList实现了可变大小的数组。它允许所有元素，包括null。ArrayList没有同步。
size，isEmpty，get，set方法运行时间为常数。但是add方法开销为分摊的常数，添加n个元素需要O(n)的时间。其他的方法运行时间为线性。
每个ArrayList实例都有一个容量（Capacity），即用于存储元素的数组的大小。这个容量可随着不断添加新元素而自动增加，但是增长算法并 没有定义。当需要插入大量元素时，在插入前可以调用ensureCapacity方法来增加ArrayList的容量以提高插入效率。
和LinkedList一样，ArrayList也是非同步的（unsynchronized）。
#### Vector
vector非常类似ArrayList，但是Vector是同步的。由Vector创建的Iterator，虽然和ArrayList创建的 Iterator是同一接口，但是，因为Vector是同步的，当一个Iterator被创建而且正在被使用，另一个线程改变了Vector的状态（例如，添加或删除了一些元素），这时调用Iterator的方法时将抛出ConcurrentModificationException，因此必须捕获该异常。
##### Stack 
Stack继承自Vector，实现一个后进先出的堆栈。Stack提供5个额外的方法使得Vector得以被当作堆栈使用。基本的push和pop 法，还有peek方法得到栈顶的元素，empty方法测试堆栈是否为空，search方法检测一个元素在堆栈中的位置。Stack刚创建后是空栈。
### Set
Set继承自Collection接口。Set是一种不能包含有重复元素的集合，即对于满足e1.equals(e2)条件的e1与e2对象元素，不能同时存在于同一个Set集合里，换句话说，Set集合里任意两个元素e1和e2都满足e1.equals(e2)==false条件，Set最多有一个null元素。 因为Set的这个制约，在使用Set集合的时候，应该注意：
- **为Set集合里的元素的实现类实现一个有效的equals(Object)方法。**
- **对Set的构造函数，传入的Collection参数不能包含重复的元素。**

*请注意：必须小心操作可变对象（Mutable Object）。如果一个Set中的可变元素改变了自身状态导致Object.equals(Object)=true将导致一些问题。*
#### HashSet
此类实现 Set 接口，由哈希表（实际上是一个 HashMap 实例）支持。它不保证集合的迭代顺序；特别是它不保证该顺序恒久不变。此类允许使用 null 元素。
HashSet不是同步的，需要用以下语句来进行S同步转换：
```
Set s = Collections.synchronizedSet(new HashSet(...));
```
### Map
Map没有继承Collection接口。也就是说Map和Collection是2种不同的集合。Collection可以看作是（value）的集合，而Map可以看作是（key，value）的集合。
Map接口由Map的内容提供3种类型的集合视图，一组key集合，一组value集合，或者一组key-value映射关系的集合。
#### Hashtable
Hashtable继承Map接口，实现一个key-value映射的哈希表。任何非空（non-null）的对象都可作为key或者value。
添加数据使用put(key, value)，取出数据使用get(key)，这两个基本操作的时间开销为常数。
Hashtable 通过initial capacity和load factor两个参数调整性能。通常缺省的load factor 0.75较好地实现了时间和空间的均衡。增大load factor可以节省空间但相应的查找时间将增大，这会影响像get和put这样的操作。
由于作为key的对象将通过计算其散列函数来确定与之对应的value的位置，因此任何作为key的对象都必须实现hashCode和equals方 法。hashCode和equals方法继承自根类Object，如果你用自定义的类当作key的话，要相当小心，按照散列函数的定义，如果两个对象相 同，即obj1.equals(obj2)=true，则它们的hashCode必须相同，但如果两个对象不同，则它们的hashCode不一定不同，如 果两个不同对象的hashCode相同，这种现象称为冲突，冲突会导致操作哈希表的时间开销增大，所以尽量定义好的hashCode()方法，能加快哈希 表的操作。
如果相同的对象有不同的hashCode，对哈希表的操作会出现意想不到的结果（期待的get方法返回null），要避免这种问题，只需要牢记一条：**要同时复写equals方法和hashCode方法，而不要只写其中一个。**
Hashtable是同步的。
#### HashMap
HashMap和Hashtable类似，不同之处在于HashMap是非同步的，并且允许null，即null value和null key。，但是将HashMap视为Collection时（values()方法可返回Collection），其迭代子操作时间开销和HashMap 的容量成比例。因此，如果迭代操作的性能相当重要的话，不要将HashMap的初始化容量设得过高，或者load factor过低。
#### WeakHashMap
WeakHashMap是一种改进的HashMap，它对key实行“弱引用”，如果一个key不再被外部所引用，那么该key可以被GC回收。
## 对集合操作的工具类
- Java提供了java.util.Collections，以及java.util.Arrays类简化对集合的操作。
-  java.util.Collections主要提供一些static方法用来操作或创建Collection，Map等集合。
- java.util.Arrays主要提供static方法对数组进行操作。
# 使用场景
- 如果涉及到堆栈，队列等操作，应该考虑用List，对于需要快速插入，删除元素，应该使用LinkedList，如果需要快速随机访问元素，应该使用ArrayList。
- 如果程序在单线程环境中，或者访问仅仅在一个线程中进行，考虑非同步的类，其效率较高，如果多个线程可能同时操作一个类，应该使用同步的类。
- 在除需要排序时使用TreeSet,TreeMap外,都应使用HashSet,HashMap,因为他们 的效率更高。
- **要特别注意对哈希表的操作，作为key的对象要正确复写equals和hashCode方法。**
- 容器类仅能持有对象引用（指向对象的指针），而不是将对象信息copy一份至数列某位置。一旦将对象置入容器内，便损失了该对象的型别信息。
- 尽量返回接口而非实际的类型，如返回List而非ArrayList，这样如果以后需要将ArrayList换成LinkedList时，客户端代码不用改变。这就是针对抽象编程。

# 注意
1. Collection没有get()方法来取得某个元素。只能通过iterator()遍历元素。
2. Set和Collection拥有一模一样的接口。
3. List，可以通过get()方法来一次取出一个元素。使用数字来选择一堆对象中的一个，get(0)...。(add/get)
4. 一般使用ArrayList。用LinkedList构造堆栈stack、队列queue。
5. Map用 put(k,v) / get(k)，还可以使用containsKey()/containsValue()来检查其中是否含有某个key/value。HashMap会利用对象的hashCode来快速找到key。
6. Map中元素，可以将key序列、value序列单独抽取出来。
 - 使用keySet()抽取key序列，将map中的所有keys生成一个Set。
 - 使用values()抽取value序列，将map中的所有values生成一个Collection.
 - 为什么一个生成Set，一个生成Collection？那是因为，key总是独一无二的，value允许重复。
 

