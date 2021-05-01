RDD编程（一）
---  
### 1、编程模型  
&emsp; 在Spark中，RDD被表示为对象，通过对象上的方法调用来对RDD进行转换。经过一系列的transformations定义RDD之后，就可以调用actions触发RDD的计算，action可以是向应用程序返回结果(count, collect等)，或者是向存储系统保存数据(saveAsTextFile等)。在Spark中，只有遇到action，才会执行RDD的计算(即延迟计算)，这样在运行时可以通过管道的方式传输多个转换。  
&emsp; 要使用Spark，开发者需要编写一个Driver程序，它被提交到集群以调度运行Worker，如下图所示。Driver中定义了一个或多个RDD，并调用RDD上的action，Worker则执行RDD分区计算任务。  

### 2、RDD的创建  
&emsp; 在Spark中创建RDD的创建方式可以分为三种：**从集合中创建RDD；从外部存储创建RDD；从其他RDD创建**。  
#### 2.1 从集合中创建  
&emsp; 从集合中创建RDD，Spark主要提供了两种函数：parallelize和makeRDD  
&emsp; 1）使用parallelize()从集合创建  
```scala
scala> val rdd = sc.parallelize(Array(1,2,3,4,5,6,7,8))
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:24
```  
&emsp; 2）使用makeRDD()从集合创建  
```scala
scala> val rdd = sc.parallelize(Array(1,2,3,4,5,6,7,8))
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:24
```  
```scala
scala> val rdd1 = sc.makeRDD(Array(1,2,3,4,5,6,7,8))
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[1] at makeRDD at <console>:24
```  
#### 2.2 由外部存储系统的数据集创建  
&emsp; 包括本地的文件系统，还有所有Hadoop支持的数据集，比如HDFS、Cassandra、HBase等。  
```scala
scala> val rdd2= sc.textFile("hdfs://hadoop102:9000/RELEASE")
rdd2: org.apache.spark.rdd.RDD[String] = hdfs:// hadoop102:9000/RELEASE MapPartitionsRDD[4] at textFile at <console>:24
```  
### 3、RDD的转换  
RDD整体上分为Value类型和Key-Value类型  
#### 3.1 Value类型  
**3.1.1 map(func)案例**   
1）作用：返回一个新的RDD，该RDD由每一个输入元素经过func函数转换后组成  
2）需求：创建一个1-10数组的RDD，将所有元素x2形成新的RDD  
（1）创建   
```scala  
scala> var source  = sc.parallelize(1 to 10)
source: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[8] at parallelize at <console>:24
```   
（2）打印   
```scala
scala> source.collect()
res7: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
```  
（3）将所有元素x2  
```scala
scala> val mapadd = source.map(_ * 2)
mapadd: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[9] at map at <console>:26
```  
（4）打印最终结果  
```scala
scala> mapadd.collect()
res8: Array[Int] = Array(2, 4, 6, 8, 10, 12, 14, 16, 18, 20)
```  
**3.1.2 mapPartitions(func) 案例**   
1）作用：类似于map，但独立地在RDD的每一个分片上运行，因此在类型为T的RDD上运行时，func的函数类型必须是Iterator[T] => Iterator[U]。假设有N个元素，有M个分区，那么map的函数的将被调用N次,而mapPartitions被调用M次,一个函数一次处理所有分区。  
2）需求：创建一个RDD，使每个元素x2组成新的RDD  
（1）创建一个RDD  
```scala
scala> val rdd = sc.parallelize(Array(1,2,3,4))
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[4] at parallelize at <console>:24
```  
（2）使每个元素x2组成新的RDD  
```scala
scala> rdd.mapPartitions(x=>x.map(_*2))
res3: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[6] at mapPartitions at <console>:27
```  
（3）打印新的RDD  
```scala
scala> res3.collect
res4: Array[Int] = Array(2, 4, 6, 8)
```  
**3.1.3 mapPartitionsWithIndex(func) 案例**  
1）作用：类似于mapPartitions，但func带有一个整数参数表示分片的索引值，因此在类型为T的RDD上运行时，func的函数类型必须是(Int, Interator[T]) => Iterator[U]；   
2）需求：创建一个RDD，使每个元素跟所在分区形成一个元组组成一个新的RDD    
（1）创建一个RDD  
```scala
scala> val rdd = sc.parallelize(Array(1,2,3,4))
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[4] at parallelize at <console>:24
```  
（2）使每个元素跟所在分区形成一个元组组成一个新的RDD  
```scala
scala> val indexRdd = rdd.mapPartitionsWithIndex((index,items)=>(items.map((index,_))))
indexRdd: org.apache.spark.rdd.RDD[(Int, Int)] = MapPartitionsRDD[5] at mapPartitionsWithIndex at <console>:26
```  
（3）打印新的RDD  
```scala
scala> indexRdd.collect
res2: Array[(Int, Int)] = Array((0,1), (0,2), (1,3), (1,4))
```   
**3.1.4 flatMap(func) 案例**  
1）作用：类似于map，但是每一个输入元素可以被映射为0或多个输出元素（所以func应该返回一个序列，而不是单一元素）  
2）需求：创建一个元素为1-5的RDD，运用flatMap创建一个新的RDD，新的RDD为原RDD的每个元素的2倍（2，4，6，8，10）  
（1）创建  
```scala
scala> val sourceFlat = sc.parallelize(1 to 5)
sourceFlat: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[12] at parallelize at <console>:24
```   
（2）打印  
```scala
scala> sourceFlat.collect()
res11: Array[Int] = Array(1, 2, 3, 4, 5)
```   
（3）根据原RDD创建新RDD（1->1,2->1,2……5->1,2,3,4,5）  
```scala
scala> val flatMap = sourceFlat.flatMap(1 to _)
flatMap: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[13] at flatMap at <console>:26
```   
（4）打印新RDD  
```scala
scala> flatMap.collect()
res12: Array[Int] = Array(1, 1, 2, 1, 2, 3, 1, 2, 3, 4, 1, 2, 3, 4, 5)
```   
**3.1.5 map()和mapPartition()的区别**  
1）map()：每次处理一条数据。  
2）mapPartition()：每次处理一个分区的数据，这个分区的数据处理完后，原RDD中分区的数据才能释放，可能导致OOM。  
3）开发指导：当内存空间较大的时候建议使用mapPartition()，以提高处理效率。  
**3.1.6 glom案例**  
1）作用：将每一个分区形成一个数组，形成新的RDD类型时RDD[Array[T]]  
2）需求：创建一个4个分区的RDD，并将每个分区的数据放到一个数组  
（1）创建  
```scala
scala> val rdd = sc.parallelize(1 to 16,4)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[65] at parallelize at <console>:24
```   
（2）将每个分区的数据放到一个数组并收集到Driver端打印  
```scala
scala> rdd.glom().collect()
res25: Array[Array[Int]] = Array(Array(1, 2, 3, 4), Array(5, 6, 7, 8), Array(9, 10, 11, 12), Array(13, 14, 15, 16))
```   
**3.1.7 groupBy(func)案例**  
1）作用：分组，按照传入函数的返回值进行分组。将相同的key对应的值放入一个迭代器。  
2）需求：创建一个RDD，按照元素模以2的值进行分组。  
（1）创建  
```scala
scala> val rdd = sc.parallelize(1 to 4)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[65] at parallelize at <console>:24
```  
（2）按照元素模以2的值进行分组  
```scala
scala> val group = rdd.groupBy(_%2)
group: org.apache.spark.rdd.RDD[(Int, Iterable[Int])] = ShuffledRDD[2] at groupBy at <console>:26
```
（3）打印结果  
```scala
scala> group.collect
res0: Array[(Int, Iterable[Int])] = Array((0,CompactBuffer(2, 4)), (1,CompactBuffer(1, 3)))
```
**3.1.8 filter(func) 案例**  
1）作用：过滤。返回一个新的RDD，该RDD由经过func函数计算后返回值为true的输入元素组成。  
2）需求：创建一个RDD（由字符串组成），过滤出一个新RDD（包含”xiao”子串）  
（1）创建  
```scala
scala> var sourceFilter = sc.parallelize(Array("xiaoming","xiaojiang","xiaohe","dazhi"))
sourceFilter: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[10] at parallelize at <console>:24
```
（2）打印  
```scala
scala> sourceFilter.collect()
res9: Array[String] = Array(xiaoming, xiaojiang, xiaohe, dazhi)
```  
（3）过滤出含” xiao”子串的形成一个新的RDD  
```scala
scala> val filter = sourceFilter.filter(_.contains("xiao"))
filter: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[11] at filter at <console>:26
```  
（4）打印新RDD  
```scala
scala> filter.collect()
res10: Array[String] = Array(xiaoming, xiaojiang, xiaohe)
```  
**3.1.9 sample(withReplacement, fraction, seed) 案例**  
1）作用：以指定的随机种子随机抽样出数量为fraction的数据，withReplacement表示是抽出的数据是否放回，true为有放回的抽样，false为无放回的抽样，seed用于指定随机数生成器种子。  
2）需求：创建一个RDD（1-10），从中选择放回和不放回抽样  
（1）创建RDD  
```scala
scala> val rdd = sc.parallelize(1 to 10)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[20] at parallelize at <console>:24
```
（2）打印   
```scala
scala> rdd.collect()
res15: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
```  
（3）放回抽样  
```scala
scala> var sample1 = rdd.sample(true,0.4,2)
sample1: org.apache.spark.rdd.RDD[Int] = PartitionwiseSampledRDD[21] at sample at <console>:26
```
（4）打印放回抽样结果  
```scala
scala> sample1.collect()
res16: Array[Int] = Array(1, 2, 2, 7, 7, 8, 9)
```  
（5）不放回抽样  
```scala
scala> var sample2 = rdd.sample(false,0.2,3)
sample2: org.apache.spark.rdd.RDD[Int] = PartitionwiseSampledRDD[22] at sample at <console>:26
```  
（6）打印不放回抽样结果  
```scala
scala> sample2.collect()
res17: Array[Int] = Array(1, 9)
```  
**3.1.10 distinct([numTasks])) 案例**  
1）作用：对源RDD进行去重后返回一个新的RDD。默认情况下，只有8个并行任务来操作，但是可以传入一个可选的numTasks参数改变它。  
2）需求：创建一个RDD，使用distinct()对其去重。  
（1）创建一个RDD  
```scala
scala> val distinctRdd = sc.parallelize(List(1,2,1,5,2,9,6,1))
distinctRdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[34] at parallelize at <console>:24
```  
（2）对RDD进行去重（不指定并行度）  
```scala
scala> val unionRDD = distinctRdd.distinct()
unionRDD: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[37] at distinct at <console>:26
```  
（3）打印去重后生成的新RDD  
```scala
scala> unionRDD.collect()
res20: Array[Int] = Array(1, 9, 5, 6, 2)
```  
（4）对RDD（指定并行度为2）  
```scala
scala> val unionRDD = distinctRdd.distinct(2)
unionRDD: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[40] at distinct at <console>:26
```  
（5）打印去重后生成的新RDD  
```scala
scala> unionRDD.collect()
res21: Array[Int] = Array(6, 2, 1, 9, 5)
```  
**3.1.11 coalesce(numPartitions) 案例**  
1）作用：缩减分区数，用于大数据集过滤后，提高小数据集的执行效率。  
2）需求：创建一个4个分区的RDD，对其缩减分区  
（1）创建一个RDD  
```scala
scala> val rdd = sc.parallelize(1 to 16,4)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[54] at parallelize at <console>:24
```  
（2）查看RDD的分区数  
```scala
scala> rdd.partitions.size
res20: Int = 4
```  
（3）对RDD重新分区  
```scala
scala> val coalesceRDD = rdd.coalesce(3)
coalesceRDD: org.apache.spark.rdd.RDD[Int] = CoalescedRDD[55] at coalesce at <console>:26
```  
（4）查看新RDD的分区数  
```scala
scala> coalesceRDD.partitions.size
res21: Int = 3
```  
**3.1.12 repartition(numPartitions) 案例**  
1）作用：根据分区数，重新通过网络随机洗牌所有数据。  
2）需求：创建一个4个分区的RDD，对其重新分区  
（1）创建一个RDD  
```scala
scala> val rdd = sc.parallelize(1 to 16,4)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[56] at parallelize at <console>:24
```  
（2）查看RDD的分区数  
```scala
scala> rdd.partitions.size
res22: Int = 4
```  
（3）对RDD重新分区  
```scala
scala> val rerdd = rdd.repartition(2)
rerdd: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[60] at repartition at <console>:26
```  
（4）查看新RDD的分区数  
```scala
scala> rerdd.partitions.size
res23: Int = 2
```  
**3.1.13 coalesce和repartition的区别**  
1）coalesce重新分区，可以选择是否进行shuffle过程。由参数shuffle: Boolean = false/true决定。  
2）repartition实际上是调用的coalesce，默认是进行shuffle的。源码如下：  
```scala
def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T] = withScope {
  coalesce(numPartitions, shuffle = true)
}
```  
**3.1.14 sortBy(func,[ascending], [numTasks]) 案例**  
1）作用；使用func先对数据进行处理，按照处理后的数据比较结果排序，默认为正序。  
2）需求：创建一个RDD，按照不同的规则进行排序  
（1）创建一个RDD  
```scala
scala> val rdd = sc.parallelize(List(2,1,3,4))
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[21] at parallelize at <console>:24
```  
（2）按照自身大小排序  
```scala
scala> rdd.sortBy(x => x).collect()
res11: Array[Int] = Array(1, 2, 3, 4)
```  
（3）按照与3余数的大小排序  
```scala
scala> rdd.sortBy(x => x%3).collect()
res12: Array[Int] = Array(3, 4, 1, 2)
```  
**3.1.15 pipe(command, [envVars]) 案例**  
1）作用：管道，针对每个分区，都执行一个shell脚本，返回输出的RDD。  
注意：脚本需要放在Worker节点可以访问到的位置  
2）需求：编写一个脚本，使用管道将脚本作用于RDD上。  
（1）编写一个脚本  
Shell脚本  
```shell
#!/bin/sh
echo "AA"
while read LINE; do
   echo ">>>"${LINE}
done
```    
（2）创建一个只有一个分区的RDD  
```scala
scala> val rdd = sc.parallelize(List("hi","Hello","how","are","you"),1)
rdd: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[50] at parallelize at <console>:24
```  
（3）将脚本作用该RDD并打印  
```scala
scala> rdd.pipe("/opt/module/spark/pipe.sh").collect()
res18: Array[String] = Array(AA, >>>hi, >>>Hello, >>>how, >>>are, >>>you)
```  
（4）创建一个有两个分区的RDD    
```scala
scala> val rdd = sc.parallelize(List("hi","Hello","how","are","you"),2)
rdd: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[52] at parallelize at <console>:24
```    
（5）将脚本作用该RDD并打印    
```scala
scala> rdd.pipe("/opt/module/spark/pipe.sh").collect()
res19: Array[String] = Array(AA, >>>hi, >>>Hello, AA, >>>how, >>>are, >>>you)
```  
#### 3.2 双Value类型交互  
**3.2.1 union(otherDataset) 案例**  
1）作用：对源RDD和参数RDD求并集后返回一个新的RDD  
2）需求：创建两个RDD，求并集  
（1）创建第一个RDD  
```scala
scala> val rdd1 = sc.parallelize(1 to 5)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[23] at parallelize at <console>:24
```  
（2）创建第二个RDD  
```scala
scala> val rdd2 = sc.parallelize(5 to 10)
rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[24] at parallelize at <console>:24
```  
（3）计算两个RDD的并集  
```scala
scala> val rdd3 = rdd1.union(rdd2)
rdd3: org.apache.spark.rdd.RDD[Int] = UnionRDD[25] at union at <console>:28
```  
（4）打印并集结果  
```scala
scala> rdd3.collect()
res18: Array[Int] = Array(1, 2, 3, 4, 5, 5, 6, 7, 8, 9, 10)
```  
**3.2.2 subtract (otherDataset) 案例**  
1）作用：计算差的一种函数，去除两个RDD中相同的元素，不同的RDD将保留下来  
2）需求：创建两个RDD，求第一个RDD与第二个RDD的差集  
（1）创建第一个RDD  
```scala
scala> val rdd = sc.parallelize(3 to 8)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[70] at parallelize at <console>:24
```  
（2）创建第二个RDD  
```scala
scala> val rdd1 = sc.parallelize(1 to 5)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[71] at parallelize at <console>:24
```  
（3）计算第一个RDD与第二个RDD的差集并打印  
```scala
scala> rdd.subtract(rdd1).collect()
res27: Array[Int] = Array(8, 6, 7)
```  
**3.2.3 intersection(otherDataset) 案例**  
1）作用：对源RDD和参数RDD求交集后返回一个新的RDD  
2）需求：创建两个RDD，求两个RDD的交集  
（1）创建第一个RDD  
```scala
scala> val rdd1 = sc.parallelize(1 to 7)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[26] at parallelize at <console>:24
```  
（2）创建第二个RDD  
```scala
scala> val rdd2 = sc.parallelize(5 to 10)
rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[27] at parallelize at <console>:24
```  
（3）计算两个RDD的交集  
```scala
scala> val rdd3 = rdd1.intersection(rdd2)
rdd3: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[33] at intersection at <console>:28
```  
（4）打印计算结果  
```scala
scala> rdd3.collect()
res19: Array[Int] = Array(5, 6, 7)
```  
**3.2.4 cartesian(otherDataset) 案例**  
1）作用：笛卡尔积（尽量避免使用）  
2）需求：创建两个RDD，计算两个RDD的笛卡尔积  
（1）创建第一个RDD  
```scala
scala> val rdd1 = sc.parallelize(1 to 3)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[47] at parallelize at <console>:24
```  
（2）创建第二个RDD  
```scala
scala> val rdd2 = sc.parallelize(2 to 5)
rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[48] at parallelize at <console>:24
```  
（3）计算两个RDD的笛卡尔积并打印  
```scala
scala> rdd1.cartesian(rdd2).collect()
res17: Array[(Int, Int)] = Array((1,2), (1,3), (1,4), (1,5), (2,2), (2,3), (2,4), (2,5), (3,2), (3,3), (3,4), (3,5))
```  
**3.2.5 zip(otherDataset)案例**  
1）作用：将两个RDD组合成Key/Value形式的RDD,这里默认两个RDD的partition数量以及元素数量都相同，否则会抛出异常。  
2）需求：创建两个RDD，并将两个RDD组合到一起形成一个(k,v)RDD  
（1）创建第一个RDD  
```scala
scala> val rdd1 = sc.parallelize(Array(1,2,3),3)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[1] at parallelize at <console>:24
```  
（2）创建第二个RDD（与1分区数相同）  
```scala
scala> val rdd2 = sc.parallelize(Array("a","b","c"),3)
rdd2: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[2] at parallelize at <console>:24 
```  
（3）第一个RDD组合第二个RDD并打印  
```scala
scala> rdd1.zip(rdd2).collect
res1: Array[(Int, String)] = Array((1,a), (2,b), (3,c))
```  
（4）第二个RDD组合第一个RDD并打印  
```scala
scala> rdd2.zip(rdd1).collect
res2: Array[(String, Int)] = Array((a,1), (b,2), (c,3))
```  
（5）创建第三个RDD（与1,2分区数不同）  
```scala
scala> val rdd3 = sc.parallelize(Array("a","b","c"),2)
rdd3: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[5] at parallelize at <console>:24
```  
（6）第一个RDD组合第三个RDD并打印  
```scala
scala> rdd1.zip(rdd3).collect
java.lang.IllegalArgumentException: Can't zip RDDs with unequal numbers of partitions: List(3, 2)
  at org.apache.spark.rdd.ZippedPartitionsBaseRDD.getPartitions(ZippedPartitionsRDD.scala:57)
  at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:252)
  at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:250)
  at scala.Option.getOrElse(Option.scala:121)
  at org.apache.spark.rdd.RDD.partitions(RDD.scala:250)
  at org.apache.spark.SparkContext.runJob(SparkContext.scala:1965)
  at org.apache.spark.rdd.RDD$$anonfun$collect$1.apply(RDD.scala:936)
  at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
  at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:112)
  at org.apache.spark.rdd.RDD.withScope(RDD.scala:362)
  at org.apache.spark.rdd.RDD.collect(RDD.scala:935)
  ... 48 elided
```  
#### 3.3 Key-Value类型   
**3.3.1 partitionBy案例**  
1）作用：对pairRDD进行分区操作，如果原有的partionRDD和现有的partionRDD是一致的话就不进行分区， 否则会生成ShuffleRDD，即会产生shuffle过程。  
2）需求：创建一个4个分区的RDD，对其重新分区  
（1）创建一个RDD  
```scala
scala> val rdd = sc.parallelize(Array((1,"aaa"),(2,"bbb"),(3,"ccc"),(4,"ddd")),4)
rdd: org.apache.spark.rdd.RDD[(Int, String)] = ParallelCollectionRDD[44] at parallelize at <console>:24
```  
（2）查看RDD的分区数  
```scala
scala> rdd.partitions.size
res24: Int = 4
```  
（3）对RDD重新分区  
```scala
scala> var rdd2 = rdd.partitionBy(new org.apache.spark.HashPartitioner(2))
rdd2: org.apache.spark.rdd.RDD[(Int, String)] = ShuffledRDD[45] at partitionBy at <console>:26
```  
（4）查看新RDD的分区数  
```scala
scala> rdd2.partitions.size
res25: Int = 2
```  
**3.3.2 groupByKey案例**  
1）作用：groupByKey也是对每个key进行操作，但只生成一个sequence。  
2）需求：创建一个pairRDD，将相同key对应值聚合到一个sequence中，并计算相同key对应值的相加结果。  
（1）创建一个pairRDD    
```scala
scala> val words = Array("one", "two", "two", "three", "three", "three")
words: Array[String] = Array(one, two, two, three, three, three)

scala> val wordPairsRDD = sc.parallelize(words).map(word => (word, 1))
wordPairsRDD: org.apache.spark.rdd.RDD[(String, Int)] = MapPartitionsRDD[4] at map at <console>:26
```  
（2）将相同key对应值聚合到一个sequence中    
```scala
scala> val group = wordPairsRDD.groupByKey()
group: org.apache.spark.rdd.RDD[(String, Iterable[Int])] = ShuffledRDD[5] at groupByKey at <console>:28
```  
（3）打印结果    
```scala
scala> group.collect()
res1: Array[(String, Iterable[Int])] = Array((two,CompactBuffer(1, 1)), (one,CompactBuffer(1)), (three,CompactBuffer(1, 1, 1)))
```  
（4）计算相同key对应值的相加结果    
```scala
scala> group.map(t => (t._1, t._2.sum))
res2: org.apache.spark.rdd.RDD[(String, Int)] = MapPartitionsRDD[6] at map at <console>:31
```   
（5）打印结果    
```scala
scala> res2.collect()
res3: Array[(String, Int)] = Array((two,2), (one,1), (three,3))
```  
**3.3.3 reduceByKey(func, [numTasks]) 案例**    
1）在一个(K,V)的RDD上调用，返回一个(K,V)的RDD，使用指定的reduce函数，将相同key的值聚合到一起，reduce任务的个数可以通过第二个可选的参数来设置。  
2）需求：创建一个pairRDD，计算相同key对应值的相加结果  
（1）创建一个pairRDD  
```scala
scala> val rdd = sc.parallelize(List(("female",1),("male",5),("female",5),("male",2)))
rdd: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[46] at parallelize at <console>:24
```  
（2）计算相同key对应值的相加结果  
```scala
scala> val reduce = rdd.reduceByKey((x,y) => x+y)
reduce: org.apache.spark.rdd.RDD[(String, Int)] = ShuffledRDD[47] at reduceByKey at <console>:26
```  
（3）打印结果  
```scala
scala> reduce.collect()
res29: Array[(String, Int)] = Array((female,6), (male,7))
```    

