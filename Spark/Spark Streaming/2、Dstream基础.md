Dstream基础  
---  
### 一、Dstream入门  
### 1、WordCount案例实操  
1）需求：使用netcat工具向9999端口不断的发送数据，通过SparkStreaming读取端口数据并统计不同单词出现的次数  
2）添加依赖  
```xml
<dependency>
	<groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.11</artifactId>
    <version>2.1.1</version>
</dependency>
```  
3）编写代码  
```scala
object Spark01_WordCount {
def main(args: Array[String]): Unit = {
	//创建配置文件对象 注意：Streaming程序至少不能设置为local，至少需要2个线程
    val conf: SparkConf = new SparkConf().setAppName("Spark01_W").setMaster("local[*]")

    //创建Spark Streaming上下文环境对象
    val ssc = new StreamingContext(conf,Seconds(3))

    //操作数据源-从端口中获取一行数据
    val socketDS: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop202",9999)

    //对获取的一行数据进行扁平化操作
    val flatMapDS: DStream[String] = socketDS.flatMap(_.split(" "))

    //结构转换
    val mapDS: DStream[(String, Int)] = flatMapDS.map((_,1))

    //对数据进行聚合
    val reduceDS: DStream[(String, Int)] = mapDS.reduceByKey(_+_)

    //输出结果   注意：调用的是DS的print函数
    reduceDS.print()

    //启动采集器
    ssc.start()

    //默认情况下，上下文对象不能关闭
    //ssc.stop()

    //等待采集结束，终止上下文环境对象
    ssc.awaitTermination()
  }
}
```  

4）启动程序并通过NetCat发送数据：  
```
nc -lk 9999
```  
&emsp; 注意：如果程序运行时，log日志太多，可以将spark conf目录下的log4j文件里面的日志级别改成WARN。  

### 2、WordCount解析  
&emsp; Discretized Stream是Spark Streaming的基础抽象，代表持续性的数据流和经过各种Spark算子操作后的结果数据流。在内部实现上，DStream是一系列连续的RDD来表示,每个RDD含有一段时间间隔内的数据，对这些 RDD的转换是由Spark引擎来计算的，DStream的操作隐藏的大多数的细节, 然后给开发者提供了方便使用的高级API如下图：    
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Streaming/1/%E5%9B%BE%E7%89%871.png"/>  
<p align="center">
</p>
</p>   

### 3、注意事项   
1）一旦StreamingContext已经启动, 则不能再添加新的streaming computations  
2）一旦一个StreamingContext已经停止(StreamingContext.stop()), 他也不能再重启   
3）在一个JVM内, 同一时间只能启动一个StreamingContext   
4）stop() 的方式停止StreamingContext, 也会把SparkContext停掉. 如果仅仅想停止StreamingContext, 则应该这样: stop(false)
一个SparkContext可以重用去创建多个StreamingContext, 前提是以前的StreamingContext已经停掉,并且SparkContext没有被停掉   

### 二、Dstream创建  
### 2.1 RDD队列  
#### 1、用法及说明  
&emsp; 测试过程中，可以通过使用ssc.queueStream(queueOfRDDs)来创建DStream，每一个推送到这个队列中的RDD，都会作为一个DStream处理。  
#### 2、案例实践  
1）需求：循环创建几个RDD，将RDD放入队列。通过SparkStream创建Dstream，计算WordCount  
2）编写代码  
```scala
object Spark02_DStreamCreate_RDDQueue {
  def main(args: Array[String]): Unit = {
    // 创建Spark配置信息对象
    val conf = new SparkConf().setMaster("local[*]").setAppName("RDDStream")

    // 创建SparkStreamingContext
    val ssc = new StreamingContext(conf, Seconds(3))

    // 创建RDD队列
    val rddQueue = new mutable.Queue[RDD[Int]]()

    // 创建QueueInputDStream
    val inputStream = ssc.queueStream(rddQueue,oneAtATime = false)

    // 处理队列中的RDD数据
    val mappedStream = inputStream.map((_,1))
    val reducedStream = mappedStream.reduceByKey(_ + _)

    // 打印结果
    reducedStream.print()

    // 启动任务
    ssc.start()

    // 循环创建并向RDD队列中放入RDD
    for (i <- 1 to 5) {
      rddQueue += ssc.sparkContext.makeRDD(1 to 5, 10)
      Thread.sleep(2000)
    }
    ssc.awaitTermination()
  }
}
```

