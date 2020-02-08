RDD概述
---
### 1、RDD 的产生
&emsp; Hadoop的**MapReduce是一种基于数据集的工作模式，面向数据**，这种工作模式**一般是从存储上加载数据集，然后操作数据集，最后写入物理存储设备**。数据更多面临的是**一次性处理**。  
&emsp; **MR的这种方式对数据领域两种常见的操作不是很高效**。第一种是**迭代式的算法**。比如机器学习中ALS、凸优化梯度下降等。这些都需要基于数据集或者数据集的衍生数据反复查询反复操作。**MR这种模式不太合适，即使多MR串行处理，性能和时间也是一个问题**。数据的共享依赖于磁盘。另外一种是**交互式数据挖掘**，MR显然不擅长。  
&emsp; MR和Spark中的迭代对比：  
&emsp; MR中的迭代  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E6%A6%82%E8%BF%B0/MR%E4%B8%AD%E7%9A%84%E8%BF%AD%E4%BB%A3.png"/>  
<p align="center">
</p>
</p>  

&emsp; Spark中的迭代  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E6%A6%82%E8%BF%B0/Spark%E4%B8%AD%E7%9A%84%E8%BF%AD%E4%BB%A3.png"/>  
<p align="center">
</p>
</p>  

&emsp; Spark是一个效率非常快，且能够支持迭代计算和有效数据共享的模型，适合数据领域常见的两种操作。而且**RDD是基于工作集的工作模式，更多的是面向工作流**。  

### 2、什么是RDD
&emsp; RDD（Resilient Distributed Dataset）叫做**分布式数据集**，是**Spark中最基本的数据抽象**。代码中是一个抽象类，它**代表一个不可变、可分区、里面的元素可并行计算的集合**。在Spark中，对数据的所有操作不外乎**创建RDD、转化已有RDD 以及调用RDD操作**进行求值。每个RDD都被分为多个分区，这些分区运行在集群中的不同节点上。RDD可以包含Python、Java、Scala中任意类型的对象，甚至可以包含用户自定义的对象。RDD具有数据流模型的**特点：自动容错、位置感知性调度和可伸缩性**。RDD允许用户在执行多个查询时显式地将工作集缓存在内存中，后续的查询能够重用工作集，这极大地提升了查询速度。  
&emsp; RDD支持两种操作：**转化操作和行动操作**。RDD的转化操作是返回一个新的RDD的操作，比如map()和filter()，而行动操作则是向驱动器程序返回结果或把结果写入外部系统的操作。比如count() 和first()。  
&emsp; Spark采用**惰性计算模式，RDD只有第一次在一个行动操作中用到时，才会真正计算**。Spark可以优化整个计算过程。**默认情况下，Spark的RDD会在你每次对它们进行行动操作时重新计算**。如果想在多个行动操作中**重用同一个RDD，可以使用RDD.persist()让Spark把这个RDD缓存下来**。  

### 3、RDD的属性
1）**一组分区（Partition），即数据集的基本组成单位。**  
&emsp; 对于RDD来说，每个分片都会被一个计算任务处理，并决定并行计算的粒度。用户可以在创建RDD时指定RDD的分片个数，如果没有指定，那么就会采用默认值。默认值就是程序所分配到的CPU Core的数目。  
2）**一个计算每个分区的函数。**  
&emsp; Spark中RDD的计算是以分片为单位的，每个RDD都会实现compute函数以达到这个目的。compute函数会对迭代器进行复合，不需要保存每次计算的结果。  
3）**RDD之间的依赖关系。**  
&emsp; RDD的每次转换都会生成一个新的RDD，所以RDD之间就会形成类似于流水线一样的前后依赖关系。在部分分区数据丢失时，Spark可以通过这个依赖关系重新计算丢失的分区数据，而不是对RDD的所有分区进行重新计算。  
4）**一个Partitioner，即RDD的分片函数。**  
&emsp; 当前Spark中实现了两种类型的分片函数，一个是基于哈希的HashPartitioner，另外一个是基于范围的RangePartitioner。只有对于于key-value的RDD，才会有Partitioner，非key-value的RDD的Parititioner的值是None。Partitioner函数不但决定了RDD本身的分片数量，也决定了parent RDD Shuffle输出时的分片数量。  
5）**一个列表，存储存取每个Partition的优先位置（preferred location）。**  
&emsp; 对于一个HDFS文件来说，这个列表保存的就是每个Partition所在的块的位置。按照“移动数据不如移动计算”的理念，Spark在进行任务调度的时候，会尽可能地将计算任务分配到其所要处理数据块的存储位置。  

