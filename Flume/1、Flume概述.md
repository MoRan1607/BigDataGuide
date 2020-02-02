一、Flume概述
---
### 1、Flume
&emsp; Flume是Cloudera提供的一个**高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统**，Flume基于流式架构，灵活简单。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flume%E6%96%87%E6%A1%A3Pics/Flume%E6%B5%81%E7%A8%8B.png"/>  
<p align="center">
</p>
</p>  

&emsp; **主要作用**：实时读取服务器本地磁盘的数据，将数据写入到HDFS。  

### 2、Flume优点
1）可以和任意存储进程集成。  
2）输入的的数据速率大于写入目的存储的速率，flume会进行缓冲，减小hdfs的压力。  
3）flume中的事务基于channel，使用了两个事务模型（sender + receiver），确保消息被可靠发送。  
&emsp; Flume使用两个独立的事务分别负责从soucrce到channel，以及从channel到sink的事件传递。一旦事务中所有的数据全部成功提交到channel，那么source才认为该数据读取完成。同理，只有成功被sink写出去的数据，才会从channel中移除。  

### 3、Flume架构
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flume%E6%96%87%E6%A1%A3Pics/Flume%E6%9E%B6%E6%9E%84.png"/>  
<p align="center">
</p>
</p>  

`Source`数据输入端的类型：avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy等。但是**目前在企业中使用最广泛的就是日志文件**。  
`Channel`位于Source和Sink之间的缓冲区。  
&emsp; Flume自带两种Channel：Memory Channel和File Channel。  
&emsp; Memory Channel是基于内存缓存，在不需要关心数据丢失的情景下适用。  
&emsp; File Channel是Flume的持久化Channel。系统宕机不会丢失数据。  
`Sink`组件目的地包括hdfs、kafka、logger、avro、thrift、ipc、file、null、HBase、solr、自定义。但是目前在企业中使用最广泛的是HDFS和Kafka。  
  
1）**`Agent`**  
&emsp; Agent是一个JVM进程，它以事件的形式将数据从源头送至目的，是**Flume数据传输的基本单元**。  
&emsp; Agent主要有3个部分组成，Source、Channel、Sink。  
2）**`Source`**  
&emsp; Source是**负责接收数据到Flume Agent的组件**。Source组件可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy。  
3）**`Channel`**  
&emsp; Channel是**位于Source和Sink之间的缓冲区**。因此，Channel允许Source和Sink运作在不同的速率上。Channel是线程安全的，可以同时处理几个Source的写入操作和几个Sink的读取操作。  
&emsp; Flume自带两种Channel：Memory Channel和File Channel。  
&emsp; **Memory Channel是内存中的队列**。Memory Channel**在不需要关心数据丢失的情景下适用**。如果需要关心数据丢失，那么Memory Channel就不应该使用，因为程序死亡、机器宕机或者重启都会导致数据丢失。  
&emsp; **File Channel将所有事件写到磁盘**。因此在程序关闭或机器宕机的情况下不会丢失数据。  
4）**`Sink`**  
&emsp; **Sink不断地轮询Channel中的事件且批量地移除它们，并将这些事件批量写入到存储或索引系统、或者被发送到另一个Flume Agent**。（sink从channel中拉取数据，然后推给下一个组件）  
&emsp; Sink是**完全事务性的**。在从Channel批量删除数据之前，每个Sink用Channel启动一个事务。批量事件一旦成功写出到存储系统或下一个Flume Agent，Sink就利用Channel提交事务。事务一旦被提交，该Channel从自己的内部缓冲区删除事件。  
&emsp; Sink组件目的地包括hdfs、logger、avro、thrift、ipc、file、null、HBase、solr、自定义。  
5）**`Event`**  
&emsp; 传输单元，**Flume数据传输的基本单元**，以事件的形式将数据从源头送至目的地。  

### 4、Flume拓扑结构
1）Flume Agent连接  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flume%E6%96%87%E6%A1%A3Pics/Flume%20Agent%E8%BF%9E%E6%8E%A5.png"/>  
<p align="center">
</p>
</p>  

&emsp; 该模式是将多个flume给顺序连接起来了，从最初的source开始到最终sink传送的目的存储系统。此模式不建议桥接过多的flume数量， flume数量过多不仅会影响传输速率，而且一旦传输过程中某个节点flume宕机，会影响整个传输系统。  

2）单source，多channel、sink  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flume%E6%96%87%E6%A1%A3Pics/%E5%8D%95source%EF%BC%8C%E5%A4%9Achannel%E3%80%81sink.png"/>  
<p align="center">
</p>
</p>  

&emsp; Flume支持将事件流向一个或者多个目的地。这种模式将数据源复制到多个channel中，每个channel都有相同的数据，sink可以选择传送的不同的目的地。  

3）负载均衡  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flume%E6%96%87%E6%A1%A3Pics/%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1.png"/>  
<p align="center">
</p>
</p>  

&emsp; Flume支持使用将多个sink逻辑上分到一个sink组，flume将数据发送到不同的sink，主要解决负载均衡和故障转移问题。

4）Flume Agent聚合  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flume%E6%96%87%E6%A1%A3Pics/Flume%20Agent%E8%81%9A%E5%90%88.png"/>  
<p align="center">
</p>
</p>  

&emsp; 这种模式是最常见的，也非常实用，日常web应用通常分布在上百个服务器，大者甚至上千个、上万个服务器。产生的日志，处理起来也非常麻烦。用flume的这种组合方式能很好的解决这一问题，每台服务器部署一个flume采集日志，传送到一个集中收集日志的flume，再由此flume上传到hdfs、hive、hbase、jms等，进行日志分析。

### 5、Flume Agent内部原理
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flume%E6%96%87%E6%A1%A3Pics/Flume%20Agent%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86.png"/>  
<p align="center">
</p>
</p>  

&emsp; Channel Selectors由两种类型：Replicating Channel Selector（默认）和Multiplexing Channel Selector。**Replicating会将source过来的events发往所有channel，而Multiplexing可以配置发往哪些Channel**。


