可回答：1）Flume的source、channel、sink分别有什么类型的？2）Flume收集工具有哪些部分组成。   

参考答案：

Flume组成架构如下图所示

<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/Pics/Flume%E9%9D%A2%E8%AF%95%E9%A2%98Pics/Flume-Flume%E6%9E%B6%E6%9E%8401.png" />  
<p align="center">
</p>   

**`Agent`**

Agent是一个JVM进程，它以事件的形式将数据从源头送至目的。

Agent主要有3个部分组成，Source、Channel、Sink。

**`Source`**

Source负责数据的产生或搜集，一般是对接一些RPC的程序或者是其他的flume节点的sink，从数据发生器接收数据，并将接收的数据以Flume的event格式传递给一个或者多个通道Channel，Flume提供多种数据接收的方式，包括**avro**、thrift、**exec**、jms、**spooling directory**、**netcat**、**taildir**、sequence generator、syslog、http、legacy、自定义。

| 类型               | 描述                                                      |
| ------------------ | --------------------------------------------------------- |
| Avro               | 监听Avro端口并接收Avro Client的流数据                     |
| Thrift             | 监听Thrift端口并接收Thrift Client的流数据                 |
| Exec               | 基于Unix的command在标准输出上生产数据                     |
| JMS                | 从JMS（Java消息服务）采集数据                             |
| Spooling Directory | 监听指定目录                                              |
| Twitter 1%         | 通过API持续下载Twitter数据（实验阶段）                    |
| Kafka              | 采集Kafka topic中的message                                |
| NetCat             | 监听端口（要求所提供的数据是换行符分隔的文本）            |
| Sequence Generator | 序列产生器，连续不断产生event，用于测试                   |
| Syslog             | 采集syslog日志消息，支持单端口TCP、多端口TCP和UDP日志采集 |
| HTTP               | 接收HTTP POST和GET数据                                    |
| Stress             | 用于source压力测试                                        |
| Legacy             | 向下兼容，接收低版本Flume的数据                           |
| Custom             | 自定义source的接口                                        |
| Scribe             | 从facebook Scribe采集数据                                 |

**`Channel`**

Channel 是一种短暂的存储容器，负责数据的存储持久化，可以持久化到jdbc、file、memory，将从source处接收到的event格式的数据缓存起来，直到它们被sinks消费掉。可以把channel看成是一个队列，队列的优点是先进先出，放好后尾部一个个event出来，Flume比较看重数据的传输，因此几乎没有数据的解析预处理。仅仅是数据的产生，封装成event然后传输。数据只有存储在下一个存储位置（可能是最终的存储位置，如HDFS，也可能是下一个Flume节点的channel），数据才会从当前的channel中删除。这个过程是通过事务来控制的，这样就保证了数据的可靠性。

不过flume的持久化也是有容量限制的，比如内存如果超过一定的量，不够分配，也一样会爆掉。

| 类型                       | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| Memory                     | Event数据存储在内存中                                        |
| JDBC                       | Event数据存储在持久化存储中，当前Flume Channel内置支持Derby  |
| Kafka                      | Event存储在kafka集群                                         |
| File Channel               | Event数据存储在磁盘文件中                                    |
| Spillable Memory Channel   | Event数据存储在内存中和磁盘上，当内存队列满了，会持久化到磁盘文件（当前试验性的，不建议生产环境使用） |
| Pseudo Transaction Channel | 测试用途                                                     |
| Custom Channel             | 自定义Channel实现                                            |

Channel是位于Source和Sink之间的缓冲区。因此，Channel允许Source和Sink运作在不同的速率上。Channel是线程安全的，可以同时处理几个Source的写入操作和几个Sink的读取操作。

**注意**

- Memory Channel是内存中的队列。Memory Channel在不需要关心数据丢失的情景下适用。如果需要关心数据丢失，那么Memory Channel就不应该使用，因为程序死亡、机器宕机或者重启都会导致数据丢失。

- File Channel将所有事件写到磁盘。因此在程序关闭或机器宕机的情况下不会丢失数据。

**`Sink`**

Sink不断地轮询Channel中的事件且批量地移除它们，并将这些事件批量写入到存储或索引系统、或者被发送到另一个Flume Agent。

Sink组件目的地包括**hdfs、Kafka、logger、avro**、thrift、ipc、**file、HBase**、solr、自定义。

| 类型           | 描述                                                |
| -------------- | --------------------------------------------------- |
| HDFS           | 数据写入HDFS                                        |
| HIVE           | 数据导入到HIVE中                                    |
| Logger         | 数据写入日志文件                                    |
| Avro           | 数据被转换成Avro Event，然后发送到配置的RPC端口上   |
| Thrift         | 数据被转换成Thrift Event，然后发送到配置的RPC端口上 |
| IRC            | 数据在IRC上进行回放                                 |
| File Roll      | 存储数据到本地文件系统                              |
| Null           | 丢弃到所有数据                                      |
| Hive           | 数据写入Hive                                        |
| HBase          | 数据写入HBase数据库                                 |
| Morphline Solr | 数据发送到Solr搜索服务器（集群）                    |
| ElasticSearch  | 数据发送到Elastic Search搜索服务器（集群）          |
| Kite Dataset   | 写数据到Kite Dataset，试验性质的                    |
| Kafka          | 数据写到Kafka Topic                                 |
| Custom         | 自定义Sink实现                                      |

**Event**

传输单元，Flume数据传输的基本单元，以Event的形式将数据从源头送至目的地。Event由Header和Body两部分组成，Header用来存放该event的一些属性，为K-V结构，Body用来存放该条数据，形式为字节数组。   

<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/Pics/Flume%E9%9D%A2%E8%AF%95%E9%A2%98Pics/Flume-Flume%E6%9E%B6%E6%9E%8402.png" />  
<p align="center">
</p>   

**欢迎加入知识星球**   
<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/Pics/%E6%98%9F%E7%90%83%E4%BC%98%E6%83%A0%E5%88%B8%20(21).png"  width="290" height="387"/>  
<p align="center">
</p>   

