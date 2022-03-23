先来看下机架感知机制，也就是HDFS上副本存储结点的选择。

<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93/Hadoop/Pics/%E6%9C%BA%E6%9E%B6%E6%84%9F%E7%9F%A5%E6%9C%BA%E5%88%B6.png"/>  
<p align="center">
</p>
</p>  

Hadoop3.x副本结点选择：

由上图可知，第一个副本在Client所处的节点上。如果客户端在集群外，随机选一个。

第二个副本在另一个机架的随机一个节点。

第三个副本在第二个副本所在机架的随机节点。

关于HDFS读写流程，这里还是给出两个版本，有助于理解

第一个版本：简洁版
HDFS写数据流程

<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93/Hadoop/Pics/HDFS%E8%AF%BB%E5%86%99%E6%95%B0%E6%8D%AE01.png"/>  
<p align="center">
</p>
</p>  

1）客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。   

2）NameNode返回是否可以上传。   

3）客户端请求第一个 block上传到哪几个datanode服务器上。   

4）NameNode返回3个datanode节点，分别为dn1、dn2、dn3。   

5）客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。   

6）dn1、dn2、dn3逐级应答客户端。   

7）客户端开始往dn1上传第一个block（先从磁盘读取数据放到一个本地内存缓存），以packet为单位，dn1收到一个packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答。   

8）当一个block传输完成之后，客户端再次请求NameNode上传第二个block的服务器。（重复执行3-7步）。

HDFS读数据流程

<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93/Hadoop/Pics/HDFS%E8%AF%BB%E5%86%99%E6%95%B0%E6%8D%AE02.png"/>  
<p align="center">
</p>
</p>  

1）客户端通过Distributed FileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。   

2）挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据。   

3）DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以packet为单位来做校验）。   

4）客户端以packet为单位接收，先在本地缓存，然后写入目标文件。

第二个版本：详细版，有助于理解
HDFS写数据流程

<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93/Hadoop/Pics/HDFS%E8%AF%BB%E5%86%99%E6%95%B0%E6%8D%AE03.png"/>  
<p align="center">
</p>
</p>  

1）Client将File按128M分块。分成两块，Block1和Block2;

2）Client向nameNode发送写数据请求，如图蓝色虚线①------>。

3）NameNode节点，记录block信息。并返回可用的DataNode，如粉色虚线②------->。

    Block1: host2,host3,host1

    Block2: host7,host8,host4

4）client向DataNode发送block1；发送过程是以流式写入。

流式写入过程：

（1）将64M的block1按64k的package划分;

（2）然后将第一个package发送给host2;

（3）host2接收完后，将第一个package发送给host3，同时client向host2发送第二个package；

（4）host3接收完第一个package后，发送给host1，同时接收host2发来的第二个package。

（5）以此类推，如图红线实线所示，直到将block1发送完毕。

（6）host2，host3，host1向NameNode，host2向Client发送通知，说“消息发送完了”。如图粉红颜色实线所示。

（7）client收到host2发来的消息后，向namenode发送消息，说我写完了。这样就完成了。如图黄色粗实线。

（8）发送完block1后，再向host7，host8，host4发送block2，如图蓝色实线所示。

（9）发送完block2后，host7，host8，host4向NameNode，host7向Client发送通知，如图浅绿色实线所示。

（10）client向NameNode发送消息，说我写完了，如图黄色粗实线。。。这样就完毕了。

HDFS读数据流程

<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93/Hadoop/Pics/HDFS%E8%AF%BB%E5%86%99%E6%95%B0%E6%8D%AE04.png"/>  
<p align="center">
</p>
</p>  

1）client向namenode发送读请求。

2）namenode查看Metadata信息，返回fileA的block的位置。

    Block1: host2,host3,host1

    Block2: host7,host8,host4

3）block的位置是有先后顺序的，先读block1，再读block2。而且block1去host2上读取；然后block2，去host7上读取。

