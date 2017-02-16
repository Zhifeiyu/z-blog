---
title: impala的原理架构
date: 2017-02-16 20:05:26
tags: [hadoop,impala,大数据]
categories: [技术积累]
---
# 概述
由cloudera公司主导开发的大数据实时查询分析工具，宣称比原来基于MapReduce的HiveSQL查询速度提升3~90倍，且更加灵活易用。提供类SQL的查询语句，能够查询存储在Hadoop的HDFS和Hbase中的PB级大数据。查询速度快是其最大的卖点。简言之impala作为大数据实时查询分析工具，具有查询速度快，灵活性高，易整合，可伸缩性强等特点。
1. 查询速度快。Impala不同于Hive，hive底层执行使用的是MapReduce引擎，仍然是一个批处理过程。不同于hive，impala中间结果不写入磁盘，即使及时通过网络以流的形式传递，大大降低的节点的IO开销。
2. 灵活性高。可以直接查询存储在HDFS上的原生数据，也可以查询经过优化设计而存储的数据，只需要数据的格式能够兼容MapReduce、hive、Pig等等。
3. 易整合。很容易和hadoop系统整合，并使用hadoop生态系统的资源和优势，不需要将数据迁移到特定的存储系统就能满足查询分析的要求。
4. 可伸缩性。可以很好的与一些BI应用系统协同工作，如Microstrategy、Tableau、Qlikview等。

