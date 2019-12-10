## Hadoop面试题总结（二）——HDFS  

### 1、 HDFS 中的 block 默认保存几份？  
&emsp; 默认保存3份  

### 2、HDFS 默认 BlockSize 是多大？  
&emsp; 默认64MB  

### 3、负责HDFS数据存储的是哪一部分？  
&emsp; DataNode负责数据存储  

### 4、SecondaryNameNode的目的是什么？  
&emsp; 他的目的使帮助NameNode合并编辑日志，减少NameNode 启动时间  

### 5、文件大小设置，增大有什么影响？  
&emsp; HDFS中的文件在物理上是分块存储（block），块的大小可以通过配置参数( dfs.blocksize)来规定，默认大小在hadoop2.x版本中是128M，老版本中是64M。  
&emsp; **思考：为什么块的大小不能设置的太小，也不能设置的太大？**  
&emsp; &emsp; HDFS的块比磁盘的块大，其目的是为了最小化寻址开销。如果块设置得足够大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。
因而，**传输一个由多个块组成的文件的时间取决于磁盘传输速率**。  
&emsp; 如果寻址时间约为10ms，而传输速率为100MB/s，为了使寻址时间仅占传输时间的1%，我们要将块大小设置约为100MB。默认的块大小128MB。  
&emsp; 块的大小：10ms×100×100M/s = 100M，如图  
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HDFS%E5%9D%97.png"/>  
&emsp; 增加文件块大小，需要增加磁盘的传输速率。  

### 6、hadoop的块大小，从哪个版本开始是128M  
&emsp; Hadoop1.x都是64M，hadoop2.x开始都是128M。  

### 7、HDFS的存储机制（☆☆☆☆☆）  
&emsp; HDFS存储机制，包括HDFS的**写入数据过程**和**读取数据过程**两部分  
&emsp; **HDFS写数据过程**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HDFS%E5%86%99%E6%95%B0%E6%8D%AE%E6%B5%81%E7%A8%8B.png"/>  
<p align="center">
</p>
</p>  

&emsp; 1）客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。  
&emsp; 2）NameNode返回是否可以上传。  
&emsp; 3）客户端请求第一个 block上传到哪几个datanode服务器上。  
&emsp; 4）NameNode返回3个datanode节点，分别为dn1、dn2、dn3。  
&emsp; 5）客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。  
&emsp; 6）dn1、dn2、dn3逐级应答客户端。  
&emsp; 7）客户端开始往dn1上传第一个block（先从磁盘读取数据放到一个本地内存缓存），以packet为单位，dn1收到一个packet就会传给dn2，dn2传给dn3；
dn1每传一个packet会放入一个应答队列等待应答。  
&emsp; 8）当一个block传输完成之后，客户端再次请求NameNode上传第二个block的服务器。（重复执行3-7步）。  

&emsp; **HDFS读数据过程**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HDFS%E8%AF%BB%E6%95%B0%E6%8D%AE%E6%B5%81%E7%A8%8B.png"/>  
<p align="center">
</p>
</p>  

&emsp; 1）客户端通过Distributed FileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。  
&emsp; 2）挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据。  
&emsp; 3）DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以packet为单位来做校验）。  
&emsp; 4）客户端以packet为单位接收，先在本地缓存，然后写入目标文件。  