3）结果展示   
```
-------------------------------------------
Time: 1582192449000 ms
-------------------------------------------
(1,1)
(2,1)
(3,1)
(4,1)
(5,1)

-------------------------------------------
Time: 1582192452000 ms
-------------------------------------------
(1,2)
(2,2)
(3,2)
(4,2)
(5,2)

-------------------------------------------
Time: 1582192455000 ms
-------------------------------------------
(1,1)
(2,1)
(3,1)
(4,1)
(5,1)
```   

### 2.2 自定义数据源
#### 1、用法及说明  
&emsp; 需要继承Receiver，并实现onStart、onStop方法来自定义数据源采集。   

#### 2、案例实践  
1)需求：自定义数据源，实现监控某个端口号，获取该端口号内容。  
2)自定义数据源  
```scala
class MyReceiver(host: String, port: Int) extends Receiver[String](StorageLevel.MEMORY_ONLY) {
	//创建一个Socket
  private var socket: Socket = _

  //最初启动的时候，调用该方法，作用为：读数据并将数据发送给Spark
  override def onStart(): Unit = {
    new Thread("Socket Receiver") {
      setDaemon(true)
      override def run() { receive() }
    }.start()
  }

  //读数据并将数据发送给Spark
  def receive(): Unit = {
    try {
      socket = new Socket(host, port)
      //创建一个BufferedReader用于读取端口传来的数据
      val reader = new BufferedReader(
        new InputStreamReader(
          socket.getInputStream, StandardCharsets.UTF_8))
      //定义一个变量，用来接收端口传过来的数据
      var input: String = null

      //读取数据 循环发送数据给Spark 一般要想结束发送特定的数据 如："==END=="
      while ((input = reader.readLine())!=null) {
        store(input)
      }
    } catch {
      case e: ConnectException =>
        restart(s"Error connecting to $host:$port", e)
        return
    }
  }

  override def onStop(): Unit = {
    if(socket != null ){
      socket.close()
      socket = null
    }
  }
}
```   

3)使用自定义的数据源采集数据  
```scala
object Spark03_CustomerReceiver {
def main(args: Array[String]): Unit = {
    //1.初始化Spark配置信息
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("StreamWordCount")

    //2.初始化SparkStreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(5))

    //3.创建自定义receiver的Streaming
    val lineStream = ssc.receiverStream(new MyReceiver("hadoop202", 9999))

    //4.将每一行数据做切分，形成一个个单词
    val wordStream = lineStream.flatMap(_.split("\t"))

    //5.将单词映射成元组（word,1）
    val wordAndOneStream = wordStream.map((_, 1))

    //6.将相同的单词次数做统计
    val wordAndCountStream = wordAndOneStream.reduceByKey(_ + _)

    //7.打印
    wordAndCountStream.print()

    //8.启动SparkStreamingContext
    ssc.start()
    ssc.awaitTermination()
  }
}
```   

### 2.3 Kafka数据源（重点）   
#### 1、版本选型   
&emsp; **ReceiverAPI**：需要一个专门的Executor去接收数据，然后发送给其他的Executor做计算。存在的问题，接收数据的Executor和计算的Executor速度会有所不同，特别在接收数据的Executor速度大于计算的Executor速度，会导致计算数据的节点内存溢出。   
&emsp; **DirectAPI**：是由计算的Executor来主动消费Kafka的数据，速度由自身控制。

#### 2、Kafka 0-8 Receive模式  
1）需求：通过SparkStreaming从Kafka读取数据，并将读取过来的数据做简单计算，最终打印到控制台。  
2）导入依赖  
```
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-kafka-0-8_2.11</artifactId>
    <version>2.1.1</version>
</dependency>
```  
 