### 4、Spark做了啥？
&emsp; RDD是一个应用层面的逻辑概念。一个RDD多个分片。**RDD就是一个元数据记录集，记录了RDD内存所有的关系数据**。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E6%A6%82%E8%BF%B0/Spark%E5%81%9A%E4%BA%86%E5%95%A5.png"/>  
<p align="center">
</p>
</p>  

### 5、RDD弹性
存储的弹性：内存与磁盘的自动切换  
容错的弹性：数据丢失可以自动恢复   
计算的弹性：计算出错重试机制   
分片的弹性：根据需要重新分片  
1）**自动进行内存和磁盘数据存储的切换**  
&emsp; Spark优先把数据放到内存中，如果内存放不下，就会放到磁盘里面，程序进行自动的存储切换  
2）**基于血统的高效容错机制**  
&emsp; 在RDD进行转换和动作的时候，会形成RDD的Lineage依赖链，当某一个RDD失效的时候，可以通过重新计算上游的RDD来重新生成丢失的RDD数据。  
3）**Task如果失败会自动进行特定次数的重试**  
&emsp; RDD的计算任务如果运行失败，会自动进行任务的重新计算，默认次数是4次。  
4）**Stage如果失败会自动进行特定次数的重试**  
&emsp; 如果Job的某个Stage阶段计算失败，框架也会自动进行任务的重新计算，默认次数也是4次。  
5）**Checkpoint和Persist可主动或被动触发**  
&emsp; RDD可以通过Persist持久化将RDD缓存到内存或者磁盘，当再次用到该RDD时直接读取就行。也可以将RDD进行检查点，检查点会将数据存储在HDFS中，该RDD的所有父RDD依赖都会被移除。  
6）**数据调度弹性**  
&emsp; Spark把这个JOB执行模型抽象为通用的有向无环图DAG，可以将多Stage的任务串联或并行执行，调度引擎自动处理Stage的失败以及Task的失败。  
7）**数据分片的高度弹性**  
&emsp; 可以根据业务的特征，动态调整数据分片的个数，提升整体的应用执行效率。  

### 6、RDD特点
&emsp; RDD表示只读的分区的数据集，对RDD进行改动，只能通过RDD的转换操作，由一个RDD得到一个新的RDD，新的RDD包含了从其他RDD衍生所必需的信息。RDDs之间存在依赖，RDD的执行是按照血缘关系延时计算的。如果血缘关系较长，可以通过持久化RDD来切断血缘关系。  
1）分区  
&emsp; RDD逻辑上是分区的，每个分区的数据是抽象存在的，计算的时候会通过一个compute函数得到每个分区的数据。如果RDD是通过已有的文件系统构建，则compute函数是读取指定文件系统中的数据，如果RDD是通过其他RDD转换而来，则compute函数是执行转换逻辑将其他RDD的数据进行转换。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E6%A6%82%E8%BF%B0/%E5%88%86%E5%8C%BA.png"/>  
<p align="center">
</p>
</p>  

2）只读  
&emsp; 如下图所示，RDD是只读的，要想改变RDD中的数据，只能在现有的RDD基础上创建新的RDD。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E6%A6%82%E8%BF%B0/%E5%8F%AA%E8%AF%BB.png"/>  
<p align="center">
</p>
</p>  

