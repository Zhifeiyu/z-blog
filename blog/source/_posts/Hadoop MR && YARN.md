---
title: Hadoop MR && YARN
date: 2017-03-15 17:37:40
tags: [hadoop,大数据]
categories: [大数据]
---

> Apache Hadoop 是一个开源软件框架，可安装在一个商用机器集群中，使机器可彼此通信并协同工作，以高度分布式的方式共同存储和处理大量数据。最初，Hadoop 包含以下两个主要组件：**Hadoop Distributed File System (HDFS)** 和**一个分布式计算引擎**，该引擎支持以 MapReduce 作业的形式实现和运行程序。
**MapReduce** 是 Google 推广的一个简单的编程模型，它对以高度并行和可扩展的方式处理大数据集很有用。MapReduce 的灵感来源于函数式编程，用户可将他们的计算表达为 map 和 reduce 函数，将数据作为键值对来处理。Hadoop 提供了一个高级 API 来在各种语言中实现自定义的 map 和 reduce 函数。
Hadoop 还提供了软件基础架构，以一系列 map 和 reduce 任务的形式运行 MapReduce 作业。Map 任务 在输入数据的子集上调用 map 函数。在完成这些调用后，reduce 任务 开始在 map 函数所生成的中间数据上调用 reduce 任务，生成最终的输出。 map 和 reduce 任务彼此单独运行，这支持并行和容错的计算。
最重要的是，Hadoop 基础架构负责处理分布式处理的所有复杂方面：并行化、调度、资源管理、机器间通信、软件和硬件故障处理，等等。得益于这种干净的抽象，实现处理数百（或者甚至数千）个机器上的数 TB 数据的分布式应用程序从未像现在这么容易过，甚至对于之前没有使用分布式系统的经验的开发人员也是如此。

## MR架构

在运行一个mapreduce计算任务时候，任务过程被分为两个阶段：map阶段和reduce阶段，每个阶段都是用键值对（key/value）作为输入（input）和输出（output）。而程序员要做的就是定义好这两个阶段的函数：map函数和reduce函数。

