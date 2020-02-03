二、Kafka深入解析
---
### 1、Kafka工作流程及文件存储机制
1）Kafka工作流程  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/Kafka%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.png"/>  
<p align="center">
</p>
</p>  

&emsp; Kafka中消息是以topic进行分类的，生产者生产消息，消费者消费消息，都是面向topic的。  
&emsp; topic是逻辑上的概念，而partition是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是producer生产的数据。Producer生产的数据会被不断追加到该log文件末端，且每条数据都有自己的offset。消费者组中的每个消费者，都会实时记录自己消费到了哪个offset，以便出错恢复时，从上次的位置继续消费。  

2）Kafka文件存储机制  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/Kafka%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8%E6%9C%BA%E5%88%B6.png"/>  
<p align="center">
</p>
</p>  

&emsp; 由于生产者生产的消息会不断追加到log文件末尾，为防止log文件过大导致数据定位效率低下，Kafka采取了分片和索引机制，将每个partition分为多个segment。每个segment对应两个文件——“.index”文件和“.log”文件。这些文件位于一个文件夹下，该文件夹的命名规则为：topic名称+分区序号。例如，first这个topic有三个分区，则其对应的文件夹为first-0,first-1,first-2。  
```txt
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log
```  

&emsp; index和log文件以当前segment的第一条消息的offset命名。下图为index文件和log文件的结构示意图。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/index%E6%96%87%E4%BB%B6%E5%92%8Clog%E6%96%87%E4%BB%B6%E7%9A%84%E7%BB%93%E6%9E%84%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg"/>  
<p align="center">
</p>
</p>  

&emsp; **“.index”文件存储大量的索引信息，“.log”文件存储大量的数据**，索引文件中的元数据指向对应数据文件中message的物理偏移地址。  

### 2、Kafka生产者
1）分区策略  
（1）分区的原因  
&emsp; ① **方便在集群中扩展**，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个Partition组成，因此整个集群就可以适应任意大小的数据了；  
&emsp; ② **可以提高并发**，因为可以以Partition为单位读写了。  
（2）分区的原则  
&emsp; 我们需要将producer发送的数据封装成一个ProducerRecord对象。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/ProducerRecord%E5%AF%B9%E8%B1%A1.png"/>  
<p align="center">
</p>
</p>  

&emsp; ① 指明partition的情况下，直接将指明的值直接作为partiton值；  
&emsp; ② 没有指明partition值但有key的情况下，将key的hash值与topic的partition数进行取余得到partition值；  
&emsp; ③ 既没有partition值又没有key值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与topic可用的partition总数取余得到partition值，也就是常说的round-robin算法。  

2）数据可靠性保证  
&emsp; 为保证producer发送的数据，能可靠的发送到指定的topic，topic的每个partition收到producer发送的数据后，都需要向producer发送ack（acknowledgement确认收到），如果producer收到ack，就会进行下一轮的发送，否则重新发送数据。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/%E6%95%B0%E6%8D%AE%E5%8F%AF%E9%9D%A0%E6%80%A7%E4%BF%9D%E8%AF%81.png"/>  
<p align="center">
</p>
</p>  

（1）副本同步策略：  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/%E5%89%AF%E6%9C%AC%E5%90%8C%E6%AD%A5%E7%AD%96%E7%95%A5.png"/>  
<p align="center">
</p>
</p>  

Kafka选择了第二种方案，原因如下：  
&emsp; 1）同样为了容忍n台节点的故障，第一种方案需要2n+1个副本，而第二种方案只需要n+1个副本，而Kafka的每个分区都有大量的数据，第一种方案会造成大量数据的冗余。  
&emsp; 2）虽然第二种方案的网络延迟会比较高，但网络延迟对Kafka的影响较小。  

（2）ISR  
&emsp; 采用第二种方案之后，设想以下情景：leader收到数据，所有follower都开始同步数据，但有一个follower，因为某种故障，迟迟不能与leader进行同步，那leader就要一直等下去，直到它完成同步，才能发送ack。这个问题怎么解决呢？  
&emsp; Leader维护了一个动态的in-sync replica set (ISR)，意为和leader保持同步的follower集合。当ISR中的follower完成数据的同步之后，leader就会给follower发送ack。如果follower长时间未向leader同步数据，则该follower将被踢出ISR，该时间阈值由replica.lag.time.max.ms参数设定。Leader发生故障之后，就会从ISR中选举新的leader。  

