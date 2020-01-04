Hadoop——MapReduce
===
## 一、MapReduce概述
&emsp; MapReduce是一个**分布式运算程序的编程框架**，是用户开发“基于hadoop的数据分析应用”的核心框架；Mapreduce核心功能是**将用户编写的业务逻辑代码**和**自带默认组件**整合成一个完整的**分布式运算程序**，并发运行在一个hadoop集群上。  

### 1、为什么要MapReduce
&emsp; 1）海量数据在单机上处理因为硬件资源限制，无法胜任  
&emsp; 2）而一旦将单机版程序扩展到集群来分布式运行，将极大增加程序的复杂度和开发难度  
&emsp; 3）引入mapreduce框架后，开发人员可以将绝大部分工作集中在业务逻辑的开发上，而将分布式计算中的复杂性交由框架来处理。   
&emsp; 4）mapreduce分布式方案考虑的问题   
&emsp; &emsp; （1）运算逻辑要不要先分后合？  
&emsp; &emsp; （2）程序如何分配运算任务（切片）？   
&emsp; &emsp; （3）两阶段的程序如何启动？如何协调？  
&emsp; &emsp; （4）整个程序运行过程中的监控？容错？重试？  
&emsp; 分布式方案需要考虑很多问题，但是我们可以将分布式程序中的公共功能封装成框架，让开发人员将精力集中于业务逻辑上。而mapreduce就是这样一个分布式程序的通用框架。  

### 2、MapReduce优缺点
**优点**：  
1）MapReduce 易于编程  
&emsp; 它简单的实现一些接口，就可以完成一个分布式程序，这个分布式程序可以分布到大量廉价的PC机器上运行。也就是说你写一个分布式程序，跟写一个简单的串行程序是一模一样的。就是因为这个特点使得MapReduce编程变得非常流行。   
2）良好的扩展性  
&emsp; 当你的计算资源不能得到满足的时候，你可以通过简单的增加机器来扩展它的计算能力。   
3）高容错性   
&emsp; MapReduce设计的初衷就是使程序能够部署在廉价的PC机器上，这就要求它具有很高的容错性。比如其中一台机器挂了，它可以把上面的计算任务转移到另外一个节点上运行，不至于这个任务运行失败，而且这个过程不需要人工参与，而完全是由Hadoop内部完成的。   
4）适合PB级以上海量数据的离线处理  
&emsp; 可以实现上千台服务器集群并发工作，提供数据处理能力。  
  
**缺点**：  
1）不擅长实时计算  
&emsp; MapReduce无法像Mysql一样，在毫秒或者秒级内返回结果。  
2）不擅长流式计算  
&emsp; 流式计算的输入数据是动态的，而MapReduce的输入数据集是静态的，不能动态变化。这是因为MapReduce自身的设计特点决定了数据源必须是静态的。   
3）不擅长DAG（有向图）计算  
&emsp; 多个应用程序存在依赖关系，后一个应用程序的输入为前一个的输出。在这种情况下，MapReduce并不是不能做，而是使用后，每个MapReduce作业的输出结果都会写入到磁盘，会造成大量的磁盘IO，导致性能非常的低下。  

### 3、MapReduce核心思想
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E6%96%87%E6%A1%A3Pics/MapReduce/MapReduce%E6%A0%B8%E5%BF%83%E6%80%9D%E6%83%B3.png"/>  
<p align="center">
</p>
</p>  

1）分布式的运算程序往往需要分成至少2个阶段。  
2）第一个阶段的MapTask并发实例，完全并行运行，互不相干。   
3）第二个阶段的ReduceTask并发实例互不相干，但是他们的数据依赖于上一个阶段的所有MapTask并发实例的输出。   
4）MapReduce编程模型只能包含一个Map阶段和一个Reduce阶段，如果用户的业务逻辑非常复杂，那就只能多个MapReduce程序，串行运行。  

### 4、MapReduce进程
一个完整的mapreduce程序在分布式运行时有三类实例进程：  
&emsp; 1）MrAppMaster：负责整个程序的过程调度及状态协调。  
&emsp; 2）MapTask：负责map阶段的整个数据处理流程。  
&emsp; 3）ReduceTask：负责reduce阶段的整个数据处理流程。  

### 5、常用数据序列化类型  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E6%96%87%E6%A1%A3Pics/MapReduce/%E5%B8%B8%E7%94%A8%E6%95%B0%E6%8D%AE%E5%BA%8F%E5%88%97%E5%8C%96%E7%B1%BB%E5%9E%8B.png"/>  
<p align="center">
</p>
</p>  