<center>![](http://p1.bpimg.com/567571/782bf941ee647106.png)</center>
<center>map reduce 过程图</center>

## JobClient JobTracker TaskTracker

<center>![](http://p1.bpimg.com/567571/8c0a589b9cf56e5c.png)</center>
<center>MR 架构</center>

1.  JobClient 向 JobTracker 请求一个新的 jobID
2.  检查作业输出说明
3.  计算作业输出划分split
4.  将运行作业所需要的资源（作业的jar文件、配置文件、计算所得的输入划分）复制到一个以作业ID命名的目录中JobTracker的文件系统。
5.  通过调用JobTracker的submitJob()方法，告诉JobTracker作业准备执行
6.  JobTracker接收到submitJob()方法调用后，把此调用放到一个内部队列中，交由作业调
7.  器进行调度，并对其进行初始化
8.  创建运行任务列表，作业调度去首先从共享文件系统中获取JobClient已经计算好的输入划分信息（图中step6），然后为每个划分创建一个Map任务（一个split对应一个map，有多少split就有多少map）。
9.  TaskTracker执行一个简单的循环，定期发送心跳（heartbeat）调用JobTracker

## 运行过程

1. **输入分片（input split）**：在进行map计算之前，mapreduce会根据输入文件计算输入分片（input split），每个输入分片（input split）针对一个map任务，输入分片（input split）存储的并非数据本身，而是一个分片长度和一个记录数据的位置的数组，输入分片（input split）往往和hdfs的block（块）关系很密切，假如我们设定hdfs的块的大小是64mb，如果我们输入有三个文件，大小分别是3mb、65mb和127mb，那么mapreduce会把3mb文件分为一个输入分片（input split），65mb则是两个输入分片（input split）而127mb也是两个输入分片（input split），换句话说我们如果在map计算前做输入分片调整，例如合并小文件，那么就会有5个map任务将执行，而且每个map执行的数据大小不均，这个也是mapreduce优化计算的一个关键点；
2. **map阶段**：就是程序员编写好的map函数了，因此map函数效率相对好控制，而且一般map操作都是本地化操作也就是在数据存储节点上进行；
3. **combiner阶段**：combiner阶段是程序员可以选择的，combiner其实也是一种reduce操作，因此我们看见WordCount类里是用reduce进行加载的。Combiner是一个本地化的reduce操作，它是map运算的后续操作，主要是在map计算出中间文件前做一个简单的合并重复key值的操作，例如我们对文件里的单词频率做统计，map计算时候如果碰到一个hadoop的单词就会记录为1，但是这篇文章里hadoop可能会出现n多次，那么map输出文件冗余就会很多，因此在reduce计算前对相同的key做一个合并操作，那么文件会变小，这样就提高了宽带的传输效率，毕竟hadoop计算力宽带资源往往是计算的瓶颈也是最为宝贵的资源，但是combiner操作是有风险的，使用它的原则是combiner的输入不会影响到reduce计算的最终输入，例如：如果计算只是求总数，最大值，最小值可以使用combiner，但是做平均值计算使用combiner的话，最终的reduce计算结果就会出错；
4. **shuffle阶段**：将map的输出作为reduce的输入的过程就是shuffle了，这个是mapreduce优化的重点地方。这里我不讲怎么优化shuffle阶段，讲讲shuffle阶段的原理，因为大部分的书籍里都没讲清楚shuffle阶段。Shuffle一开始就是map阶段做输出操作，一般mapreduce计算的都是海量数据，map输出时候不可能把所有文件都放到内存操作，因此map写入磁盘的过程十分的复杂，更何况map输出时候要对结果进行排序，内存开销是很大的，map在做输出时候会在内存里开启一个环形内存缓冲区，这个缓冲区专门用来输出的，默认大小是100mb，并且在配置文件里为这个缓冲区设定了一个阀值，默认是0.80（这个大小和阀值都是可以在配置文件里进行配置的），同时map还会为输出操作启动一个守护线程，如果缓冲区的内存达到了阀值的80%时候，这个守护线程就会把内容写到磁盘上，这个过程叫spill，另外的20%内存可以继续写入要写进磁盘的数据，写入磁盘和写入内存操作是互不干扰的，如果缓存区被撑满了，那么map就会阻塞写入内存的操作，让写入磁盘操作完成后再继续执行写入内存操作，前面我讲到写入磁盘前会有个排序操作，这个是在写入磁盘操作时候进行，不是在写入内存时候进行的，如果我们定义了combiner函数，那么排序前还会执行combiner操作。每次spill操作也就是写入磁盘操作时候就会写一个溢出文件，也就是说在做map输出有几次spill就会产生多少个溢出文件，等map输出全部做完后，map会合并这些输出文件。这个过程里还会有一个Partitioner操作，对于这个操作很多人都很迷糊，其实Partitioner操作和map阶段的输入分片（Input split）很像，一个Partitioner对应一个reduce作业，如果我们mapreduce操作只有一个reduce操作，那么Partitioner就只有一个，如果我们有多个reduce操作，那么Partitioner对应的就会有多个，Partitioner因此就是reduce的输入分片，这个程序员可以编程控制，主要是根据实际key和value的值，根据实际业务类型或者为了更好的reduce负载均衡要求进行，这是提高reduce效率的一个关键所在。到了reduce阶段就是合并map输出文件了，Partitioner会找到对应的map输出文件，然后进行复制操作，复制操作时reduce会开启几个复制线程，这些线程默认个数是5个，程序员也可以在配置文件更改复制线程的个数，这个复制过程和map写入磁盘过程类似，也有阀值和内存大小，阀值一样可以在配置文件里配置，而内存大小是直接使用reduce的tasktracker的内存大小，复制时候reduce还会进行排序操作和合并文件操作，这些操作完了就会进行reduce计算了；
5. **reduce阶段**：和map函数一样也是程序员编写的，最终结果是存储在hdfs上的。

## YARN

YARN（Yet Another Resource Negotiator）,下一代MapReduce框架的名称，为了容易记忆，一般称为MRv2（MapReduce version 2）。该框架已经不再是一个传统的MapReduce框架，甚至与MapReduce无关，她是一个通用的运行时框架，用户可以编写自己的计算框架，在该运行环境中运行。用于自己编写的框架作为客户端的一个lib，在运用提交作业时打包即可。

### why YARN instead of MR
#### MR 的缺点
经典 MapReduce 的最严重的限制主要关系到**可伸缩性、资源利用和对与 MapReduce 不同的工作负载的支持**。在 MapReduce 框架中，作业执行受两种类型的进程控制：

- 一个称为 JobTracker 的主要进程，它协调在集群上运行的所有作业，分配要在 TaskTracker 上运行的 map 和 reduce 任务。
- 许多称为 TaskTracker 的下级进程，它们运行分配的任务并定期向 JobTracker 报告进度。
大型的 Hadoop 集群显现出了由单个 JobTracker 导致的可伸缩性瓶颈。

此外，较小和较大的 Hadoop 集群都从未最高效地使用他们的计算资源。在 Hadoop MapReduce 中，每个从属节点上的计算资源由集群管理员分解为固定数量的 map 和 reduce slot，这些 slot 不可替代。设定 map slot 和 reduce slot 的数量后，节点在任何时刻都不能运行比 map slot 更多的 map 任务，即使没有 reduce 任务在运行。这影响了集群的利用率，因为在所有 map slot 都被使用（而且我们还需要更多）时，我们无法使用任何 reduce slot，即使它们可用，反之亦然。
Hadoop 设计为仅运行 MapReduce 作业。随着替代性的编程模型（比如 Apache Giraph 所提供的图形处理）的到来，除 MapReduce 外，越来越需要为可通过高效的、公平的方式在同一个集群上运行并共享资源的其他编程模型提供支持。

#### 原MapReduce框架的不足
- JobTracker是集群事务的集中处理点，存在单点故障
- JobTracker是集群事务的集中处理点，存在单点故障
- 在taskTracker端，用map/reduce task作为资源的表示过于简单，没有考虑到CPU、内存等资源情况，当把两个需要消耗大内存的task调度到一起，很容易出现OOM
- 把资源强制划分为map/reduce slot,当只有map task时，reduce slot不能用；当只有reduce task时，map slot不能用，容易造成资源利用不足。


#### 解决可伸缩性问题
在 Hadoop MapReduce 中，JobTracker 具有两种不同的职责：
- 管理集群中的计算资源，这涉及到维护活动节点列表、可用和占用的 map 和 reduce slots 列表，以及依据所选的调度策略将可用 slots 分配给合适的作业和任务
- 协调在集群上运行的所有任务，这涉及到指导 TaskTracker 启动 map 和 reduce 任务，监视任务的执行，重新启动失败的任务，推测性地运行缓慢的任务，计算作业计数器值的总和，等等

为单个进程安排大量职责会导致重大的可伸缩性问题，尤其是在较大的集群上，JobTracker 必须不断跟踪数千个 TaskTracker、数百个作业，以及数万个 map 和 reduce 任务。相反，TaskTracker 通常近运行十来个任务，这些任务由勤勉的 JobTracker 分配给它们。

为了解决可伸缩性问题，一个简单而又绝妙的想法应运而生：我们减少了单个 JobTracker 的职责，将部分职责委派给 TaskTracker，因为集群中有许多 TaskTracker。在新设计中，这个概念通过将 JobTracker 的双重职责（集群资源管理和任务协调）分开为两种不同类型的进程来反映。

#### YARN 的优点

1. 更快地MapReduce计算
2. 对多框架支持
3. 框架升级更容易

<center>![](http://p1.bpimg.com/567571/1ae5e3582108b3aa.png)</center>
<center>YARN</center>

- ResourceManager 代替集群管理器
- ApplicationMaster 代替一个专用且短暂的 JobTracker
- NodeManager 代替 TaskTracker
- 一个分布式应用程序代替一个 MapReduce 作业

一个全局 ResourceManager 以主要后台进程的形式运行，它通常在专用机器上运行，在各种竞争的应用程序之间仲裁可用的集群资源。
在用户提交一个应用程序时，一个称为 ApplicationMaster 的轻量型进程实例会启动来协调应用程序内的所有任务的执行。这包括监视任务，重新启动失败的任务，推测性地运行缓慢的任务，以及计算应用程序计数器值的总和。有趣的是，ApplicationMaster 可在容器内运行任何类型的任务。
NodeManager 是 TaskTracker 的一种更加普通和高效的版本。没有固定数量的 map 和 reduce slots，NodeManager 拥有许多动态创建的资源容器。