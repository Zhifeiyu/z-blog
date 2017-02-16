---
title: Hbase原理、基本概念、基本架构
date: 2017-02-16 15:02:22
tags: [hadoop,Hbase,大数据]
categories: [大数据]
---
# 概述
HBase是一个构建在HDFS上的分布式列存储系统；
HBase是基于Google BigTable模型开发的，典型的key/value系统；
HBase是Apache Hadoop生态系统中的重要一员，主要用于海量结构化数据存储；
从逻辑上讲，HBase将数据按照表、行和列进行存储。
与hadoop一样，Hbase目标主要依靠横向扩展，通过不断增加廉价的商用服务器，来增加计算和存储能力。
## Hbase 表特点
- **大**：一个表可以有数十亿行，上百万列；
- **无模式**：每行都有一个可排序的主键和任意多的列，列可以根据需要动态的增加，同一张表中不同的行可以有截然不同的列；
- **面向列**：面向列（族）的存储和权限控制，列（族）独立检索；
- **稀疏**：空（null）列并不占用存储空间，表可以设计的非常稀疏；
- **数据多版本**：每个单元中的数据可以有多个版本，默认情况下版本号自动分配，是单元格插入时的时间戳；
- **数据类型单一**：Hbase中的数据都是字符串，没有类型。

# Hbase 数据模型
## Hbase 逻辑视图
<center>![](http://i1.piimg.com/567571/041186b6b6f7c6d5.jpg)</center>
## Hbase 基本概念
- RowKey：是Byte array，是表中每条记录的“主键”，方便快速查找，Rowkey的设计非常重要。
- Column Family：列族，拥有一个名称(string)，包含一个或者多个相关列
- Column：属于某一个columnfamily，familyName:columnName，每条记录可动态添加
- Version Number：类型为Long，默认值是系统时间戳，可由用户自定义
- Value(Cell)：Byte array

# Hbase 物理模型
每个column family存储在HDFS上的一个单独文件中，空值不会被保存。
Key 和 Version number在每个 column family中均有一份；
HBase 为每个值维护了多级索引，即：\<key, column family, column name, timestamp\>
## 物理存储
-  Table中所有行都按照row key的字典序排列；
-  Table在行的方向上分割为多个Region；
- Region按大小分割的，每个表开始只有一个Region，随着数据增多，Region不断增大，当增大到一个阀值的时候，Region就会等分会两个新的Region，之后会有越来越多的Region；
- Region是Hbase中分布式存储和负载均衡的最小单元，不同Region分布到不同RegionServer上;
<center>![](http://p1.bpimg.com/567571/f2b44b62778590fd.png)</center>
- Region虽然是分布式存储的最小单元，但并不是存储的最小单元。Region由一个或者多个Store组成，每个store保存一个columns family；每个Strore又由一个memStore和0至多个StoreFile组成，StoreFile包含HFile；memStore存储在内存中，StoreFile存储在HDFS上。
<center>![](http://i1.piimg.com/567571/cef3cba296a41b07.png)</center>

# Hbase 架构及基本组件
<center>![](http://i1.piimg.com/567571/e5524df5446fab63.jpg)</center>
## 基本组件说明
- **Client**：包含访问HBase的接口，并维护cache来加快对HBase的访问，比如Region的位置信息
- **Master**
 - 为Region server分配Region
 - 负责Region server的负载均衡
 - 发现失效的Region server并重新分配其上的Region
 - 管理用户对table的增删改查操作
- **Region server**
 - Regionserver维护Region，处理对这些Region的IO请求
 - Regionserver负责切分在运行过程中变得过大的region
- **Zookeeper**
 - 通过选举，保证任何时候，集群中只有一个master，Master与RegionServers 启动时会向ZooKeeper注册
 - 存贮所有Region的寻址入口
 - 实时监控Region server的上线和下线信息。并实时通知给Master
 - 存储HBase的schema和table元数据
 - 默认情况下，HBase 管理ZooKeeper 实例，比如， 启动或者停止ZooKeeper
 - Zookeeper的引入使得Master不再是单点故障
- **Write-Ahead-Log（WAL）**(预写式日志)
<center>![](http://p1.bqimg.com/567571/daa136c290a5faf5.png)</center>
该机制用于数据的容错和恢复：
每个HRegionServer中都有一个HLog对象，HLog是一个实现Write Ahead Log的类，在每次用户操作写入MemStore的同时，也会写一份数据到HLog文件中，HLog文件定期会滚动出新的，并删除旧的文件（已持久化到StoreFile中的数据）。当HRegionServer意外终止后，HMaster会通过Zookeeper感知到，HMaster首先会处理遗留的 HLog文件，将其中不同Region的Log数据进行拆分，分别放到相应region的目录下，然后再将失效的region重新分配，领取到这些region的HRegionServer在Load Region的过程中，会发现有历史HLog需要处理，因此会Replay HLog中的数据到MemStore中，然后flush到StoreFiles，完成数据恢复。
## HBase容错性
-  Master容错：Zookeeper重新选择一个新的Master
 - 无Master过程中，数据读取仍照常进行；
 - 无master过程中，region切分、负载均衡等无法进行；
- RegionServer容错：定时向Zookeeper汇报心跳，如果一旦时间内未出现心跳，Master将该RegionServer上的Region重新分配到其他RegionServer上，失效服务器上“预写”日志由主服务器进行分割并派送给新的RegionServer
- Zookeeper容错：Zookeeper是一个可靠地服务，一般配置3或5个Zookeeper实例
## Region定位流程
<center>![](http://i1.piimg.com/567571/bb229506635f0f77.jpg)</center>
- 寻找RegionServer
 ZooKeeper--> -ROOT-(单Region)--> .META.--> 用户表
- -ROOT-
 - 表包含.META.表所在的region列表，该表只会有一个Region；
 - Zookeeper中记录了-ROOT-表的location。
- .META.
 表包含所有的用户空间region列表，以及RegionServer的服务器地址。

# Hbase 使用场景
- 大数据量存储，大数据量高并发操作
- 需要对数据随机读写操作
- 读写访问均是非常简单的操作

# 参考文档
[读和写的流程](http://wenku.baidu.com/view/b46eadd228ea81c758f578f4.html)
[HBase的Region机制](http://blog.csdn.net/dianacody/article/details/39530165)