### 6、MapReduce编程规范  
用户编写的程序分成三个部分：Mapper、Reducer和Driver。  
&emsp; 1）**Mapper阶段**  
&emsp; （1）用户自定义的Mapper要继承自己的父类  
&emsp; （2）Mapper的输入数据是KV对的形式（KV的类型可自定义）  
&emsp; （3）Mapper中的业务逻辑写在map()方法中  
&emsp; （4）Mapper的输出数据是KV对的形式（KV的类型可自定义）  
&emsp; （5）**map()方法（MapTask进程）对每一个<K,V>调用一次**  
&emsp; 2）**Reducer阶段**  
&emsp; （1）用户自定义的Reducer要继承自己的父类   
&emsp; （2）Reducer的输入数据类型对应Mapper的输出数据类型，也是KV   
&emsp; （3）Reducer的业务逻辑写在reduce()方法中  
&emsp; （4）**Reducetask进程对每一组相同k的<k,v>组调用一次reduce()方法**  
&emsp; 3）**Driver阶段**  
&emsp; 相当于yarn集群的客户端，用于提交我们整个程序到yarn集群，提交的是封装了mapreduce程序相关运行参数的job对象  

## 二、MapReduce框架原理
&emsp; 介绍MapReduce框架原理之前，先看下MapReduce框架的流程图，了解MapReduce的具体流程  
**MapReduce详细工作流程（一）**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E6%96%87%E6%A1%A3Pics/MapReduce/MapReduce%E8%AF%A6%E7%BB%86%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89.png"/>  
<p align="center">
</p>
</p>  

**MapReduce详细工作流程（二）**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E6%96%87%E6%A1%A3Pics/MapReduce/MapReduce%E8%AF%A6%E7%BB%86%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%EF%BC%88%E4%BA%8C%EF%BC%89.png"/>  
<p align="center">
</p>
</p>  

1-6步，是**InputFormat数据输入阶段 -> Map阶段**  
7-16步，是**Shuffle阶段 -> Reduce阶段 -> OutPutFormat阶段**  

**MapReduce的数据流**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E6%96%87%E6%A1%A3Pics/MapReduce/MapReduce%E7%9A%84%E6%95%B0%E6%8D%AE%E6%B5%81.png"/>  
<p align="center">
</p>
</p>  

### 1、InputFormat数据输入  
1）**Job提交流程源码和切片源码详解**  
（1）Job提交流程源码详解  
```java  
waitForCompletion()
submit();

// 1建立连接
connect();
    // 1）创建提交Job的代理
    new Cluster(getConfiguration());
        //（1）判断是本地yarn还是远程
        initialize(jobTrackAddr, conf);
// 2 提交job
submitter.submitJobInternal(Job.this, cluster)
    // 1）创建给集群提交数据的Stag路径
    Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);
    // 2）获取jobid ，并创建Job路径
    JobID jobId = submitClient.getNewJobID();
    // 3）拷贝jar包到集群
    copyAndConfigureFiles(job, submitJobDir);
    rUploader.uploadFiles(job, jobSubmitDir);
    // 4）计算切片，生成切片规划文件
    writeSplits(job, submitJobDir);
    maps = writeNewSplits(job, jobSubmitDir);
    input.getSplits(job);
    // 5）向Stag路径写XML配置文件
    writeConf(conf, submitJobFile);
    conf.writeXml(out);
    // 6）提交Job,返回提交状态
    status = submitClient.submitJob(jobId, submitJobDir.toString(), job.getCredentials());
```  

（2）FileInputFormat切片源码解析(**input.getSplits(job)**)  
&emsp; （1）找到你数据存储的目录。  
&emsp; （2）开始遍历处理（规划切片）目录下的每一个文件   
&emsp; （3）遍历第一个文件ss.txt   
&emsp; &emsp; a）获取文件大小fs.sizeOf(ss.txt);   
&emsp; &emsp; b）计算切片大小  
&emsp; &emsp; &emsp; computeSliteSize(Math.max(minSize,Math.max(maxSize,blocksize)))=blocksize=128M   
&emsp; &emsp; c）默认情况下，切片大小=blocksize  
&emsp; &emsp; d）开始切，形成第1个切片：ss.txt—0:128M 第2个切片ss.txt—128:256M 第3个切片ss.txt—256M:300M  
&emsp; &emsp; （每次切片时，都要判断切完剩下的部分是否大于块的1.1倍，不大于1.1倍就划分一块切片）  
&emsp; &emsp; e）将切片信息写到一个切片规划文件中   
&emsp; &emsp; f）整个切片的核心过程在getSplit()方法中完成。  
&emsp; &emsp; g）数据切片只是在逻辑上对输入数据进行分片，并不会再磁盘上将其切分成分片进行存储。InputSplit只记录了分片的元数据信息，比如起始位置、长度以及所在的节点列表等。   
&emsp; &emsp; h）注意：block是HDFS上物理上存储的存储的数据，切片是对数据逻辑上的划分。   
&emsp; （4）提交切片规划文件到yarn上，yarn上的MrAppMaster就可以根据切片规划文件计算开启maptask个数。  

