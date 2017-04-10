---
title: python学习笔记—python基础&函数
date: 2017-04-10 14:35:42
tags: [python]
categories: [python]
---


# python 基础

## 数据类型和变量

1. 基本数据类型
	- 整数型：任意大小整数，无short,int,long区分 0x表示16进制
	- 浮点型：即小数（科学计数法小数点可浮动）。 ***PS：整数运算永远准确，浮点数运算存在四舍五入***
	- 字符串：用''或""括起来的内容 。 ***PS：转义字符：‘\’。不转义：r'...'。多行内容：'''...'''***
	- 布尔值：True，False。*PS：and or not运算*
	- 空值：None
2. 变量：下划线、字母、数字组合不能以数字开头。 ***PS：动态语言，变量无类型***
3. 常量：名称通常大写。**实际无法约束常量不变**


## 字符串与编码

1. 字符编码
	- 一个比特位只能表示0或者1
	- 8个比特位表示一个字节
	- n个字节可以表示一个字符（n视编码方式而定）
	- ASCII编码：一个字节，只能表示英文及常用字符
	- Unicode编码：常用两个字节表示，通用
	- UTF-8编码：变字节长度，英文一个字节，兼容ASCII，中文３个
	- **存储和网络传输常用UTF-8，内存中常用Unicode**
2. 字符串
	- ord()函数：取字符整数表示
	- chr()函数：取整数字符表示
	- b'...'表示比特类型数据
	- .encode('utf-8,ascii,unicode')函数，将字符按指定编码方式编码为比特型
	- .decode('utf-8,ascii,unicode')函数，将比特按指定编码方式编码为字符
3. 格式化字符串
	- %d 整数型
	- %s 字符串型
	- %f 浮点数型
	- %x 十六进制整数型 PS：整数和浮点数型可指定是否补0或小数点位数（%02d,%.2f）


## list和tuple
1. list(列表)
	- list是一种有序的集合，可以随时添加和删除其中的元素
	- 定义：mylist = [e1,e2...]
2. tuple(元祖)	
	-  tuple和list非常类似，但是tuple一旦初始化就不能修改
	-  定义：mytuple = (e1,e2...)
3. list和tuple是Python内置的有序集合，一个可变，一个不可变。


## dict 与 set
1. dict(字典，同java map)
	- 使用哈希函数实现，查找快，占用内存多
	- 定义：mydict = {key1:value,key2:value2,...,keyn:valuen} 
2. set(集合)
	- key集合，key不重复 
	- 定义：myset = set([一个列表])
3. 不可变对象
	- <font color=red>**通过对象自身的任何方法都不能改变对象自身的内容**</font>
	- ps: 通过对象自身的任何方法都不能改变对象自身的内容


# 函数

## 定义
1. 定义一个函数要使用def语句，依次写出函数名、括号、括号中的参数和冒号:，然后，在缩进块中编写函数体，函数的返回值用return语句返回
```
def function_name(x):
        return x
```
2. 空函数: pass可以用来作为占位符，比如现在还没想好怎么写函数的代码，就可以先放一个pass，让代码能运行起来
```
def nop():
    pass
```
3. 返回多个值: 函数可以同时返回多个值，但其实就是一个tuple。
```
import math

def move(x, y, step, angle=0):
    nx = x + step * math.cos(angle)
    ny = y - step * math.sin(angle)
    return nx, ny
```

## 函数参数
1. 位置参数
2. 默认参数
3. 可变参数
4. 关键字参数
5. 命名关键字参数


