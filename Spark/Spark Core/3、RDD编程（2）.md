RDD编程（二）
---  
### 1、RDD依赖关系  
#### 1.1 Lineage  
&emsp; RDD只支持粗粒度转换，即在大量记录上执行的单个操作。将创建RDD的一系列Lineage（血统）记录下来，以便恢复丢失的分区。RDD的Lineage会记录RDD的元数据信息和转换行为，当该RDD的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E7%BC%96%E7%A8%8B%EF%BC%881%EF%BC%89/%E5%9B%BE%E7%89%871.png"/>  
<p align="center">
</p>
</p>  
（1）读取一个HDFS文件并将其中内容映射成一个个元组   

```scala
scala> val wordAndOne = sc.textFile("/fruit.tsv").flatMap(_.split("\t")).map((_,1))
wordAndOne: org.apache.spark.rdd.RDD[(String, Int)] = MapPartitionsRDD[22] at map at <console>:24
```   
（2）统计每一种key对应的个数  
```scala
scala> val wordAndCount = wordAndOne.reduceByKey(_+_)
wordAndCount: org.apache.spark.rdd.RDD[(String, Int)] = ShuffledRDD[23] at reduceByKey at <console>:26
```  
（3）查看“wordAndOne”的Lineage  
```scala
scala> wordAndOne.toDebugString
res5: String =
(2) MapPartitionsRDD[22] at map at <console>:24 []
 |  MapPartitionsRDD[21] at flatMap at <console>:24 []
 |  /fruit.tsv MapPartitionsRDD[20] at textFile at <console>:24 []
 |  /fruit.tsv HadoopRDD[19] at textFile at <console>:24 []
```  
（4）查看“wordAndCount”的Lineage  
```scala
scala> wordAndCount.toDebugString
res6: String =
(2) ShuffledRDD[23] at reduceByKey at <console>:26 []
 +-(2) MapPartitionsRDD[22] at map at <console>:24 []
    |  MapPartitionsRDD[21] at flatMap at <console>:24 []
    |  /fruit.tsv MapPartitionsRDD[20] at textFile at <console>:24 []
    |  /fruit.tsv HadoopRDD[19] at textFile at <console>:24 []
```  
（5）查看“wordAndOne”的依赖类型  
```scala
scala> wordAndOne.dependencies
res7: Seq[org.apache.spark.Dependency[_]] = List(org.apache.spark.OneToOneDependency@5d5db92b)
```  
（6）查看“wordAndCount”的依赖类型  
```scala
scala> wordAndCount.dependencies
res8: Seq[org.apache.spark.Dependency[_]] = List(org.apache.spark.ShuffleDependency@63f3e6a8)
```  
**注意**：RDD和它依赖的父RDD（s）的关系有两种不同的类型，即窄依赖（narrow dependency）和宽依赖（wide dependency）。  
#### 1.2 窄依赖  
&emsp; 窄依赖指的是每一个父RDD的Partition最多被子RDD的一个Partition使用,**窄依赖我们形象的比喻为独生子女**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E7%BC%96%E7%A8%8B%EF%BC%881%EF%BC%89/%E5%9B%BE%E7%89%872.png"/>  
<p align="center">
</p>
</p>   

#### 1.3 宽依赖    

&emsp; 宽依赖指的是多个子RDD的Partition会依赖同一个父RDD的Partition，会引起shuffle,总结：**宽依赖我们形象的比喻为超生**    
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E7%BC%96%E7%A8%8B%EF%BC%881%EF%BC%89/%E5%9B%BE%E7%89%873.png"/>  
<p align="center">
</p>
</p>  

#### 1.4 DAG  
&emsp; DAG(Directed Acyclic Graph)叫做有向无环图，原始的RDD通过一系列的转换就就形成了DAG，根据RDD之间的依赖关系的不同将DAG划分成不同的Stage，对于窄依赖，partition的转换处理在Stage中完成计算。对于宽依赖，由于有Shuffle的存在，只能在parent RDD处理完成后，才能开始接下来的计算，因此**宽依赖是划分Stage的依据**。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E7%BC%96%E7%A8%8B%EF%BC%881%EF%BC%89/%E5%9B%BE%E7%89%874.png"/>  
<p align="center">
</p>
</p>  

#### 1.5 任务划分  
RDD任务切分中间分为：Application、Job、Stage和Task  
1）Application：初始化一个SparkContext即生成一个Application。  
2）Job：一个Action算子就会生成一个Job。  
3）Stage：根据RDD之间的依赖关系的不同将Job划分成不同的Stage，遇到一个宽依赖则划分一个Stage。  
4）Task：Stage是一个TaskSet，将Stage划分的结果发送到不同的Executor执行即为一个Task。  
**注意**：Application->Job->Stage-> Task每一层都是1对n的关系。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E7%BC%96%E7%A8%8B%EF%BC%881%EF%BC%89/%E5%9B%BE%E7%89%875.png"/>  
<p align="center">
</p>
</p>  