（3）FileInputFormat切片机制  
&emsp; 1、默认切片机制  
&emsp; &emsp; （1）简单地按照文件的内容长度进行切片  
&emsp; &emsp; （2）切片大小，默认等于block大小  
&emsp; &emsp; （3）切片时不考虑数据集整体，而是逐个针对每一个文件单独切片  
&emsp; 2、案例分析  
&emsp; 输入数据由两个文件：  
&emsp; &emsp; f1.txt&emsp; 300M  
&emsp; &emsp; f2.txt&emsp; 100M  
&emsp; 经过FileInputFormat的切片机制运算后，形成的切片信息如下：  
&emsp; &emsp; f1.txt.split1&emsp; 0~128M  
&emsp; &emsp; f1.txt.split2&emsp; 128~256M  
&emsp; &emsp; f1.txt.split3&emsp; 256~300M  
&emsp; &emsp; f2.txt.split1&emsp; 0~100M  
&emsp; 3、FileInputFormat切片大小的参数配置  
&emsp; &emsp; 通过分析源码，在FileInputFormat中，计算切片大小的逻辑：  Math.max(minSize, Math.min(maxSize, blockSize));   
&emsp; 切片主要由这几个值来运算决定   
&emsp; &emsp; mapreduce.input.fileinputformat.split.minsize=1&emsp; 默认值为1   
&emsp; &emsp; mapreduce.input.fileinputformat.split.maxsize= Long.MAXValue&emsp; 默认值Long.MAXValue  
&emsp; 因此，默认情况下，切片大小=blocksize。  
&emsp; &emsp; maxsize（切片最大值）：参数如果调得比blocksize小，则会让切片变小，而且就等于配置的这个参数的值。   
&emsp; &emsp; minsize（切片最小值）：参数调的比blockSize大，则可以让切片变得比blocksize还大。  

### 2、MapTask工作机制
1）**数据切片与MapTask并行度决定机制**  
&emsp; （1）一个job的map阶段并行度由客户端在提交job时决定  
&emsp; （2）每一个split切片分配一个mapTask并行实例处理  
&emsp; （3）默认情况下，切片大小=blocksize  
&emsp; （4）切片时不考虑数据集整体，而是逐个针对每一个文件单独切片  

2）**MapTask工作机制**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E6%96%87%E6%A1%A3Pics/MapReduce/MapTask%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6.png"/>  
<p align="center">
</p>
</p>  

（1）`Read阶段`：MapTask通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。  
（2）`Map阶段`：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。  
（3）`Collect收集阶段`：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。   
（4）`Spill阶段`：即**“溢写”**，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。  
&emsp; **溢写阶段详情**：  
&emsp; 步骤1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号partition进行排序，然后按照key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。   
&emsp; 步骤2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。   
&emsp; 步骤3：将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。  
（5）`Combine阶段`：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。  
&emsp; 当所有数据处理完后，MapTask会将所有临时文件合并成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。   
&emsp; 在进行文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并io.sort.factor（默认100）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。   
&emsp; 让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。  

### 3、Shuffle机制
&emsp; Map方法之后，Reduce方法之前的数据处理过程称之为Shuffle。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E6%96%87%E6%A1%A3Pics/MapReduce/Shuffle%E6%9C%BA%E5%88%B6.png"/>  
<p align="center">
</p>
</p>  