（3）ack应答机制  
&emsp; 对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等ISR中的follower全部接收成功。  
&emsp; 所以Kafka为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡，选择以下的配置。  
**`acks参数配置`**：  
&emsp; acks：  
&emsp; 0：producer不等待broker的ack，这一操作提供了一个最低的延迟，broker一接收到还没有写入磁盘就已经返回，当broker故障时**有可能丢失数据**；  
&emsp; 1：producer等待broker的ack，partition的leader落盘成功后返回ack，如果在follower同步成功之前leader故障，那么**将会丢失数据**；  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/acks%3D1.png"/>  
<p align="center">
</p>
</p>  

&emsp; -1（all）：producer等待broker的ack，partition的leader和follower全部落盘成功后才返回ack。但是如果在follower同步完成后，broker发送ack之前，leader发生故障，那么会**造成数据重复**。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/acks%3D-1.png"/>  
<p align="center">
</p>
</p>  

（4）故障处理细节  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/%E6%95%85%E9%9A%9C%E5%A4%84%E7%90%86%E7%BB%86%E8%8A%82.png"/>  
<p align="center">
</p>
</p>  

① follower故障  
&emsp; follower发生故障后会被临时踢出ISR，待该follower恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步。等该follower的LEO大于等于该Partition的HW，即follower追上leader之后，就可以重新加入ISR了。  
② leader故障  
&emsp; leader发生故障之后，会从ISR中选出一个新的leader，之后，为保证多个副本之间的数据一致性，其余的follower会先将各自的log文件高于HW的部分截掉，然后从新的leader同步数据。  
&emsp; **注意**：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。  

### 3、Kafka消费者
1）消费方式  
&emsp; **consumer采用pull（拉）模式从broker中读取数据**。  
&emsp; **push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的**。它的目标是尽可能以最快速度传递消息，但是这样很容易造成consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而pull模式则可以根据consumer的消费能力以适当的速率消费消息。  
&emsp; **pull模式不足之处是，如果kafka没有数据，消费者可能会陷入循环中，一直返回空数据**。针对这一点，Kafka的消费者在消费数据时会传入一个时长参数timeout，如果当前没有数据可供消费，consumer会等待一段时间之后再返回，这段时长即为timeout。  
2）分区分配策略  
&emsp; 一个consumer group中有多个consumer，一个 topic有多个partition，所以必然会涉及到partition的分配问题，即确定那个partition由哪个consumer来消费。  
&emsp; Kafka有两种分配策略：**RoundRobin和Range**。  
（1）**`RoundRobin`**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/RoundRobin.png"/>  
<p align="center">
</p>
</p>  

（2）**`Range`**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/Range.png"/>  
<p align="center">
</p>
</p>  

3）offset维护  
&emsp; 由于consumer在消费过程中可能会出现断电宕机等故障，consumer恢复后，需要从故障前的位置的继续消费，所以consumer需要实时记录自己消费到了哪个offset，以便故障恢复后继续消费。  
&emsp; **Kafka 0.9版本之前，consumer默认将offset保存在Zookeeper中，从0.9版本开始，consumer默认将offset保存在Kafka一个内置的topic中，该topic为__consumer_offsets**。  

### 4、Kafka高效读写数据
1）顺序写磁盘  
&emsp; Kafka的producer生产数据，要写入到log文件中，写的过程是一直追加到文件末端，为顺序写。官网有数据表明，**同样的磁盘，顺序写能到到600M/s，而随机写只有100k/s**。这与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。  
2）零复制技术  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/%E9%9B%B6%E5%A4%8D%E5%88%B6%E6%8A%80%E6%9C%AF.jpg"/>  
<p align="center">
</p>
</p>  

### 5、Zookeeper在Kafka中的作用
&emsp; Kafka集群中有一个broker会被选举为Controller，负责管理集群broker的上下线，所有topic的分区副本分配和leader选举等工作。  
&emsp; Controller的管理工作都是依赖于Zookeeper的。  
&emsp; 以下为partition的leader选举过程：  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Kafka%E6%96%87%E6%A1%A3Pics/Kafka%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90/partition%E7%9A%84leader%E9%80%89%E4%B8%BE%E8%BF%87%E7%A8%8B.jpg"/>  
<p align="center">
</p>
</p>  

















