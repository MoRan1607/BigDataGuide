Hadoop——HDFS
===
## 一、HDFS概述
### 1、产生背景
&emsp; 随着数据量越来越大，在一个操作系统管辖的范围内存不下了，那么就分配到更多的操作系统管理的磁盘中，但是不方便管理和维护，迫切**需要一种系统来管理多台机器上的文件**，这就是分布式文件管理系统。**HDFS只是分布式文件管理系统中的一种**。  

### 2、概念  
&emsp; HDFS（Hadoop Distributed File System），它是一个文件系统，用于存储文件，通过目录树来定位文件；其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。  
&emsp; HDFS的设计适合一次写入，多次读出的场景，且不支持文件的修改。适合用来做数据分析，并不适合用来做网盘应用。  

### 3、优缺点
**优点**：  
1）高容错性  
&emsp; （1）数据自动保存多个副本。它通过增加副本的形式，提高容错性；  
&emsp; （2）某一个副本丢失以后，它可以自动恢复。  
2）适合大数据处理  
&emsp; （1）数据规模：能够处理数据规模达到GB、TB甚至PB级别的数据；  
&emsp; （2）文件规模：能够处理百万规模以上的文件数量，数量相当之大。  
3）流式数据访问，它能保证数据的一致性。  
4）可构建在廉价机器上，通过多副本机制，提高可靠性。  
**缺点**：  
1）不适合低延时数据访问，比如毫秒级的存储数据，是做不到的。  
2）无法高效的对大量小文件进行存储。         
&emsp; （1）存储大量小文件的话，它会占用 NameNode大量的内存来存储文件、目录和块信息。这样是不可取的，因为NameNode的内存总是有限的；         
&emsp; （2）小文件存储的寻址时间会超过读取时间，它违反了HDFS的设计目标。   
3）不支持并发写入、文件随机修改。         
&emsp; （1）一个文件只能有一个写，不允许多个线程同时写；         
&emsp; （2）仅支持数据append（追加），不支持文件的随机修改。

### 4、HDFS组成架构（重点）
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HDFS%E6%96%87%E6%A1%A3-Pics/HDFS%E7%BB%84%E6%88%90%E6%9E%B6%E6%9E%84.png"/>  
<p align="center">
</p>
</p>  

这种架构主要由四个部分组成，分别为HDFS Client、NameNode、DataNode和Secondary NameNode。下面我们分别介绍这四个组成部分。  
1）Client：就是客户端。        
&emsp; （1）文件切分。文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行存储；         
&emsp; （2）与NameNode交互，获取文件的位置信息；         
&emsp; （3）与DataNode交互，读取或者写入数据；        
&emsp; （4）Client提供一些命令来管理HDFS，比如启动或者关闭HDFS；         
&emsp; （5）Client可以通过一些命令来访问HDFS；  
2）NameNode：就是Master，它是一个主管、管理者。        
&emsp; （1）管理HDFS的名称空间；         
&emsp; （2）管理数据块（Block）映射信息；         
&emsp; （3）配置副本策略；       
&emsp; （4）处理客户端读写请求。  
3）DataNode：就是Slave。NameNode下达命令，DataNode执行实际的操作。       
&emsp; （1）存储实际的数据块；         
&emsp; （2）执行数据块的读/写操作。  
4）Secondary NameNode：并非NameNode的热备。当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务。        
&emsp; （1）辅助NameNode，分担其工作量；         
&emsp; （2）定期合并Fsimage和Edits，并推送给NameNode；         
&emsp; （3）在紧急情况下，可辅助恢复NameNode。  
  
[**NameNode、Secondary NameNode、DataNode工作机制详解**](https://github.com/Dr11ft/BigDataGuide/blob/master/Hadoop/NN%E3%80%812NN%E3%80%81DN%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6.md)    

## 二、HDFS数据流
**1、HDFS的写数据流程**
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HDFS%E6%96%87%E6%A1%A3-Pics/HDFS%E7%9A%84%E5%86%99%E6%95%B0%E6%8D%AE%E6%B5%81%E7%A8%8B.png"/>  
<p align="center">
</p>
</p>  

步骤：  
&emsp; 1）客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。   
&emsp; 2）NameNode返回是否可以上传。   
&emsp; 3）客户端请求第一个 block上传到哪几个datanode服务器上。   
&emsp; 4）NameNode返回3个datanode节点，分别为dn1、dn2、dn3。   
&emsp; 5）客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。   
&emsp; 6）dn1、dn2、dn3逐级应答客户端。   
&emsp; 7）客户端开始往dn1上传第一个block（先从磁盘读取数据放到一个本地内存缓存），以packet为单位，dn1收到一个packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答。   
&emsp; 8）当一个block传输完成之后，客户端再次请求NameNode上传第二个block的服务器。（重复执行3-7步）。  

**网络拓扑概念**  
&emsp; 在本地网络中，两个节点被称为“彼此近邻”是什么意思？在海量数据处理中，其主要限制因素是节点之间数据的传输速率——带宽很稀缺。这里的想法是将两个节点间的带宽作为距离的衡量标准。  
&emsp; **节点距离：两个节点到达最近的共同祖先的距离总和**。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HDFS%E6%96%87%E6%A1%A3-Pics/%E7%BD%91%E7%BB%9C%E6%8B%93%E6%89%91%E6%A6%82%E5%BF%B5.png"/>  
<p align="center">
</p>
</p>  

 &emsp; 例如，假设有数据中心d1机架r1中的节点n1。该节点可以表示为/d1/r1/n1。利用这种标记，这里给出四种距离描述，如上图  

**机架感知**  
（1）[官方ip地址](http://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-common/RackAwareness.html)  
（2）低版本Hadoop副本节点选择  
&emsp; 第一个副本在Client所处的节点上。如果客户端在集群外，随机选一个。  
&emsp; 第二个副本和第一个副本位于不相同机架的随机节点上。  
&emsp; 第三个副本和第二个副本位于相同机架，节点随机。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HDFS%E6%96%87%E6%A1%A3-Pics/%E6%9C%BA%E6%9E%B6%E6%84%9F%E7%9F%A51.png"/>  
<p align="center">
</p>
</p>  

（3）Hadoop2.7.2副本节点选择  
&emsp; 第一个副本在Client所处的节点上。如果客户端在集群外，随机选一个。   
&emsp; 第二个副本和第一个副本位于相同机架，随机节点。   
&emsp; 第三个副本位于不同机架，随机节点。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HDFS%E6%96%87%E6%A1%A3-Pics/%E6%9C%BA%E6%9E%B6%E6%84%9F%E7%9F%A52.png"/>  
<p align="center">
</p>
</p>  

**2、HDFS的读数据流程**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HDFS%E6%96%87%E6%A1%A3-Pics/HDFS%E7%9A%84%E8%AF%BB%E6%95%B0%E6%8D%AE%E6%B5%81%E7%A8%8B.png"/>  
<p align="center">
</p>
</p>  

步骤：  
&emsp; 1）客户端通过Distributed FileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。   
&emsp; 2）挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据。   
&emsp; 3）DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以packet为单位来做校验）。   
&emsp; 4）客户端以packet为单位接收，先在本地缓存，然后写入目标文件。  



