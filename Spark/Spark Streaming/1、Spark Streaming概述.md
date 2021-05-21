Spark Streaming概述
---   
>首先，我们来看几个概念
### 1、离线和实时概念  
数据处理的延迟   
1）离线计算  
&emsp; 就是在计算开始前已知所有输入数据，输入数据不会产生变化，一般计算量级较大，计算时间也较长。例如今天早上一点，把昨天累积的日志，计算出所需结果。最经典的就是Hadoop的MapReduce方式。  
2）实时计算  
&emsp; 输入数据是可以以序列化的方式一个个输入并进行处理的，也就是说在开始的时候并不需要知道所有的输入数据。与离线计算相比，运行时间短，计算量级相对较小。强调计算过程的时间要短，即所查当下给出结果。  

### 2、批量和流式概念  
&emsp; 数据处理的方式  
&emsp; **批**：处理离线数据，冷数据。单个处理数据量大，处理速度比流慢。  
&emsp; **流**：在线，实时产生的数据。单次处理的数据量小，但处理速度更快。   
&emsp; 近年来，在Web应用、网络监控、传感监测等领域，兴起了一种新的数据密集型应用——流数据，即**数据以大量、快速、时变的流形式持续到达**。实例：PM2.5检测、电子商务网站用户点击流。  
&emsp; 流数据具有如下特征：  
&emsp; &emsp; 数据快速持续到达，潜在大小也许是无穷无尽的  
&emsp; &emsp; 数据来源众多，格式复杂  
&emsp; &emsp; 数据量大，但是不十分关注存储，一旦经过处理，要么被丢弃，要么被归档存储  
&emsp; &emsp; 注重数据的整体价值，不过分关注个别数据  

### 3、Spark Streaming是什么  
&emsp; **Spark Streaming用于流式数据的处理**。Spark Streaming支持的数据输入源很多，例如：Kafka、Flume、Twitter、ZeroMQ和简单的TCP套接字等等。数据输入后可以用Spark的高度抽象算子如：map、reduce、join、window等进行运算。而结果也能保存在很多地方，如HDFS，数据库等。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Streaming/1/%E5%9B%BE%E7%89%871.png"/>  
<p align="center">
</p>
</p>   

&emsp; 在Spark Streaming中，处理数据的单位是一批而不是单条，而数据采集却是逐条进行的，因此Spark Streaming系统需要设置间隔使得数据汇总到一定的量后再一并操作，这个间隔就是批处理间隔。批处理间隔是Spark Streaming的核心概念和关键参数，它决定了Spark Streaming提交作业的频率和数据处理的延迟，同时也影响着数据处理的吞吐量和性能。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Streaming/1/%E5%9B%BE%E7%89%872.png"/>  
<p align="center">
</p>
</p>   
   
&emsp; 和Spark基于RDD的概念很相似，Spark Streaming使用了一个高**级抽象离散化流(discretized stream)，叫作DStreams。DStreams是随时间推移而收到的数据的序列**。在内部，每个时间区间收到的数据都作为RDD存在，而DStreams是由这些RDD所组成的序列(因此得名“离散化”)。DStreams可以由来自数据源的输入数据流来创建, 也可以通过在其他的 DStreams上应用一些高阶操作来得到。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Streaming/1/%E5%9B%BE%E7%89%873.png"/>  
<p align="center">
</p>
</p>   

### 4、Spark Streaming特点  
**优点**  
1）**易用**  
&emsp; Spark Streaming支持Java、Python、Scala等编程语言，可以像编写批处理作业一样编写流式作业。  
2）**容错性**  
&emsp; Spark Streaming在没有额外代码和配置的情况下，可以恢复丢失的数据。对于实时计算来说，容错性至关重要。首先要明确一下Spak中RDD的容错机制，即每一个RDD都是个不可变的分布式可重算的数据集，它记录着确定性的操作继承关系(lineage)，所以只要输入数据是可容错的，那么任意一个RDD的分区(Partition)出错或不可用，都可以使用原始输入数据经过转换操作重新计算得到。  
3）**易整合性**  
&emsp; Spark Streaming可以在Spark上运行，并且还允许重复使用相同的代码进行批处理。也就是说，实时处理可以与离线处理相结合，实现交互式的查询操作。  
**缺点**  
&emsp; Spark Streaming是一种“微量批处理”架构, 和其他基于“一次处理一条记录”架构的系统相比, 它的延迟会相对高一些。  

### 5、Spark Streaming架构  
#### 1）架构图  
（1）Spark Streaming架构图  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Streaming/1/%E5%9B%BE%E7%89%874.png"/>  
<p align="center">
</p>
</p>   

（2）整体架构图  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Streaming/1/%E5%9B%BE%E7%89%875.png"/>  
<p align="center">
</p>
</p>   

#### 2）背压机制  
&emsp; Spark 1.5以前版本，用户如果要限制Receiver的数据接收速率，可以通过设置静态配制参数“spark.streaming.receiver.maxRate”的值来实现，此举虽然可以通过限制接收速率，来适配当前的处理能力，防止内存溢出，但也会引入其它问题。比如：producer数据生产高于maxRate，当前集群处理能力也高于maxRate，这就会造成资源利用率下降等问题。  
&emsp; 为了更好的协调数据接收速率与资源处理能力，1.5版本开始Spark Streaming可以动态控制数据接收速率来适配集群数据处理能力。**背压机制（即Spark Streaming Backpressure）: 根据JobScheduler反馈作业的执行信息来动态调整Receiver数据接收率**。  
&emsp; 通过属性“spark.streaming.backpressure.enabled”来控制是否启用backpressure机制，默认值false，即不启用。  
