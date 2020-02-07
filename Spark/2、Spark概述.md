Spark概述
---
### 1、什么是Spark
&emsp; Spark是一种**基于内存的快速、通用、可扩展的大数据分析引擎**  
&emsp; Spark是基于内存计算的大数据并行计算框架，除了扩展了广泛使用的MapReduce计算模型，而且高效地支持更多计算模式，包括交互式查询和流处理。Spark适用于各种各样原先需要多种不同的分布式平台的场景，包括批处理、迭代算法、交互式查询、流处理。通过在一个统一的框架下支持这些不同的计算，Spark使用户可以简单而低耗地把各种处理流程整合在一起。而这样的组合，在实际的数据分析过程中是很有意义的。不仅如此，Spark 的这种特性还大大减轻了原先需要对各种平台分别管理的负担。  

###  2、Spark内置模块
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E5%9F%BA%E7%A1%80/%E5%86%85%E7%BD%AE%E6%A8%A1%E5%9D%97.png">
</p>
</p>  

**`Spark Core`**  
&emsp; Spark Core实现了Spark的基本功能，包含任务调度、内存管理、错误恢复、与存储系统 交互等模块。Spark Core中还包含了对弹性分布式数据集(resilient distributed dataset，简称RDD)的API定义。  
**`Spark SQL`**  
&emsp; Spark SQL是Spark用来操作结构化数据的程序包。通过Spark SQL我们可以使用 SQL或者Apache Hive版本的SQL方言(HQL)来查询数据。Spark SQL支持多种数据源，比如Hive表、Parquet以及JSON等。
**`Spark Streaming`**  
&emsp; Spark Streaming是Spark提供的对实时数据进行流式计算的组件。提供了用来操作数据流的API，并且与Spark Core中的RDD API高度对应。  
**`Spark MLlib`**  
&emsp; 提供常见的机器学习(ML)功能的程序库。包括分类、回归、聚类、协同过滤等，还提供了模型评估、数据 导入等额外的支持功能。  
**`集群管理器`**  
&emsp; Spark 设计为可以高效地在一个计算节点到数千个计算节点之间伸缩计算。为了实现这样的要求，同时获得最大灵活性，Spark支持在各种集群管理器(cluster manager)上运行，包括Hadoop YARN、Apache Mesos，以及Spark自带的一个简易调度器，叫作独立调度器（standalone）。  

### 3、Spark特点
**`快`**  
&emsp; 与Hadoop的MapReduce相比，**Spark基于内存的运算要快100倍以上，基于硬盘的运算也要快10倍以上**。Spark实现了高效的DAG执行引擎，可以通过基于内存来高效处理数据流。计算的中间结果是存在于内存中的。  
**`易用`**  
&emsp; Spark**支持Java、Python和Scala的API，还支持超过80种高级算法**，使用户可以快速构建不同的应用。而且Spark支持交互式的Python和Scala的shell，可以非常方便地在这些shell中使用Spark集群来验证解决问题的方法。  
**`通用`**  
&emsp; Spark提供了统一的解决方案。Spark可以**用于批处理、交互式查询（Spark SQL）、实时流处理（Spark Streaming）、机器学习（Spark MLlib）和图计算（GraphX）**。这些不同类型的处理都可以在同一个应用中无缝使用。Spark统一的解决方案非常具有吸引力，毕竟任何公司都想用统一的平台去处理遇到的问题，减少开发和维护的人力成本和部署平台的物力成本。  
**`兼容性`**  
&emsp; Spark**可以非常方便地与其他的开源产品进行融合**。比如，Spark可以使用Hadoop的YARN和Apache Mesos作为它的资源管理和调度器，并且可以处理所有Hadoop支持的数据，包括HDFS、HBase和Cassandra等。这对于已经部署Hadoop集群的用户特别重要，因为不需要做任何数据迁移就可以使用Spark的强大处理能力。Spark也可以不依赖于第三方的资源管理和调度器，它实现了Standalone作为其内置的资源管理和调度框架，这样进一步降低了Spark的使用门槛，使得所有人都可以非常容易地部署和使用Spark。此外，Spark还提供了在EC2上部署Standalone的Spark集群的工具。  