具体Shuffle过程详解：  
&emsp; 1）MapTask收集我们的map()方法输出的kv对，放到内存缓冲区中  
&emsp; 2）从内存缓冲区不断溢出本地磁盘文件，可能会溢出多个文件   
&emsp; 3）多个溢出文件会被合并成大的溢出文件  
&emsp; 4）在溢出过程及合并的过程中，都要调用Partitioner进行分区和针对key进行排序   
&emsp; 5）ReduceTask根据自己的分区号，去各个MapTask机器上取相应的结果分区数据   
&emsp; 6）ReduceTask会取到同一个分区的来自不同MapTask的结果文件，ReduceTask会将这些文件再进行合并（归并排序）  
&emsp; 7）**合并成大文件后，Shuffle的过程也就结束了**，后面进入ReduceTask的逻辑运算过程（从文件中取出一个一个的键值对Group，调用用户自定义的reduce()方法）  

1）**Partition分区**  
&emsp; 要求将统计结果按照条件输出到不同文件中（分区）。比如：将统计结果按照手机归属地不同省份输出到不同文件中（分区）  
（1）默认Partition分区  
```java  
public class HashPartitioner<K, V> extends Partitioner<K, V> {
  public int getPartition(K key, V value, int numReduceTasks) {
    return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
  }
}
```  

&emsp; 默认分区是根据key的hashCode对reduceTasks个数取模得到的。用户没法控制哪个key存储到哪个分区。  
（2）自定义Partition步骤  
&emsp; a、自定义类继承Partitioner，重写getPartition()方法  
```java
public class ProvincePartitioner extends Partitioner<Text, FlowBean> {
    @Override
    public int getPartition(Text key, FlowBean value, int numPartitions) {
    //控制分区逻辑代码
            ......
    }
}
```  

&emsp; b、在job驱动中，设置自定义partitioner  
&emsp; &emsp; job.setPartitionerClass(CustomPartitioner.class);    
&emsp; c、自定义partition后，要根据自定义partitioner的逻辑设置相应数量的reduce task   
&emsp; &emsp; job.setNumReduceTasks(5);  
（3）总结  
&emsp; a、如果reduceTask的数量> getPartition的结果数，则会多产生几个空的输出文件part-r-000xx；  
&emsp; b、如果1<reduceTask的数量<getPartition的结果数，则有一部分分区数据无处安放，会Exception；   
&emsp; c、如果reduceTask的数量=1，则不管mapTask端输出多少个分区文件，最终结果都交给这一个reduceTask，最终也就只会产生一个结果文件 part-r-00000；  
&emsp; d、分区数必须从零开始，逐一累加。  
&emsp; 案例：假设自定义分区数为5，则  
&emsp; （1）job.setNumReduceTasks(1);&emsp; 会正常运行，只不过会产生一个输出文件  
&emsp; （2）job.setNumReduceTasks(2);&emsp; 会报错   
&emsp; （3）job.setNumReduceTasks(6);&emsp; 大于5，程序会正常运行，会产生空文件  

2）**排序**  
&emsp; 排序是MapReduce框架中最重要的操作之一。Map Task和Reduce Task均会对数据按照key进行排序。该操作属于Hadoop的默认行为。任何应用程序中的数据均会被排序，而不管逻辑上是否需要。默认排序是按照字典顺序排序，且实现该排序的方法是快速排序。  
&emsp; 对于Map Task，它会将处理的结果暂时放到一个缓冲区中，当缓冲区使用率达到一定阈值后，再对缓冲区中的数据进行一次排序，并将这些有序数据写到磁盘上，而当数据处理完毕后，它会对磁盘上所有文件进行归并排序。   
&emsp; 对于Reduce Task，它从每个Map Task上远程拷贝相应的数据文件，如果文件大小超过一定阈值，则放到磁盘上，否则放到内存中。如果磁盘上文件数目达到一定阈值，则进行一次合并以生成一个更大文件；如果内存中文件大小或者数目超过一定阈值，则进行一次合并后将数据写到磁盘上。当所有数据拷贝完毕后，Reduce Task统一对内存和磁盘上的所有数据进行一次合并。  
&emsp; **排序分类**：  
&emsp; a、部分排序（区内排序——环形缓冲区）  
&emsp; &emsp; MapReduce根据输入记录的键对数据集排序。保证输出的每个文件内部排序。  
&emsp; b、全排序  
&emsp; &emsp; 最终输出结果只有一个文件，且文件内部有序。实现方式是只设置一个ReduceTask，但该方法在处理大型文件时效率极低，因为只有一台机器处理所有文件，完全丧失了MapReduce所提供的并行架构。  
&emsp; c、辅助排序（GroupingComparator分组）  
&emsp; &emsp; 在Reduce端对Key进行分组。应用于：在接受的key为bean对象时，想让一个或几个字段相同（全部字段不相同）的key进入到同一个reduce方法时，可以采用分组排序。  
&emsp; d、二次排序  
&emsp; &emsp; 在自定义排序过程中，如果compareTo中的判断条件为两个即为二次排序。  