3）编写代码   
&emsp; 0-8Receive模式，offset维护在zk中，程序停止后，继续生产数据，再次启动程序，仍然可以继续消费。可通过get /consumers/bigdata/offsets/主题名/分区号 查看  
```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream}
import org.apache.spark.streaming.kafka.KafkaUtils
import org.apache.spark.streaming.{Seconds, StreamingContext}

object Spark04_ReceiverAPI {

  def main(args: Array[String]): Unit = {

    //1.创建SparkConf
    val sparkConf: SparkConf = new SparkConf().setAppName("Spark04_ReceiverAPI").setMaster("local[*]")

    //2.创建StreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(3))

    //3.使用ReceiverAPI读取Kafka数据创建DStream
    val kafkaDStream: ReceiverInputDStream[(String, String)] = KafkaUtils.createStream(ssc,
      "hadoop202:2181,hadoop203:2181,hadoop204:2181",
      "bigdata",
      //v表示的主题的分区数
      Map("mybak" -> 2))

    //4.计算WordCount并打印        new KafkaProducer[String,String]().send(new ProducerRecord[]())
    val lineDStream: DStream[String] = kafkaDStream.map(_._2)
    val word: DStream[String] = lineDStream.flatMap(_.split(" "))
    val wordToOneDStream: DStream[(String, Int)] = word.map((_, 1))
    val wordToCountDStream: DStream[(String, Int)] = wordToOneDStream.reduceByKey(_ + _)
    wordToCountDStream.print()

    //5.开启任务
    ssc.start()
    ssc.awaitTermination()
  }
}
```    

#### 3、Kafka 0-8 Direct模式   
1）需求：通过SparkStreaming从Kafka读取数据，并将读取过来的数据做简单计算，最终打印到控制台。  
2）导入依赖   
```
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-kafka-0-8_2.11</artifactId>
    <version>2.1.1</version>
</dependency>
```   

3）编写代码（自动维护offset1）   
&emsp; offset维护在checkpoint中，但是获取StreamingContext的方式需要改变，目前这种方式会丢失消息   
```scala
import kafka.serializer.StringDecoder
import org.apache.kafka.clients.consumer.ConsumerConfig
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.InputDStream
import org.apache.spark.streaming.kafka.KafkaUtils
import org.apache.spark.streaming.{Seconds, StreamingContext}

object Spark05_DirectAPI_Auto01 {

  def main(args: Array[String]): Unit = {

    //1.创建SparkConf
    val sparkConf: SparkConf = new SparkConf().setAppName("Spark05_DirectAPI_Auto01").setMaster("local[*]")

    //2.创建StreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(3))

    ssc.checkpoint("D:\\dev\\workspace\\my-bak\\spark-bak\\cp")

    //3.准备Kafka参数
    val kafkaParams: Map[String, String] = Map[String, String](
      ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> "hadoop202:9092,hadoop203:9092,hadoop204:9092",
      ConsumerConfig.GROUP_ID_CONFIG -> "bigdata"
    )

    //4.使用DirectAPI自动维护offset的方式读取Kafka数据创建DStream
    val kafkaDStream: InputDStream[(String, String)] = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](ssc,
      kafkaParams,
      Set("mybak"))

    //5.计算WordCount并打印
    kafkaDStream.map(_._2)
      .flatMap(_.split(" "))
      .map((_, 1))
      .reduceByKey(_ + _)
      .print()

    //6.开启任务
    ssc.start()
    ssc.awaitTermination()
  }
}
```     

