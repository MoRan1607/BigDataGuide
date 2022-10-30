RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是 Spark 中最基本的数据处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合。

**RDD特点**

RDD表示只读的分区的数据集，对RDD进行改动，只能通过RDD的转换操作，由一个RDD得到一个新的RDD，新的RDD包含了从其他RDD衍生所必需的信息。RDDs之间存在依赖，RDD的执行是按照血缘关系延时计算的。如果血缘关系较长，可以通过持久化RDD来切断血缘关系。

**1、分区**

RDD逻辑上是分区的，每个分区的数据是抽象存在的，计算的时候会通过一个compute函数得到每个分区的数据。如果RDD是通过已有的文件系统构建，则compute函数是读取指定文件系统中的数据，如果RDD是通过其他RDD转换而来，则compute函数是执行转换逻辑将其他RDD的数据进行转换。

<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/Spark%E9%9D%A2%E8%AF%95%E9%A2%98%E6%95%B4%E7%90%86/Spark/Pics/%E8%AF%B4%E4%B8%8B%E5%AF%B9RDD%E7%9A%84%E7%90%86%E8%A7%A3%EF%BC%9FRDD%E7%89%B9%E7%82%B9%E3%80%81%E7%AE%97%E5%AD%90/%E8%AF%B4%E4%B8%8B%E5%AF%B9RDD%E7%9A%84%E7%90%86%E8%A7%A3%EF%BC%9FRDD%E7%89%B9%E7%82%B9%E3%80%81%E7%AE%97%E5%AD%9001.png" />  
<p align="center">
</p>
</p>  

**2、只读**

如下图所示，RDD是只读的，要想改变RDD中的数据，只能在现有的RDD基础上创建新的RDD。

<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/Spark%E9%9D%A2%E8%AF%95%E9%A2%98%E6%95%B4%E7%90%86/Spark/Pics/%E8%AF%B4%E4%B8%8B%E5%AF%B9RDD%E7%9A%84%E7%90%86%E8%A7%A3%EF%BC%9FRDD%E7%89%B9%E7%82%B9%E3%80%81%E7%AE%97%E5%AD%90/%E8%AF%B4%E4%B8%8B%E5%AF%B9RDD%E7%9A%84%E7%90%86%E8%A7%A3%EF%BC%9FRDD%E7%89%B9%E7%82%B9%E3%80%81%E7%AE%97%E5%AD%902.png" />  
<p align="center">
</p>
</p>  

由一个RDD转换到另一个RDD，可以通过丰富的操作算子实现，不再像MapReduce那样只能写map和reduce了，如下图所示。
<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/Spark%E9%9D%A2%E8%AF%95%E9%A2%98%E6%95%B4%E7%90%86/Spark/Pics/%E8%AF%B4%E4%B8%8B%E5%AF%B9RDD%E7%9A%84%E7%90%86%E8%A7%A3%EF%BC%9FRDD%E7%89%B9%E7%82%B9%E3%80%81%E7%AE%97%E5%AD%90/%E8%AF%B4%E4%B8%8B%E5%AF%B9RDD%E7%9A%84%E7%90%86%E8%A7%A3%EF%BC%9FRDD%E7%89%B9%E7%82%B9%E3%80%81%E7%AE%97%E5%AD%903.png" />  
<p align="center">
</p>
</p>  

RDD的操作算子包括两类，一类叫做transformations，它是用来将RDD进行转化，构建RDD的血缘关系；另一类叫做actions，它是用来触发RDD的计算，得到RDD的相关计算结果或者将RDD保存的文件系统中。下图是RDD所支持的操作算子列表。

**3、依赖**

RDDs通过操作算子进行转换，转换得到的新RDD包含了从其他RDDs衍生所必需的信息，RDDs之间维护着这种血缘关系，也称之为依赖。如下图所示，依赖包括两种，一种是窄依赖，RDDs之间分区是一一对应的，另一种是宽依赖，下游RDD的每个分区与上游RDD(也称之为父RDD)的每个分区都有关，是多对多的关系。
<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/Spark%E9%9D%A2%E8%AF%95%E9%A2%98%E6%95%B4%E7%90%86/Spark/Pics/%E8%AF%B4%E4%B8%8B%E5%AF%B9RDD%E7%9A%84%E7%90%86%E8%A7%A3%EF%BC%9FRDD%E7%89%B9%E7%82%B9%E3%80%81%E7%AE%97%E5%AD%90/%E8%AF%B4%E4%B8%8B%E5%AF%B9RDD%E7%9A%84%E7%90%86%E8%A7%A3%EF%BC%9FRDD%E7%89%B9%E7%82%B9%E3%80%81%E7%AE%97%E5%AD%904.png" />  
<p align="center">
</p>
</p>  


**4、缓存**

如果在应用程序中多次使用同一个RDD，可以将该RDD缓存起来，该RDD只有在第一次计算的时候会根据血缘关系得到分区的数据，在后续其他地方用到该RDD的时候，会直接从缓存处取而不用再根据血缘关系计算，这样就加速后期的重用。如下图所示，RDD-1经过一系列的转换后得到RDD-n并保存到hdfs，RDD-1在这一过程中会有个中间结果，如果将其缓存到内存，那么在随后的RDD-1转换到RDD-m这一过程中，就不会计算其之前的RDD-0了。

<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/Spark%E9%9D%A2%E8%AF%95%E9%A2%98%E6%95%B4%E7%90%86/Spark/Pics/%E8%AF%B4%E4%B8%8B%E5%AF%B9RDD%E7%9A%84%E7%90%86%E8%A7%A3%EF%BC%9FRDD%E7%89%B9%E7%82%B9%E3%80%81%E7%AE%97%E5%AD%90/%E8%AF%B4%E4%B8%8B%E5%AF%B9RDD%E7%9A%84%E7%90%86%E8%A7%A3%EF%BC%9FRDD%E7%89%B9%E7%82%B9%E3%80%81%E7%AE%97%E5%AD%905.png" />  
<p align="center">
</p>
</p>  

**5、CheckPoint**

虽然RDD的血缘关系天然地可以实现容错，当RDD的某个分区数据失败或丢失，可以通过血缘关系重建。但是对于长时间迭代型应用来说，随着迭代的进行，RDDs之间的血缘关系会越来越长，一旦在后续迭代过程中出错，则需要通过非常长的血缘关系去重建，势必影响性能。为此，RDD支持checkpoint将数据保存到持久化的存储中，这样就可以切断之前的血缘关系，因为checkpoint后的RDD不需要知道它的父RDDs了，它可以从checkpoint处拿到数据。

RDD算子   
 
RDD整体上分为Value类型和Key-Value类型   

比如map、flatmap等这些，回答几个，讲一下原理就差不多了  
 
map：遍历RDD,将函数f应用于每一个元素，返回新的RDD(transformation算子)。  

foreach：遍历RDD,将函数f应用于每一个元素，无返回值(action算子)。  

mapPartitions：遍历操作RDD中的每一个分区，返回新的RDD（transformation算子）。  

foreachPartition：遍历操作RDD中的每一个分区。无返回值(action算子)。  