3）**Combiner合并**  
（1）combiner是MR程序中Mapper和Reducer之外的一种组件。  
（2）combiner组件的父类就是Reducer。   
（3）combiner和reducer的区别在于运行的位置：  
&emsp; **Combiner是在每一个maptask所在的节点运行**;  
&emsp; **Reducer是接收全局所有Mapper的输出结果**；   
（4）combiner的意义就是对每一个maptask的输出进行局部汇总，以减小网络传输量。   
（5）**combiner能够应用的前提是不能影响最终的业务逻辑**，而且，combiner的输出kv应该跟reducer的输入kv类型要对应起来。  

### 4、ReduceTask工作机制  
1）**ReduceTask工作机制**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E6%96%87%E6%A1%A3Pics/MapReduce/ReduceTask%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6.png"/>  
<p align="center">
</p>
</p>  

（1）`Copy阶段`：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。  
（2）`Merge阶段`：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。   
（3）`Sort阶段`：按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。   
（4）`Reduce阶段`：reduce()函数将计算结果写到HDFS上。  

2）**ReduceTask并行度（个数）**  
&emsp; ReduceTask的并行度同样影响整个Job的执行并发度和执行效率，但与MapTask的并发数由切片数决定不同，ReduceTask数量的决定是可以直接手动设置。  
&emsp; 默认值是1，可自己手动设置为4  
&emsp; **job.setNumReduceTasks(4);**  

### 5、OutputFormat数据输出  
1）OutputFormat接口实现类  
&emsp; OutputFormat是MapReduce输出的基类，所有实现MapReduce输出都实现了 OutputFormat接口。以下是几种常见的OutputFormat实现类。  
&emsp; a、文本输出TextOutputFormat        
&emsp; 默认的输出格式是TextOutputFormat，它把每条记录写为文本行。它的键和值可以是任意类型，因为TextOutputFormat调用toString()方法把它们转换为字符串。  
&emsp; b、SequenceFileOutputFormat   
&emsp; SequenceFileOutputFormat将它的输出写为一个顺序文件。如果输出需要作为后续 MapReduce任务的输入，这便是一种好的输出格式，因为它的格式紧凑，很容易被压缩。  
&emsp; c、自定义OutputFormat  
&emsp; 根据用户需求，自定义实现输出。  
&emsp; （1）自定义一个类继承FileOutputFormat。  
&emsp; （2）改写recordwriter，具体改写输出数据的方法write()。  

### 6、Join的应用  
**Map Join**  
1）适用场景  
&emsp; 一张表十分小、一张表很大的场景。  
2）优点  
&emsp; 思考：在Reduce端处理过多的表，非常容易产生数据倾斜。怎么办？  
&emsp; &emsp; 在Map端缓存多张表，提前处理业务逻辑，这样增加Map端业务，减少Reduce端数据的压力，尽可能的减少数据倾斜。  
3）具体实现：  
&emsp; （1）在Mapper的setup阶段，将文件读取到缓存集合中。  
&emsp; （2）在驱动函数中加载缓存。   
&emsp; &emsp; 缓存普通文件到Task运行节点。   
&emsp; &emsp; **job.addCacheFile(new URI("file:......"));**

**Reduce Join**  
1）原理  
&emsp; Map端的主要工作：为来自不同表(文件)的key/value对打标签以区别不同来源的记录。然后用连接字段作为key，其余部分和新加的标志作为value，最后进行输出。  
&emsp; Reduce端的主要工作：在reduce端以连接字段作为key的分组已经完成，我们只需要在每一个分组当中将那些来源于不同文件的记录(在map阶段已经打标志)分开，最后进行合并就ok了。  
2）缺点  
&emsp; 这种方式的缺点很明显就是会造成map和reduce端也就是**shuffle阶段出现大量的数据传输，效率很低**。（**①reduce端有一个数组，如果数据较大，很容易出现内存溢出；②reduce端进行join很容易出现数据倾斜；③ruduce端join需要从maptask端copy大量数据，会有大量网络IO问题**）

### 7、数据清洗（ETL）  
&emsp; 在运行核心业务MapReduce程序之前，往往要先对数据进行清洗，清理掉不符合用户要求的数据。清理的过程往往只需要运行Mapper程序，不需要运行Reduce程序。  
&emsp; **注意：案例在后面**  