# 架构
<center>![](http://p1.bpimg.com/567571/e5d78868e694ffd2.jpg)</center>
Impala没有再使用缓慢的Hive+MapReduce批处理，而是通过使用与商用并行关系数据库中类似的分布式查询引擎（由Query Planner、Query Coordinator和Query Exec Engine三部分组成），可以直接从HDFS或HBase中用SELECT、JOIN和统计函数查询数据，从而大大降低了延迟。
Impala主要由Impalad， State Store和CLI组成。
- **Impalad**: 与DataNode运行在同一节点上，由Impalad进程表示，它接收客户端的查询请求（接收查询请求的Impalad为Coordinator，Coordinator通过JNI调用java前端解释SQL查询语句，生成查询计划树，再通过调度器把执行计划分发给具有相应数据的其它Impalad进行执行），读写数据，并行执行查询，并把结果通过网络流式的传送回给Coordinator，由Coordinator返回给客户端。同时Impalad也与State Store保持连接，用于确定哪个Impalad是健康和可以接受新的工作。在Impalad中启动三个ThriftServer: beeswax_server（连接客户端），hs2_server（借用Hive元数据）， be_server（Impalad内部使用）和一个ImpalaServer服务。
- **Impala State Store**: 跟踪集群中的Impalad的健康状态及位置信息，由statestored进程表示，它通过创建多个线程来处理Impalad的注册订阅和与各Impalad保持心跳连接，各Impalad都会缓存一份State Store中的信息，当State Store离线后（Impalad发现State Store处于离线时，会进入recovery模式，反复注册，当State Store重新加入集群后，自动恢复正常，更新缓存数据）因为Impalad有State Store的缓存仍然可以工作，但会因为有些Impalad失效了，而已缓存数据无法更新，导致把执行计划分配给了失效的Impalad，导致查询失败。
- **CLI**: 提供给用户查询使用的命令行工具（Impala Shell使用python实现），同时Impala还提供了Hue，JDBC， ODBC使用接口。
上图可以看出，位于Datanode上的每个impalad进程，都具有Query Planner,QueryCoordinator,Query ExecEnginer这几个组件，每个impala节点在功能上是对等的，也就是说，任何一个节点都能接受外部查询请求。当有一个节点发生故障后，其他节点仍然能够接管，这还得益于HDFS的数据冗余备份机制，即使某个impalad节点挂掉，只要挂掉的节点上的数据在其他节点上有备份，仍然是可以计算的。

# 查询处理过程

<center>![](http://i1.piimg.com/567571/3149d21b5be7b352.png)</center>

# Impala VS Hive
 Impala与Hive都是构建在Hadoop之上的数据查询工具各有不同的侧重适应面，但从客户端使用来看Impala与Hive有很多的共同之处，如数据表元数据、ODBC/JDBC驱动、SQL语法、灵活的文件格式、存储资源池等。Impala与Hive在Hadoop中的关系如图 2所示。**Hive适合于长时间的批处理查询分析，而Impala适合于实时交互式SQL查询**。可以先使用hive进行数据转换处理，之后使用Impala在Hive处理后的结果数据集上进行快速的数据分析。
<center>![](http://p1.bpimg.com/567571/f027ca73464a765a.jpg)</center>
## 异同
- **数据存储**：使用相同的存储数据池都支持把数据存储于HDFS, HBase。
- **元数据**：两者使用相同的元数据。
- **SQL解释处理**：比较相似都是通过词法分析生成执行计划。
- **执行计划**：
 - Hive:依赖于MapReduce执行框架，执行计划分成 map->shuffle->reduce->map->shuffle->reduce…的模型。如果一个Query会 被编译成多轮MapReduce，则会有更多的写中间结果。由于MapReduce执行框架本身的特点，过多的中间过程会增加整个Query的执行时间。
 - Impala:把执行计划表现为一棵完整的执行计划树，可以更自然地分发执行计划到各个Impalad执行查询，而不用像Hive那样把它组合成管道型的map->reduce模式，以此保证Impala有更好的并发性和避免不必要的中间sort与shuffle。
- **数据流**：
 - Hive:采用推的方式，每一个计算节点计算完成后将数据主动推给后续节点。
 - Impala:采用拉的方式，后续节点通过getNext主动向前面节点要数据，以此方式数据可以流式的返回给客户端，且只要有1条数据被处理完，就可以立即展现出来，而不用等到全部处理完成，更符合SQL交互式查询使用。
- **内存使用**：
 - Hive:在执行过程中如果内存放不下所有数据，则会使用外存，以保证Query能顺 序执行完。每一轮MapReduce结束，中间结果也会写入HDFS中，同样由于MapReduce执行架构的特性，shuffle过程也会有写本地磁盘的操作。
 - Impala:在遇到内存放不下数据时，当前版本1.0.1是直接返回错误，而不会利用外存，以后版本应该会进行改进。这使用得Impala目前处理Query会受到一定的限制，最好还是与Hive配合使用。Impala在多个阶段之间利用网络传输数据，在执行过程不会有写磁盘的操作（insert除外）。
- **调度**：
 - Hive:任务调度依赖于Hadoop的调度策略。
 - Impala:调度由自己完成，目前只有一种调度器simple-schedule，它会尽量满足数据的局部性，扫描数据的进程尽量靠近数据本身所在的物理机器。调度器目前还比较简单，在SimpleScheduler::GetBackend中可以看到，现在还没有考虑负载，网络IO状况等因素进行调度。但目前Impala已经有对执行过程的性能统计分析，应该以后版本会利用这些统计信息进行调度吧。
- **容错**：
 - Hive:依赖于Hadoop的容错能力。
 - Impala:在查询过程中，没有容错逻辑，如果在执行过程中发生故障，则直接返回错误（这与Impala的设计有关，因为Impala定位于实时查询，一次查询失败，再查一次就好了，再查一次的成本很低）。但从整体来看，Impala是能很好的容错，所有的Impalad是对等的结构，用户可以向任何一个 Impalad提交查询，如果一个Impalad失效，其上正在运行的所有Query都将失败，但用户可以重新提交查询由其它Impalad代替执行，不会影响服务。对于State Store目前只有一个，但当State Store失效，也不会影响服务，每个Impalad都缓存了State Store的信息，只是不能再更新集群状态，有可能会把执行任务分配给已经失效的Impalad执行，导致本次Query失败。
- **适用面**：
 - Hive:复杂的批处理查询任务，数据转换任务。
 - Impala：实时数据分析，因为不支持UDF，能处理的问题域有一定的限制，与Hive配合使用,对Hive的结果数据集进行实时分析。

## Impala相对于Hive的优化技术
- 没有使用 MapReduce进行并行计算，虽然MapReduce是非常好的并行计算框架，但它更多的面向批处理模式，而不是面向交互式的SQL执行。与MapReduce相比：Impala把整个查询分成一执行计划树，而不是一连串的MapReduce任务，在分发执行计划后，Impala使用拉式获取数据的方式获取结果，把结果数据组成按执行树流式传递汇集，减少的了把中间结果写入磁盘的步骤，再从磁盘读取数据的开销。Impala使用服务的方式避免每次执行查询都需要启动的开销，即相比Hive没了MapReduce启动时间。
- 使用LLVM产生运行代码，针对特定查询生成特定代码，同时使用Inline的方式减少函数调用的开销，加快执行效率。
- 充分利用可用的硬件指令（SSE4.2）。
- 更好的IO调度，Impala知道数据块所在的磁盘位置能够更好的利用多磁盘的优势，同时Impala支持直接数据块读取和本地代码计算checksum。
- 通过选择合适的数据存储格式可以得到最好的性能（Impala支持多种存储格式）。
- 最大使用内存，中间结果不写磁盘，及时通过网络以stream的方式传递。

# Impala的优缺点
## 优点
- 支持SQL查询，快速查询大数据。
- 可以对已有数据进行查询，减少数据的加载，转换。
- 多种存储格式可以选择（Parquet, Text, Avro, RCFile, SequeenceFile）。
- 可以与Hive配合使用。

## 缺点
- 不支持用户定义函数UDF。
- 不支持text域的全文搜索。
- 不支持Transforms。
- 不支持查询期的容错。
- 对内存要求高。
 