### 2、RDD缓存  
&emsp; RDD通过persist方法或cache方法可以将前面的计算结果缓存，默认情况下 persist() 会把数据以序列化的形式缓存在 JVM 的堆空间中。   
但是并不是这两个方法被调用时立即缓存，而是触发后面的action时，该RDD将会被缓存在计算节点的内存中，并供后面重用。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E7%BC%96%E7%A8%8B%EF%BC%881%EF%BC%89/%E5%9B%BE%E7%89%876.png"/>  
<p align="center">
</p>
</p>  
&emsp; 通过查看源码发现cache最终也是调用了persist方法，默认的存储级别都是仅在内存存储一份，Spark的存储级别还有好多种，存储级别在object StorageLevel中定义的。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E7%BC%96%E7%A8%8B%EF%BC%881%EF%BC%89/%E5%9B%BE%E7%89%877.png"/>  
<p align="center">
</p>
</p>  
&emsp; 在存储级别的末尾加上“_2”来把持久化数据存为两份  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Core/RDD%E7%BC%96%E7%A8%8B%EF%BC%881%EF%BC%89/%E5%9B%BE%E7%89%878.png"/>  
<p align="center">
</p>
</p>  
&emsp; 缓存有可能丢失，或者存储存储于内存的数据由于内存不足而被删除，RDD的缓存容错机制保证了即使缓存丢失也能保证计算的正确执行。通过基于RDD的一系列转换，丢失的数据会被重算，由于RDD的各个Partition是相对独立的，因此只需要计算丢失的部分即可，并不需要重算全部Partition。    

（1）创建一个RDD  

```scala
scala> val rdd = sc.makeRDD(Array("atguigu"))
rdd: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[19] at makeRDD at <console>:25
```  
（2）将RDD转换为携带当前时间戳不做缓存  
```scala
scala> val nocache = rdd.map(_.toString+System.currentTimeMillis)
nocache: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[20] at map at <console>:27
```  
（3）多次打印结果  
```scala
scala> nocache.collect
res0: Array[String] = Array(atguigu1538978275359)

scala> nocache.collect
res1: Array[String] = Array(atguigu1538978282416)

scala> nocache.collect
res2: Array[String] = Array(atguigu1538978283199)
```  
（4）将RDD转换为携带当前时间戳并做缓存  
```scala
scala> val cache =  rdd.map(_.toString+System.currentTimeMillis).cache
cache: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[21] at map at <console>:27
```  
（5）多次打印做了缓存的结果  
```scala
scala> cache.collect
res3: Array[String] = Array(atguigu1538978435705)                                   

scala> cache.collect
res4: Array[String] = Array(atguigu1538978435705)

scala> cache.collect
res5: Array[String] = Array(atguigu1538978435705)
```  
### 3、RDD CheckPoint  
&emsp; Spark中对于数据的保存除了持久化操作之外，还提供了一种检查点的机制，检查点（本质是通过将RDD写入Disk做检查点）是为了通过lineage做容错的辅助，lineage过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果之后有节点出现问题而丢失分区，从做检查点的RDD开始重做Lineage，就会减少开销。检查点通过将数据写入到HDFS文件系统实现了RDD的检查点功能。  
&emsp; 为当前RDD设置检查点。该函数将会创建一个二进制的文件，并存储到checkpoint目录中，该目录是用SparkContext.setCheckpointDir()设置的。在checkpoint的过程中，该RDD的所有依赖于父RDD中的信息将全部被移除。对RDD进行checkpoint操作并不会马上被执行，必须执行Action操作才能触发。  
案例实操：  
（1）设置检查点  
```scala
scala> sc.setCheckpointDir("hdfs://hadoop102:9000/checkpoint")
```  
（2）创建一个RDD  
```scala
scala> val rdd = sc.parallelize(Array("atguigu"))
rdd: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[14] at parallelize at <console>:24
```  
（3）将RDD转换为携带当前时间戳并做checkpoint  
```scala
scala> val ch = rdd.map(_+System.currentTimeMillis)
ch: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[16] at map at <console>:26

scala> ch.checkpoint
```  
（4）多次打印结果  
```scala
scala> ch.collect
res55: Array[String] = Array(atguigu1538981860336)

scala> ch.collect
res56: Array[String] = Array(atguigu1538981860504)

scala> ch.collect
res57: Array[String] = Array(atguigu1538981860504)

scala> ch.collect
res58: Array[String] = Array(atguigu1538981860504)
```  
