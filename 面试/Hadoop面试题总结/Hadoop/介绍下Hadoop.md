**1、先说下Hadoop是什么**  

Hadoop是一个分布式系统基础架构，主要是为了解决<font color=red>海量数据的存储</font>和<font color=red>海量数据的分析</font>计算问题。  

**2、说下Hadoop核心组件**  

Hadoop自诞生以来，主要有Hadoop 1.x、2.x、3.x三个系列多个版本；  

Hadoop 1.x组成：HDFS（具有高可靠性、高吞吐量的分布式文件系统，用于数据存储），MapReduce（同时处理业务逻辑运算和资源的调度），Common（辅助工具，为其它Hadoop模块提供基础设施）；  

Hadoop 2.x和Hadoop 3.x组成上无变化，和Hadoop 1.x相比，增加了YARN，分担了MapReduce的工作，组件包括：HDFS（具有高可靠性、高吞吐量的分布式文件系统，用于数据存储），MapReduce（处理业务逻辑运算），YARN（负责作业调度与集群资源管理），Common（辅助工具，为其它Hadoop模块提供基础设施）。  

**3、Hadoop核心组件作用**  

![Hadoop组成部分](https://github.com/Dr11ft/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93/Hadoop/Pics/Hadoop-Hadoop%E7%BB%84%E6%88%90%E9%83%A8%E5%88%86.png)

Hadoop主要组件如上图，主要是HDFS、MapReduce、YARN、Common

**HDFS**

HDFS是一个文件系统，用于存储文件，通过目录树来定位文件。

其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。

HDFS 的使用场景：适合一次写入，多次读出的场景。一个文件经过创建、写入和关闭之后就不需要改变。

**MapReduce**

MapReduce是一个分布式运算程序的编程框架，是用户开发“基于Hadoop的数据分析应用”的核心框架。

MapReduce核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个Hadoop集群上。

MapReduce将计算过程分为两个阶段：Map和Reduce

Map阶段并行处理输入数据

Reduce阶段对Map结果进行汇总

**YARN**

先来看两个问题，在Hadoop中

如何管理集群资源？

如何给任务合理分配资源？

YARN在Hadoop中的作用，就是上面两个问题的答案。Yarn 是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而MapReduce等运算程序则相当于运行于操作系统之上的应用程序。

**Common**

Hadoop体系最底层的一个模块，为Hadoop各子项目提供各种工具，如：配置文件和日志操作等。