4）编写代码（自动维护offset2）   
&emsp; offset维护在checkpoint中，获取StreamingContext为getActiveOrCreate   
&emsp; 这种方式缺点：   
&emsp; &emsp; checkpoint小文件过多   
&emsp; &emsp; checkpoint记录最后一次时间戳，再次启动的时候会把间隔时间的周期再执行一次    
```scala
import kafka.serializer.StringDecoder
import org.apache.kafka.clients.consumer.ConsumerConfig
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.streaming.dstream.InputDStream
import org.apache.spark.streaming.kafka.KafkaUtils

object Spark06_DirectAPI_Auto02 {

  def main(args: Array[String]): Unit = {
    val ssc: StreamingContext = StreamingContext.getActiveOrCreate("D:\\dev\\workspace\\my-bak\\spark-bak\\cp", () => getStreamingContext)

    ssc.start()
    ssc.awaitTermination()
  }

  def getStreamingContext: StreamingContext = {
    //1.创建SparkConf
    val sparkConf: SparkConf = new SparkConf().setAppName("DirectAPI_Auto01").setMaster("local[*]")
    
    //2.创建StreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(3))
    ssc.checkpoint("D:\\dev\\workspace\\my-bak\\spark-bak\\cp")
    
    //3.准备Kafka参数
    val kafkaParams: Map[String, String] = Map[String, String](
      ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> "hadoop202:9092,hadoop203:9092,hadoop204:9092",
      ConsumerConfig.GROUP_ID_CONFIG -> "bigdata"
    )
    
    //4.使用DirectAPI自动维护offset的方式读取Kafka数据创建DStream
    val kafkaDStream: InputDStream[(String, String)] = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](ssc,
      kafkaParams,
      Set("mybak"))
    
    //5.计算WordCount并打印
    kafkaDStream.map(_._2)
      .flatMap(_.split(" "))
      .map((_, 1))
      .reduceByKey(_ + _)
      .print()
    
    //6.返回结果
    ssc
  }
}
```   

5）编写代码（手动维护offset）   
```scala
import kafka.common.TopicAndPartition
import kafka.message.MessageAndMetadata
import kafka.serializer.StringDecoder
import org.apache.kafka.clients.consumer.ConsumerConfig
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.InputDStream
import org.apache.spark.streaming.kafka.{HasOffsetRanges, KafkaUtils, OffsetRange}
import org.apache.spark.streaming.{Seconds, StreamingContext}

object Spark07_DirectAPI_Handler {

  def main(args: Array[String]): Unit = {

    //1.创建SparkConf
    val conf: SparkConf = new SparkConf().setAppName("DirectAPI_Handler").setMaster("local[*]")

    //2.创建StreamingContext
    val ssc = new StreamingContext(conf, Seconds(3))

    //3.创建Kafka参数
    val kafkaParams: Map[String, String] = Map[String, String](
      ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> "hadoop202:9092,hadoop203:9092,hadoop204:9092",
      ConsumerConfig.GROUP_ID_CONFIG -> "bigdata"
    )

    //4.获取上一次消费的位置信息
    val fromOffsets: Map[TopicAndPartition, Long] = Map[TopicAndPartition, Long](
      TopicAndPartition("mybak", 0) -> 13L,
      TopicAndPartition("mybak", 1) -> 10L
    )

    //5.使用DirectAPI手动维护offset的方式消费数据
    val kafakDStream: InputDStream[String] = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder, String](
      ssc,
      kafkaParams,
      fromOffsets,
      (m: MessageAndMetadata[String, String]) => m.message())

    //6.定义空集合用于存放数据的offset
    var offsetRanges = Array.empty[OffsetRange]

    //7.将当前消费到的offset进行保存
    kafakDStream.transform { rdd =>
      offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
      rdd
    }.foreachRDD { rdd =>
      for (o <- offsetRanges) {
        println(s"${o.fromOffset}-${o.untilOffset}")
      }
    }

    //8.开启任务
    ssc.start()
    ssc.awaitTermination()

  }
}
```    

#### 4、消费Kafka数据模式总结   
**0-8 ReceiverAPI**:  
&emsp; 1)专门的Executor读取数据，速度不统一  
&emsp; 2)跨机器传输数据，WAL  
&emsp; 3)Executor读取数据通过多个线程的方式，想要增加并行度，则需要多个流union  
&emsp; 4)offset存储在Zookeeper中   

**0-8 DirectAPI**:  
&emsp; 1)Executor读取数据并计算  
&emsp; 2)增加Executor个数来增加消费的并行度  
&emsp; 3)offset存储  
&emsp; &emsp; a)CheckPoint(getActiveOrCreate方式创建StreamingContext)  
&emsp; &emsp; b)手动维护(有事务的存储系统)  
&emsp; &emsp; c)获取offset必须在第一个调用的算子中：offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges  
