## 一、NameNode和SecondaryNameNode  

**1、NN和2NN的工作机制**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HDFS%E6%96%87%E6%A1%A3-Pics/NN%E5%92%8C2NN%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6.png"/>  
<p align="center">
</p>
</p>  

1）**第一阶段：NameNode启动**  
&emsp; （1）第一次启动NameNode格式化后，创建fsimage和edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。   
&emsp; （2）客户端对元数据进行增删改的请求。   
&emsp; （3）NameNode记录操作日志，更新滚动日志。   
&emsp; （4）NameNode在内存中对数据进行增删改查。  
2）**第二阶段：Secondary NameNode工作**   
&emsp; （1）Secondary NameNode询问NameNode是否需要checkpoint。直接带回NameNode是否检查结果。   
&emsp; （2）Secondary NameNode请求执行checkpoint。   
&emsp; （3）NameNode滚动正在写的edits日志。   
&emsp; （4）将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode。   
&emsp; （5）Secondary NameNode加载编辑日志和镜像文件到内存，并合并。   
&emsp; （6）生成新的镜像文件fsimage.chkpoint。   
&emsp; （7）拷贝fsimage.chkpoint到NameNode。   
&emsp; （8）NameNode将fsimage.chkpoint重新命名成fsimage。   

`NN和2NN工作机制详解`：  
&emsp; `Fsimage`：namenode内存中元数据序列化后形成的文件。   
&emsp; `Edits`：记录客户端更新元数据信息的每一步操作（可通过Edits运算出元数据）。   
&emsp; namenode启动时，先滚动edits并生成一个空的edits.inprogress，然后加载edits和fsimage到内存中，此时namenode内存就持有最新的元数据信息。client开始对namenode发送元数据的增删改查的请求，这些请求的操作首先会被记录的edits.inprogress中（查询元数据的操作不会被记录在edits中，因为查询操作不会更改元数据信息），如果此时namenode挂掉，重启后会从edits中读取元数据的信息。然后，namenode会在内存中执行元数据的增删改查的操作。   
&emsp; 由于edits中记录的操作会越来越多，edits文件会越来越大，导致namenode在启动加载edits时会很慢，所以需要对edits和fsimage进行合并（所谓合并，就是将edits和fsimage加载到内存中，照着edits中的操作一步步执行，最终形成新的fsimage）。secondarynamenode的作用就是帮助namenode进行edits和fsimage的合并工作。   
&emsp; secondarynamenode首先会询问namenode是否需要checkpoint（触发checkpoint需要满足两个条件中的任意一个，定时时间到和edits中数据写满了）。直接带回namenode是否检查结果。secondarynamenode执行checkpoint操作，首先会让namenode滚动edits并生成一个空的edits.inprogress，滚动edits的目的是给edits打个标记，以后所有新的操作都写入edits.inprogress，其他未合并的edits和fsimage会拷贝到secondarynamenode的本地，然后将拷贝的edits和fsimage加载到内存中进行合并，生成fsimage.chkpoint，然后将fsimage.chkpoint拷贝给namenode，重命名为fsimage后替换掉原来的fsimage。namenode在启动时就只需要加载之前未合并的edits和fsimage即可，因为合并过的edits中的元数据信息已经被记录在fsimage中。  

## 二、DataNode
**1、DataNode工作机制**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HDFS%E6%96%87%E6%A1%A3-Pics/DataNode%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6.png"/>  
<p align="center">
</p>
</p>  

1）一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。2）DataNode启动后向NameNode注册，通过后，周期性（1小时）的向NameNode上报所有的块信息。   
3）心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用。   
4）集群运行中可以安全加入和退出一些机器。  

**2、数据完整性**  
1）当DataNode读取block的时候，它会计算checksum。  
2）如果计算后的checksum，与block创建时值不一样，说明block已经损坏。   
3）client读取其他DataNode上的block。   
4）datanode在其文件创建后周期验证checksum，如下图  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98Pics/HDFS%E6%96%87%E6%A1%A3-Pics/%E6%95%B0%E6%8D%AE%E5%AE%8C%E6%95%B4%E6%80%A7.png"/>  
<p align="center">
</p>
</p>  

**3、掉线时限参数设置**  
&emsp; DataNode进程死亡或者网络故障造成DataNode无法与NameNode通信，NameNode不会立即把该节点判定为死亡，要经过一段时间，这段时间暂称作超时时长。HDFS默认的超时时长为10分钟+30秒。如果定义超时时间为timeout，则超时时长的计算公式为：  
&emsp; timeout  = 2 * dfs.namenode.heartbeat.recheck-interval + 10 * dfs.heartbeat.interval。  
&emsp; 而默认的dfs.namenode.heartbeat.recheck-interval 大小为5分钟，dfs.heartbeat.interval默认为3秒。   
&emsp; 需要注意的是hdfs-site.xml 配置文件中的heartbeat.recheck.interval的单位为毫秒，dfs.heartbeat.interval的单位为秒。  
```xml
<property>  
    <name>dfs.namenode.heartbeat.recheck-interval</name>  
    <value>300000</value>  
</property>  
<property>  
    <name> dfs.heartbeat.interval </name>  
    <value>3</value>  
</property>
```




