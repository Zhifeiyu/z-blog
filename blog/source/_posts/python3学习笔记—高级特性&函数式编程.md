---
title: python3学习笔记—高级特性&函数式编程
date: 2017-04-11 17:54:27
tags: [python]
categories: [python]
---

# python 高级特性

## 切片
1. L[0:3]  取list或tuple或字符串 L前3个元素
2. a = m [ 0 : 100 : 10 ]  #  带步进的切片（步进值=10）
3. str[::-1] 翻转字符串
```
>>> str='pythontab.com'
>>> str[::-1]
'moc.batnohtyp'
>>> 

```


## 迭代
1. for ... in
2. 判断是否可迭代（是否是Iterable对象）：isinstance('abc', Iterable) 
3. enumerate函数可以把一个list变成索引-元素对
4. 引用两个变量
```
>>> for i, value in enumerate(['A', 'B', 'C']):
...     print(i, value)
...
0 A
1 B
2 C
```

## 列表生成式
```
>>> list(range(1, 11))
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

一层循环
```
>>> [x * x for x in range(1, 11)]
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

两层循环
```
>>> [m + n for m in 'ABC' for n in 'XYZ']
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```

三层和三层以上的循环就很少用到。


## 生成器
1. **生成器表达式**
	-  通列表解析语法，只不过把列表解析的[]换成()
	```
	
	>>> L = [x * x for x in range(10)]
	>>> L
	[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
	>>> g = (x * x for x in range(10))
	>>> g
	<generator object <genexpr> at 0x1022ef630>
	
	>>> next(g)
	0
	>>> next(g)
	1
	>>> next(g)
	4
	>>> next(g)
	9
	>>> next(g)
	16
	>>> next(g)
	25
	>>> next(g)
	36
	>>> next(g)
	49
	>>> next(g)
	64
	>>> next(g)
	81
	>>> next(g)
	Traceback (most recent call last):
	  File "<stdin>", line 1, in <module>
	StopIteration
	
	```
2. **生成器函数**
	- 在函数中如果出现了yield关键字，那么该函数就不再是普通函数，而是生成器函数。
	```
	
	def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
    
    >>> f = fib(6)
	>>> f
	<generator object fib at 0x104feaaa0>
	```
3. 生成器函数 通过捕获StopIteration错误获取return返回值，返回值包含在StopIteration的value中
```
>>> g = fib(6)
>>> while True:
...     try:
...         x = next(g)
...         print('g:', x)
...     except StopIteration as e:
...         print('Generator return value:', e.value)
...         break
...
g: 1
g: 1
g: 2
g: 3
g: 5
g: 8
Generator return value: done
``` 


## 迭代器

1. **可迭代对象**：Iterable
	-  集合数据类型，如list、tuple、dict、set、str等
	-  generator，包括生成器和带yield的generator function
2. **迭代器**：Iterator，next()函数调用并不断返回下一个值的对象
	- 生成器都是Iterator对象，但list、dict、str虽然是Iterable，却不是Iterator
	- 把list、dict、str等Iterable变成Iterator可以使用iter()函数
	```
	
	>>> isinstance(iter([]), Iterator)
	True
	>>> isinstance(iter('abc'), Iterator)
	True
	
	```


凡是可作用于for循环的对象都是Iterable类型；
凡是可作用于next()函数的对象都是Iterator类型，它们表示一个惰性计算的序列
	

# 函数式编程

函数式编程就是一种抽象程度很高的编程范式，**纯粹的函数式编程语言编写的函数没有变量**，因此，任意一个函数，只要输入是确定的，输出就是确定的，这种纯函数我们称之为没有副作用。而允许使用变量的程序设计语言，由于函数内部的变量状态不确定，同样的输入，可能得到不同的输出，因此，这种函数是有副作用的。

函数式编程的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数！

Python对函数式编程提供部分支持。由于Python允许使用变量，因此，**Python不是纯函数式编程语言**。

## 高阶函数
1. 函数本身也可以赋值给变量，即：变量可以指向函数
```
>>> f = abs
>>> f(-10)
10
```
2. 函数名也是变量，函数名其实就是指向函数的变量
```
>>> abs = 10
>>> abs(-10)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'int' object is not callable
```
3. 传入函数
```
def add(x, y, f):
    return f(x) + f(y)
```

**把函数作为参数传入，这样的函数称为高阶函数。**

### map/reduce
1. map：map()函数接收两个参数，一个是函数，一个是Iterable，map将传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回
```

>>> def f(x):
...     return x * x
...
>>> r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> list(r)
[1, 4, 9, 16, 25, 36, 49, 64, 81]

```
2. reduce：reduce把一个函数作用在一个序列[x1, x2, x3, ...]上，这个函数必须接收两个参数，一个是函数，一个是Iterable, reduce把结果继续和序列的下一个元素做累积计算，其效果就是：

