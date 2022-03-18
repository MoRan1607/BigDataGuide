<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/HBase%E9%9D%A2%E8%AF%95%E9%A2%98%E6%95%B4%E7%90%86/HBase/Pics/HBase%E6%9E%B6%E6%9E%84.png"/>  
<p align="center">
</p>
</p>  

从Hbase的架构图上可以看出，Hbase中的存储包括HMaster、HRegionSever、HRegion、HLog、Store、MemStore、StoreFile、HFile等。

Hbase中的每张表都通过键按照一定的范围被分割成多个子表（HRegion），默认一个HRegion超过256M就要被分割成两个，这个过程由HRegionServer管理,而HRegion的分配由HMaster管理。

1）HMaster的作用：

为HRegionServer分配HRegion

负责HRegionServer的负载均衡

发现失效的HRegionServer并重新分配

HDFS上的垃圾文件回收

处理Schema更新请求

2）HRegionServer的作用：

维护HMaster分配给它的HRegion，处理对这些HRegion的IO请求

负责切分正在运行过程中变得过大的HRegion

通过架构可以得知，Client访问Hbase上的数据并不需要HMaster参与，寻址访问ZooKeeper和HRegionServer，数据读写访问HRegionServer，HMaster仅仅维护Table和Region的元数据信息，Table的元数据信息保存在ZooKeeper上，负载很低。HRegionServer存取一个子表时，会创建一个HRegion对象，然后对表的每个列簇创建一个Store对象，每个Store都会有一个MemStore和0或多个StoreFile与之对应，每个StoreFile都会对应一个HFile，HFile就是实际的存储文件。因此，一个HRegion有多少列簇就有多少个Store。

一个HRegionServer会有多个HRegion和一个HLog。

3）HRegion

Table在行的方向上分割为多个HRegion，HRegion是Hbase中分布式存储和负载均衡的最小单元，即不同的HRegion可以分别在不同的HRegionServer上，但同一个HRegion是不会拆分到多个HRegionServer上的。HRegion按大小分割，每个表一般只有一个HRegion，随着数据不断插入表，HRegion不断增大，当HRegion的某个列簇达到一个阀值（默认256M）时就会分成两个新的HRegion。

<表名，StartRowKey, 创建时间>

由目录表(-ROOT-和.META.)记录该Region的EndRowKey

HRegion被分配给哪个HRegionServer是完全动态的，所以需要机制来定位HRegion具体在哪个HRegionServer，Hbase使用三层结构来定位HRegion：

通过zk里的文件/Hbase/rs得到-ROOT-表的位置。-ROOT-表只有一个region。

通过-ROOT-表查找.META.表的第一个表中相应的HRegion位置。其实-ROOT-表是.META.表的第一个region；.META.表中的每一个Region在-ROOT-表中都是一行记录。

通过.META.表找到所要的用户表HRegion的位置。用户表的每个HRegion在.META.表中都是一行记录。-ROOT-表永远不会被分隔为多个HRegion，保证了最多需要三次跳转，就能定位到任意的region。Client会将查询的位置信息保存缓存起来，缓存不会主动失效，因此如果Client上的缓存全部失效，则需要进行6次网络来回，才能定位到正确的HRegion，其中三次用来发现缓存失效，另外三次用来获取位置信息。

4）Store

每一个HRegion由一个或多个Store组成，至少是一个Store，Hbase会把一起访问的数据放在一个Store里面，即为每个ColumnFamily建一个Store，如果有几个ColumnFamily，也就有几个Store。一个Store由一个MemStore和0或者多个StoreFile组成。 Hbase以Store的大小来判断是否需要切分HRegion。

5）MemStore

MemStore 是放在内存里的，保存修改的数据即keyValues。当MemStore的大小达到一个阀值（默认64MB）时，MemStore会被Flush到文件，即生成一个快照。目前Hbase会有一个线程来负责MemStore的Flush操作。

6）StoreFile

MemStore内存中的数据写到文件后就是StoreFile，StoreFile底层是以HFile的格式保存。

7）HFile

Hbase中KeyValue数据的存储格式，是Hadoop的二进制格式文件。 首先HFile文件是不定长的，长度固定的只有其中的两块：Trailer和FileInfo。

Trailer中有指针指向其他数据块的起始点，FileInfo记录了文件的一些meta信息。Data Block是Hbase IO的基本单元，为了提高效率，HRegionServer中有基于LRU的Block Cache机制。每个Data块的大小可以在创建一个Table的时候通过参数指定（默认块大小64KB），大号的Block有利于顺序Scan，小号的Block利于随机查询。每个Data块除了开头的Magic以外就是一个个KeyValue对拼接而成，Magic内容就是一些随机数字，目的是防止数据损坏，结构如下。

<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/HBase%E9%9D%A2%E8%AF%95%E9%A2%98%E6%95%B4%E7%90%86/HBase/Pics/HBase-%E6%9E%B6%E6%9E%84-HFile.png"/>  
<p align="center">
</p>
</p>  

 HFile结构图如下：

<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/HBase%E9%9D%A2%E8%AF%95%E9%A2%98%E6%95%B4%E7%90%86/HBase/Pics/HBase-%E6%9E%B6%E6%9E%84-HFile%E7%BB%93%E6%9E%84.png"/>  
<p align="center">
</p>
</p>  

Data Block段用来保存表中的数据，这部分可以被压缩。 Meta Block段（可选的）用来保存用户自定义的kv段，可以被压缩。 FileInfo段用来保存HFile的元信息，不能被压缩，用户也可以在这一部分添加自己的元信息。 Data Block Index段（可选的）用来保存Meta Blcok的索引。 Trailer这一段是定长的。保存了每一段的偏移量，读取一个HFile时，会首先读取Trailer，Trailer保存了每个段的起始位置(段的Magic Number用来做安全check)，然后，DataBlock Index会被读取到内存中，这样，当检索某个key时，不需要扫描整个HFile，而只需从内存中找到key所在的block，通过一次磁盘io将整个 block读取到内存中，再找到需要的key。DataBlock Index采用LRU机制淘汰。 HFile的Data Block，Meta Block通常采用压缩方式存储，压缩之后可以大大减少网络IO和磁盘IO，随之而来的开销当然是需要花费cpu进行压缩和解压缩。（备注：DataBlock Index的缺陷：a) 占用过多内存　b) 启动加载时间缓慢）

8）HLog

HLog(WAL log)：WAL意为write ahead log，用来做灾难恢复使用，HLog记录数据的所有变更，一旦region server 宕机，就可以从log中进行恢复。