&emsp; 由一个RDD转换到另一个RDD，可以通过丰富的操作算子实现，不再像MapReduce那样只能写map和reduce了，如下图所示。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E6%A6%82%E8%BF%B0/%E4%B8%80%E4%B8%AARDD%E8%BD%AC%E6%8D%A2%E5%88%B0%E5%8F%A6%E4%B8%80%E4%B8%AARDD.png"/>  
<p align="center">
</p>
</p>  

&emsp; RDD的操作算子包括两类，一类叫做transformations，它是用来将RDD进行转化，构建RDD的血缘关系；另一类叫做actions，它是用来触发RDD的计算，得到RDD的相关计算结果或者将RDD保存的文件系统中。下图是RDD所支持的操作算子列表。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E6%A6%82%E8%BF%B0/RDD%E7%9A%84%E6%93%8D%E4%BD%9C%E7%AE%97%E5%AD%90.png"/>  
<p align="center">
</p>
</p>  

3）依赖  
&emsp; RDDs通过操作算子进行转换，转换得到的新RDD包含了从其他RDDs衍生所必需的信息，RDDs之间维护着这种血缘关系，也称之为依赖。如下图所示，依赖包括两种，一种是窄依赖，RDDs之间分区是一一对应的，另一种是宽依赖，下游RDD的每个分区与上游RDD(也称之为父RDD)的每个分区都有关，是多对多的关系。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E6%A6%82%E8%BF%B0/%E4%BE%9D%E8%B5%961.png"/>  
<p align="center">
</p>
</p>  

&emsp; 通过RDDs之间的这种依赖关系，一个任务流可以描述为DAG(有向无环图)，如下图所示，在实际执行过程中宽依赖对应于Shuffle(图中的reduceByKey和join)，窄依赖中的所有转换操作可以通过类似于管道的方式一气呵成执行(图中map和union可以一起执行)。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E6%A6%82%E8%BF%B0/%E4%BE%9D%E8%B5%962.png"/>  
<p align="center">
</p>
</p>  

4）缓存  
&emsp; 如果在应用程序中多次使用同一个RDD，可以将该RDD缓存起来，该RDD只有在第一次计算的时候会根据血缘关系得到分区的数据，在后续其他地方用到该RDD的时候，会直接从缓存处取而不用再根据血缘关系计算，这样就加速后期的重用。如下图所示，RDD-1经过一系列的转换后得到RDD-n并保存到hdfs，RDD-1在这一过程中会有个中间结果，如果将其缓存到内存，那么在随后的RDD-1转换到RDD-m这一过程中，就不会计算其之前的RDD-0了。  

5）checkpoint  
&emsp; 虽然RDD的血缘关系天然地可以实现容错，当RDD的某个分区数据失败或丢失，可以通过血缘关系重建。但是对于长时间迭代型应用来说，随着迭代的进行，RDDs之间的血缘关系会越来越长，一旦在后续迭代过程中出错，则需要通过非常长的血缘关系去重建，势必影响性能。为此，RDD支持checkpoint将数据保存到持久化的存储中，这样就可以切断之前的血缘关系，因为checkpoint后的RDD不需要知道它的父RDDs了，它可以从checkpoint处拿到数据。  
  
**注意**：  
&emsp; 给定一个RDD我们至少可以知道如下几点信息：  
&emsp; 1、分区数以及分区方式；  
&emsp; 2、由父RDDs衍生而来的相关依赖信息；  
&emsp; 3、计算每个分区的数据，计算步骤为：  
&emsp; &emsp; 1）如果被缓存，则从缓存中取的分区的数据；  
&emsp; &emsp; 2）如果被checkpoint，则从checkpoint处恢复数据；  
&emsp; &emsp; 3）根据血缘关系计算分区的数据。  