```
>>> from functools import reduce
>>> def add(x, y):
...     return x + y
...
>>> reduce(add, [1, 3, 5, 7, 9])
25
```


```

>>> from functools import reduce
>>> def fn(x, y):
...     return x * 10 + y
...
>>> reduce(fn, [1, 3, 5, 7, 9])
13579

```

```

>>> from functools import reduce
>>> def fn(x, y):
...     return x * 10 + y
...
>>> def char2num(s):
...     return {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}[s]
...
>>> reduce(fn, map(char2num, '13579'))
13579

```

### filter
和map()类似，filter()也接收一个函数和一个序列。和map()不同的是，filter()把传入的函数依次作用于每个元素，然后根据返回值是True还是False决定保留还是丢弃该元素。

### sort
1. Python内置的sorted()函数就可以对list进行排序
2. sorted()函数也是一个高阶函数，它还可以接收一个key函数来实现自定义的排序

## 返回函数
1. 函数作为返回值
2. 闭包
```
def count():
    fs = []
    for i in range(1, 4):
        def f():
             return i*i
        fs.append(f)
    return fs

f1, f2, f3 = count()

# 预期结果1，4，9， 但实际结果是
>>> f1()
9
>>> f2()
9
>>> f3()
9
```

**原因：返回的函数引用了变量i，但它并非立刻执行。等到3个函数都返回时，它们所引用的变量i已经变成了3，因此最终结果为9**
<font color=red>返回闭包时牢记的一点就是：**返回函数不要引用任何循环变量，或者后续会发生变化的变量**。</font>

如果一定要引用循环变量怎么办？方法是再创建一个函数，用该函数的参数绑定循环变量当前的值，无论该循环变量后续如何更改，已绑定到函数参数的值不变：

```
def count():
    def f(j):
        def g():
            return j*j
        return g
    fs = []
    for i in range(1, 4):
        fs.append(f(i)) # f(i)立刻被执行，因此i的当前值被传入f()
    return fs

>>> f1, f2, f3 = count()
>>> f1()
1
>>> f2()
4
>>> f3()
9
```

## 匿名函数
1. 关键字lambda表示匿名函数，冒号前面的x表示函数参数。
2. 匿名函数有个限制，就是只能有一个表达式，不用写return，返回值就是该表达式的结果。


## 装饰器
在函数调用前后自动打印日志，但又不希望修改now()函数的定义，这种在**代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator）**。
本质上，decorator就是一个返回函数的高阶函数。所以，我们要定义一个能打印日志的decorator，可以定义如下：

```
def log(func):
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper
```

```
@log
def now():
    print('2015-3-25')
```

把@log放到now()函数的定义处，相当于执行了语句：
```
now = log(now)
```

```
>>> now()
call now():
2015-3-25
```

由于log()是一个decorator，返回一个函数，所以，原来的now()函数仍然存在，只是现在同名的now变量指向了新的函数，于是调用now()将执行新函数，即在log()函数中返回的wrapper()函数。
wrapper()函数的参数定义是(*args, **kw)，因此，wrapper()函数可以接受任意参数的调用。在wrapper()函数内，首先打印日志，再紧接着调用原始函数。

如果decorator本身需要传入参数，那就需要编写一个返回decorator的高阶函数，写出来会更复杂。比如，要自定义log的文本：
```
def log(text):
    def decorator(func):
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator
```

```
@log('execute')
def now():
    print('2015-3-25')

>>> now()
execute now():
2015-3-25
```

和两层嵌套的decorator相比，3层嵌套的效果是这样的：
```
>>> now = log('execute')(now)
```
首先执行log('execute')，返回的是decorator函数，再调用返回的函数，参数是now函数，返回值最终是wrapper函数。

看经过decorator装饰之后的函数，它们的__name__已经从原来的'now'变成了'wrapper'：
```
>>> now.__name__
'wrapper'
```
因为返回的那个wrapper()函数名字就是'wrapper'，所以，需要把原始函数的__name__等属性复制到wrapper()函数中，否则，有些依赖函数签名的代码执行就会出错。
Python内置的functools.wraps就是干这个事的，所以，一个完整的decorator的写法如下：
```
import functools

def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper
```

```
import functools

def log(text):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator
```


[参考-装饰器](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014318435599930270c0381a3b44db991cd6d858064ac0000)

## 偏函数
1. functools.partial
2. functools.partial的作用就是，把一个函数的某些参数给固定住（也就是设置默认值），返回一个新的函数