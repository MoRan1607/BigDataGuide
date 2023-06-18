可回答：Hadoop块大小及其原因   

参考答案：

**1、首先先来了解几个概念**

寻址时间：HDFS中找到目标文件block块所花费的时间。

原理：文件块越大，寻址时间越短，但磁盘传输时间越长；文件块越小，寻址时间越长，但磁盘传输时间越短。

**2、为什么block不能设置过大，也不能设置过小**

如果块设置过大，一方面从磁盘传输数据的时间会明显大于寻址时间，导致程序在处理这块数据时，变得非常慢；另一方面，MapReduce中的map任务通常一次只处理一个块中的数据，如果块过大运行速度也会很慢。

如果设置过小，一方面存放大量小文件会占用NameNode中大量内存来存储元数据，而NameNode的内存是有限的，不可取；另一方面块过小，寻址时间增长，导致程序一直在找block的开始位置。因此，块适当设置大一些，减少寻址时间，传输一个有多个块组成的文件的时间主要取决于磁盘的传输速度。

**3、块大小多少合适**

如果寻址时间约为10ms，而传输速率为100MB/s，为了使寻址时间仅占传输时间的1%，我们要将块大小设置约为100MB。默认的块大小128MB。    

块的大小：10ms x 100 x 100M/s = 100M，如图  

<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/Hadoop/%E9%9D%A2%E8%AF%95%E9%A2%98/Hadoop/%E9%9D%A2%E8%AF%95%E9%A2%98/HDFS%E7%9A%84block%E4%B8%BA%E4%BB%80%E4%B9%88%E6%98%AF128M%EF%BC%9F%E5%A2%9E%E5%A4%A7%E6%88%96%E5%87%8F%E5%B0%8F%E6%9C%89%E4%BB%80%E4%B9%88%E5%BD%B1%E5%93%8D%EF%BC%9F/Hadoop-Hadoop%E7%9A%84%E9%BB%98%E8%AE%A4%E5%9D%97%E5%A4%A7%E5%B0%8F.png"/>  
<p align="center">
</p>   

如果增加文件块大小，那就需要增加磁盘的传输速率。  

比如，磁盘传输速率为200MB/s时，一般设定block大小为256MB；磁盘传输速率为400MB/s时，一般设定block大小为512MB。   

**欢迎加入知识星球，获取《大数据面试题 V4.0》以及更多大数据开发内容**   
<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/Pics/%E6%98%9F%E7%90%83%E4%BC%98%E6%83%A0%E5%88%B8%20(21).png"  width="290" height="387"/>  
<p align="center">
</p>   
