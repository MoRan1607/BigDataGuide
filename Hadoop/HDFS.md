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