**3.3.4 reduceByKey和groupByKey的区别**  
1）reduceByKey：按照key进行聚合，在shuffle之前有combine（预聚合）操作，返回结果是RDD[k,v]。  
2）groupByKey：按照key进行分组，直接进行shuffle。  
3）开发指导：reduceByKey比groupByKey，建议使用。但是需要注意是否会影响业务逻辑。  
**3.3.5 aggregateByKey案例**  
参数：(zeroValue:U,[partitioner: Partitioner]) (seqOp: (U, V) => U,combOp: (U, U) => U)  
1）作用：在kv对的RDD中，，按key将value进行分组合并，合并时，将每个value和初始值作为seq函数的参数，进行计算，返回的结果作为一个新的kv对，然后再将结果按照key进行合并，最后将每个分组的value传递给combine函数进行计算（先将前两个value进行计算，将返回结果和下一个value传给combine函数，以此类推），将key与计算结果作为一个新的kv对输出。  
2）参数描述：  
（1）zeroValue：给每一个分区中的每一个key一个初始值；  
（2）seqOp：函数用于在每一个分区中用初始值逐步迭代value；  
（3）combOp：函数用于合并每个分区中的结果。  
3）需求：创建一个pairRDD，取出每个分区相同key对应值的最大值，然后相加  
（1）创建一个pairRDD  
```scala
scala> val rdd = sc.parallelize(List(("a",3),("a",2),("c",4),("b",3),("c",6),("c",8)),2)
rdd: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[0] at parallelize at <console>:24
```    
（2）取出每个分区相同key对应值的最大值，然后相加  
```scala
scala> val agg = rdd.aggregateByKey(0)(math.max(_,_),_+_)
agg: org.apache.spark.rdd.RDD[(String, Int)] = ShuffledRDD[1] at aggregateByKey at <console>:26
```    
（3）打印结果  
```scala
scala> agg.collect()
res0: Array[(String, Int)] = Array((b,3), (a,3), (c,12))
```    
**3.3.6 foldByKey案例**  
参数：(zeroValue: V)(func: (V, V) => V): RDD[(K, V)]  
1.作用：aggregateByKey的简化操作，seqop和combop相同  
2.需求：创建一个pairRDD，计算相同key对应值的相加结果  
（1）创建一个pairRDD  
```scala
scala> val rdd = sc.parallelize(List((1,3),(1,2),(1,4),(2,3),(3,6),(3,8)),3)
rdd: org.apache.spark.rdd.RDD[(Int, Int)] = ParallelCollectionRDD[91] at parallelize at <console>:24
```    
（2）计算相同key对应值的相加结果  
```scala
scala> val agg = rdd.foldByKey(0)(_+_)
agg: org.apache.spark.rdd.RDD[(Int, Int)] = ShuffledRDD[92] at foldByKey at <console>:26
```    
（3）打印结果  
```scala
scala> agg.collect()
res61: Array[(Int, Int)] = Array((3,14), (1,9), (2,3))
```    
**3.3.7 combineByKey[C] 案例**  
参数：(createCombiner: V => C,  mergeValue: (C, V) => C,  mergeCombiners: (C, C) => C)   
1）作用：对相同K，把V合并成一个集合。  
2）参数描述：  
（1）createCombiner: combineByKey() 会遍历分区中的所有元素，因此每个元素的键要么还没有遇到过，要么就和之前的某个元素的键相同。如果这是一个新的元素,combineByKey()会使用一个叫作createCombiner()的函数来创建那个键对应的累加器的初始值  
（2）mergeValue: 如果这是一个在处理当前分区之前已经遇到的键，它会使用mergeValue()方法将该键的累加器对应的当前值与这个新的值进行合并  
（3）mergeCombiners: 由于每个分区都是独立处理的，因此对于同一个键可以有多个累加器。如果有两个或者更多的分区都有对应同一个键的累加器，就需要使用用户提供的 mergeCombiners() 方法将各个分区的结果进行合并。  
3）需求：创建一个pairRDD，根据key计算每种key的均值。（先计算每个key出现的次数以及可以对应值的总和，再相除得到结果）  
（1）创建一个pairRDD  
```scala
scala> val input = sc.parallelize(Array(("a", 88), ("b", 95), ("a", 91), ("b", 93), ("a", 95), ("b", 98)),2)
input: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[52] at parallelize at <console>:26
```    
（2）将相同key对应的值相加，同时记录该key出现的次数，放入一个二元组  
```scala
scala> val combine = input.combineByKey((_,1),(acc:(Int,Int),v)=>(acc._1+v,acc._2+1),(acc1:(Int,Int),acc2:(Int,Int))=>(acc1._1+acc2._1,acc1._2+acc2._2))
combine: org.apache.spark.rdd.RDD[(String, (Int, Int))] = ShuffledRDD[5] at combineByKey at <console>:28
```    
（3）打印合并后的结果  
```scala
scala> combine.collect
res5: Array[(String, (Int, Int))] = Array((b,(286,3)), (a,(274,3)))
```    
（4）计算平均值  
```scala
scala> val result = combine.map{case (key,value) => (key,value._1/value._2.toDouble)}
result: org.apache.spark.rdd.RDD[(String, Double)] = MapPartitionsRDD[54] at map at <console>:30
```    
（5）打印结果  
```scala
scala> result.collect()
res33: Array[(String, Double)] = Array((b,95.33333333333333), (a,91.33333333333333))
```    
**3.3.8 sortByKey([ascending], [numTasks]) 案例**  
1）作用：在一个(K,V)的RDD上调用，K必须实现Ordered接口，返回一个按照key进行排序的(K,V)的RDD  
2）需求：创建一个pairRDD，按照key的正序和倒序进行排序  
（1）创建一个pairRDD  
```scala
scala> val rdd = sc.parallelize(Array((3,"aa"),(6,"cc"),(2,"bb"),(1,"dd")))
rdd: org.apache.spark.rdd.RDD[(Int, String)] = ParallelCollectionRDD[14] at parallelize at <console>:24
```    
（2）按照key的正序  
```scala
scala> rdd.sortByKey(true).collect()
res9: Array[(Int, String)] = Array((1,dd), (2,bb), (3,aa), (6,cc))
```    
（3）按照key的倒序  
```scala
scala> rdd.sortByKey(false).collect()
res10: Array[(Int, String)] = Array((6,cc), (3,aa), (2,bb), (1,dd))
```    
**3.3.9 mapValues案例**  
1）针对于(K,V)形式的类型只对V进行操作  
2）需求：创建一个pairRDD，并将value添加字符串"|||"  
（1）创建一个pairRDD  
```scala
scala> val rdd3 = sc.parallelize(Array((1,"a"),(1,"d"),(2,"b"),(3,"c")))
rdd3: org.apache.spark.rdd.RDD[(Int, String)] = ParallelCollectionRDD[67] at parallelize at <console>:24
```    
（2）对value添加字符串"|||"  
```scala
scala> rdd3.mapValues(_+"|||").collect()
res26: Array[(Int, String)] = Array((1,a|||), (1,d|||), (2,b|||), (3,c|||))
```    
**3.3.10 join(otherDataset, [numTasks]) 案例**  
1）作用：在类型为(K,V)和(K,W)的RDD上调用，返回一个相同key对应的所有元素对在一起的(K,(V,W))的RDD  
2）需求：创建两个pairRDD，并将key相同的数据聚合到一个元组。  
（1）创建第一个pairRDD  
```scala
scala> val rdd = sc.parallelize(Array((1,"a"),(2,"b"),(3,"c")))
rdd: org.apache.spark.rdd.RDD[(Int, String)] = ParallelCollectionRDD[32] at parallelize at <console>:24
```    
（2）创建第二个pairRDD  
```scala
scala> val rdd1 = sc.parallelize(Array((1,4),(2,5),(3,6)))
rdd1: org.apache.spark.rdd.RDD[(Int, Int)] = ParallelCollectionRDD[33] at parallelize at <console>:24
```    
（3）join操作并打印结果  
```scala
scala> rdd.join(rdd1).collect()
res13: Array[(Int, (String, Int))] = Array((1,(a,4)), (2,(b,5)), (3,(c,6)))
```    
**3.3.11 cogroup(otherDataset, [numTasks]) 案例**  
1）作用：在类型为(K,V)和(K,W)的RDD上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的RDD  
2）需求：创建两个pairRDD，并将key相同的数据聚合到一个迭代器。  
（1）创建第一个pairRDD  
```scala
scala> val rdd = sc.parallelize(Array((1,"a"),(2,"b"),(3,"c")))
rdd: org.apache.spark.rdd.RDD[(Int, String)] = ParallelCollectionRDD[37] at parallelize at <console>:24
```    
（2）创建第二个pairRDD  
```scala
scala> val rdd1 = sc.parallelize(Array((1,4),(2,5),(3,6)))
rdd1: org.apache.spark.rdd.RDD[(Int, Int)] = ParallelCollectionRDD[38] at parallelize at <console>:24
```    
（3）cogroup两个RDD并打印结果  
```scala
scala> rdd.cogroup(rdd1).collect()
res14: Array[(Int, (Iterable[String], Iterable[Int]))] = Array((1,(CompactBuffer(a),CompactBuffer(4))), (2,(CompactBuffer(b),CompactBuffer(5))), (3,(CompactBuffer(c),CompactBuffer(6))))
```    
#### 3.4 Action  
**3.4.1 reduce(func)案例**  
1）作用：通过func函数聚集RDD中的所有元素，先聚合分区内数据，再聚合分区间数据。  
2）需求：创建一个RDD，将所有元素聚合得到结果。  
（1）创建一个RDD[Int]  
```scala
scala> val rdd1 = sc.makeRDD(1 to 10,2)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[85] at makeRDD at <console>:24
```    
（2）聚合RDD[Int]所有元素  
```scala
scala> rdd1.reduce(_+_)
res50: Int = 55
```    
（3）创建一个RDD[String]  
```scala
scala> val rdd2 = sc.makeRDD(Array(("a",1),("a",3),("c",3),("d",5)))
rdd2: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[86] at makeRDD at <console>:24
```    
（4）聚合RDD[String]所有数据  
```scala
scala> rdd2.reduce((x,y)=>(x._1 + y._1,x._2 + y._2))
res51: (String, Int) = (adca,12)
```    
**3.4.2 collect()案例**  
1）作用：在驱动程序中，以数组的形式返回数据集的所有元素。  
2）需求：创建一个RDD，并将RDD内容收集到Driver端打印  
（1）创建一个RDD  
```scala
scala> val rdd = sc.parallelize(1 to 10)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:24
```    
（2）将结果收集到Driver端  
```scala
scala> rdd.collect
res0: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)  
```    
**3.4.3 count()案例**  
1）作用：返回RDD中元素的个数  
2）需求：创建一个RDD，统计该RDD的条数  
（1）创建一个RDD  
```scala
scala> val rdd = sc.parallelize(1 to 10)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:24、
```   
（2）统计该RDD的条数  
```scala
scala> rdd.count
res1: Long = 10
```   
**3.4.4 first()案例**  
1）作用：返回RDD中的第一个元素  
2）需求：创建一个RDD，返回该RDD中的第一个元素  
（1）创建一个RDD  
```scala
scala> val rdd = sc.parallelize(1 to 10)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:24
```   
（2）统计该RDD的条数  
```scala
scala> rdd.first
res2: Int = 1
```   
**3.4.5 take(n)案例**  
1）作用：返回一个由RDD的前n个元素组成的数组  
2）需求：创建一个RDD，统计该RDD的条数  
（1）创建一个RDD  
```scala
scala> val rdd = sc.parallelize(Array(2,5,4,6,8,3))
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[2] at parallelize at <console>:24
```   
（2）统计该RDD的条数  
```scala
scala> rdd.take(3)
res10: Array[Int] = Array(2, 5, 4)
```   
**3.4.6 takeOrdered(n)案例**  
1）作用：返回该RDD排序后的前n个元素组成的数组  
2）需求：创建一个RDD，统计该RDD的条数  
（1）创建一个RDD  
```scala
scala> val rdd = sc.parallelize(Array(2,5,4,6,8,3))
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[2] at parallelize at <console>:24
```   
（2）统计该RDD的条数  
```scala
scala> rdd.takeOrdered(3)
res18: Array[Int] = Array(2, 3, 4)
```   
**3.4.7 aggregate案例**  
1）参数：(zeroValue: U)(seqOp: (U, T) ⇒ U, combOp: (U, U) ⇒ U)  
2）作用：aggregate函数将每个分区里面的元素通过seqOp和初始值进行聚合，然后用combine函数将每个分区的结果和初始值(zeroValue)进行combine操作。这个函数最终返回的类型不需要和RDD中元素类型一致。  
3）需求：创建一个RDD，将所有元素相加得到结果  
（1）创建一个RDD  
```scala
scala> var rdd1 = sc.makeRDD(1 to 10,2)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[88] at makeRDD at <console>:24
```   
（2）将该RDD所有元素相加得到结果  
```scala
scala> rdd.aggregate(0)(_+_,_+_)
res22: Int = 55
```   
**3.4.8 fold(num)(func)案例**  
1）作用：折叠操作，aggregate的简化操作，seqop和combop一样。  
2）需求：创建一个RDD，将所有元素相加得到结果  
（1）创建一个RDD  
```scala
scala> var rdd1 = sc.makeRDD(1 to 10,2)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[88] at makeRDD at <console>:24
```   
（2）将该RDD所有元素相加得到结果  
```scala
scala> rdd.fold(0)(_+_)
res24: Int = 55
```   
**3.4.9 saveAsTextFile(path)**  
作用：  
将数据集的元素以textfile的形式保存到HDFS文件系统或者其他支持的文件系统，对于每个元素，Spark将会调用toString方法，将它装换为文件中的文本  
**3.4.10 saveAsSequenceFile(path)**  
作用：  
将数据集中的元素以Hadoop sequencefile的格式保存到指定的目录下，可以使HDFS或者其他Hadoop支持的文件系统。  
**3.4.11 saveAsObjectFile(path)**  
作用：  
用于将RDD中的元素序列化成对象，存储到文件中。  
**3.4.12 countByKey()案例**  
1）作用：针对(K,V)类型的RDD，返回一个(K,Int)的map，表示每一个key对应的元素个数。  
2）需求：创建一个PairRDD，统计每种key的个数  
（1）创建一个PairRDD  
```scala
scala> val rdd = sc.parallelize(List((1,3),(1,2),(1,4),(2,3),(3,6),(3,8)),3)
rdd: org.apache.spark.rdd.RDD[(Int, Int)] = ParallelCollectionRDD[95] at parallelize at <console>:24
```   
（2）统计每种key的个数  
```scala
scala> rdd.countByKey
res63: scala.collection.Map[Int,Long] = Map(3 -> 2, 1 -> 3, 2 -> 1)
```   
**3.4.13 foreach(func)案例**  
1）作用：在数据集的每一个元素上，运行函数func进行更新。  
2）需求：创建一个RDD，对每个元素进行打印  
（1）创建一个RDD  
```scala
scala> var rdd = sc.makeRDD(1 to 5,2)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[107] at makeRDD at <console>:24
```   
（2）对该RDD每个元素进行打印  
```scala
scala> rdd.foreach(println(_))
3
4
5
1
2
```   
