​
1、相关问题描述

当我们使用spark sql执行etl时候出现了，可能最终结果大小只有几百k，但是小文件一个分区有上千的情况。

这样就会导致以下的一些危害：




2、解决方案

1） 方法一：通过spark的coalesce()方法和repartition()方法

```scala
val rdd2 = rdd1.coalesce(8, true) // true表示是否shuffle    
val rdd3 = rdd1.repartition(8)   
```  
coalesce：coalesce()方法的作用是返回指定一个新的指定分区的Rdd，如果是生成一个窄依赖的结果，那么可以不发生shuffle，分区的数量发生激烈的变化，计算节点不足，不设置true可能会出错。   

repartition：coalesce()方法shuffle为true的情况。    

2）方法二：降低spark并行度，即调节spark.sql.shuffle.partitions     

比如之前设置的为100，按理说应该生成的文件数为100；但是由于业务比较特殊，采用的大量的union all，且union all在spark中属于窄依赖，不会进行shuffle，所以导致最终会生成（union all数量+1）*100的文件数。如有10个union all，会生成1100个小文件。这样导致降低并行度为10之后，执行时长大大增加，且文件数依旧有110个，效果有，但是不理想。    

3）方法三：新增一个并行度=1任务，专门合并小文件。   

先将原来的任务数据写到一个临时分区（如tmp）；再起一个并行度为1的任务，类似：    
```sql
insert overwrite 目标表 select * from 临时分区
```   
但是结果小文件数还是没有减少，原因：‘select * from 临时分区’ 这个任务在spark中属于窄依赖；并且spark DAG中分为宽依赖和窄依赖，只有宽依赖会进行shuffle；故并行度shuffle，spark.sql.shuffle.partitions=1也就没有起到作用；由于数据量本身不是特别大，所以可以直接采用group by（在spark中属于宽依赖）的方式，类似：
```sql
insert overwrite 目标表 select * from 临时分区 group by *
```   
先运行原任务，写到tmp分区，‘dfs -count’查看文件数，1100个，运行加上group by的临时任务（spark.sql.shuffle.partitions=1），查看结果目录，文件数=1，成功。   

最后又加了个删除tmp分区的任务。    

3、总结   

1）方便的话，可以采用coalesce()方法和repartition()方法。  

2）如果任务逻辑简单，数据量少，可以直接降低并行度。   

3）任务逻辑复杂，数据量很大，原任务大并行度计算写到临时分区，再加两个任务：一个用来将临时分区的文件用小并行度（加宽依赖）合并成少量文件到实际分区；另一个删除临时分区。   

4）hive任务减少小文件相对比较简单，可以直接设置参数，如：  

Map-only的任务结束时合并小文件：  
```scala
sethive.merge.mapfiles = true  
```    
在Map-Reduce的任务结束时合并小文件：  
```scala
sethive.merge.mapredfiles= true  
```     
当输出文件的平均大小小于1GB时，启动一个独立的map-reduce任务进行文件merge：   
```scala
sethive.merge.smallfiles.avgsize=1024000000 
```    

​
