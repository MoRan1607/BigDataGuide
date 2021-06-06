Dstream进阶
---  
### 一、DStream转换  
&emsp; DStream上的操作与RDD的类似，分为Transformations（转换）和Output Operations（输出）两种，此外转换操作中还有一些比较特殊的算子，如：updateStateByKey()、transform()以及各种Window相关的算子。   

### 1、无状态转换操作  
&emsp; 无状态转化操作就是把简单的RDD转化操作应用到每个批次上，也就是转化DStream中的每一个RDD。部分无状态转化操作列在了下表中   
&emsp; 需要记住的是，尽管这些函数看起来像作用在整个流上一样，但事实上每个DStream在内部是由许多RDD（批次）组成，且无状态转化操作是分别应用到每个RDD上的。  
&emsp; **例如：reduceByKey()会归约每个时间区间中的数据，但不会归约不同区间之间的数据。在之前的wordcount程序中，我们只会统计几秒内接收到的数据的单词个数，而不会累加。**  
#### 1）Transform  
&emsp; Transform允许DStream上执行任意的RDD-to-RDD函数。即使这些函数并没有在DStream的API中暴露出来，通过该函数可以方便的扩展Spark API。该函数每一批次调度一次。其实也就是对DStream中的RDD应用转换。   
```scala
object Spark06_Nostate_Transform {
  def main(args: Array[String]): Unit = {
    //创建SparkConf
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")

    //创建StreamingContext
    val ssc = new StreamingContext(conf, Seconds(3))

    //创建DStream
    val lineDStream: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop202", 9999)

    //转换为RDD操作
    val wordAndCountDStream: DStream[(String, Int)] = lineDStream.transform(rdd => {
    val words: RDD[String] = rdd.flatMap(_.split(" "))
    val wordAndOne: RDD[(String, Int)] = words.map((_, 1))
    val value: RDD[(String, Int)] = wordAndOne.reduceByKey(_ + _)
      value
    })

    //打印
    wordAndCountDStream.print

    //启动
    ssc.start()
    ssc.awaitTermination()
  }
}
```  

### 2、有状态转化操作   
#### 1）UpdateStateByKey  
&emsp; UpdateStateByKey算子用于将历史结果应用到当前批次，该操作允许在使用新信息不断更新状态的同时能够保留他的状态。   
&emsp; 有时，我们需要在DStream中跨批次维护状态(例如流计算中累加wordcount)。针对这种情况，updateStateByKey()为我们提供了对一个状态变量的访问，用于键值对形式的DStream。给定一个由(键，事件)对构成的DStream，并传递一个指定如何根据新的事件更新每个键对应状态的函数，它可以构建出一个新的DStream，其内部数据为(键，状态) 对。   
&emsp; UpdateStateByKey() 的结果会是一个新的DStream，其内部的RDD 序列是由每个时间区间对应的(键，状态)对组成的。  
&emsp; 为使用这个功能，需要做下面两步：  
&emsp; &emsp; 1.定义状态，状态可以是一个任意的数据类型。  
&emsp; &emsp; 2.定义状态更新函数，用此函数阐明如何使用之前的状态和来自输入流的新值对状态进行更新。  
&emsp; 使用updateStateByKey需要对检查点目录进行配置，会使用检查点来保存状态。   
&emsp; 更新版的wordcount  
**UpdateStateByKey数据流解析**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Streaming/1/%E5%9B%BE%E7%89%873.png"/>  
<p align="center">
</p>
</p>   

例如：每隔一段时间景点人流量变化（从程序启动开始，在原有递增）  

1）编写代码   
```scala
object Spark07_State_updateStateByKey {
  def main(args: Array[String]): Unit = {
    //创建SparkConf
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")

    //创建StreamingContext
    val ssc = new StreamingContext(conf, Seconds(3))

    //设置检查点路径  用于保存状态
    ssc.checkpoint("D:\\dev\\workspace\\my-bak\\spark-bak\\cp")

    //创建DStream
    val lineDStream: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop202", 9999)

    //扁平映射
    val flatMapDS: DStream[String] = lineDStream.flatMap(_.split(" "))

    //结构转换
    val mapDS: DStream[(String, Int)] = flatMapDS.map((_,1))

    //聚合
    // 注意：DStreasm中reduceByKey只能对当前采集周期（窗口）进行聚合操作，没有状态
    //val reduceDS: DStream[(String, Int)] = mapDS.reduceByKey(_+_)
    val stateDS: DStream[(String, Int)] = mapDS.updateStateByKey(
      (seq: Seq[Int], state: Option[Int]) => {
        Option(seq.sum + state.getOrElse(0))
      }
    )

	//打印输出
    stateDS.print()

    //启动
    ssc.start()
    ssc.awaitTermination()
  }
}
```   

2）启动程序并向9999端口发送数据  
```
nc -lk 9999
```  

3）查看结果为累加   

#### 2）Window Operations(窗口操作)   
&emsp; Spark Streaming 也提供了窗口计算, 允许执行转换操作作用在一个窗口内的数据。默认情况下, 计算只对一个时间段内的RDD进行, 有了窗口之后, 可以把计算应用到一个指定的窗口内的所有RDD上。一个窗口可以包含多个时间段，基于窗口的操作会在一个比StreamingContext的批次间隔更长的时间范围内，通过整合多个批次的结果，计算出整个窗口的结果。所有基于窗口的操作都需要两个参数，分别为**窗口时长**以及**滑动步长**。  
&emsp; **窗口时长：计算内容的时间范围；**    
&emsp; **滑动步长：隔多久触发一次计算。**  
&emsp; **注意：这两者都必须为采集周期的整数倍。**  

&emsp; 如下图所示WordCount案例：窗口大小为批次的2倍，滑动步等于批次大小。   
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20Streaming/1/%E5%9B%BE%E7%89%873.png"/>  
<p align="center">
</p>
</p>   

**例如：一小时人流量的变化，窗口（6秒）和间隔（3秒）不一致，不一定从程序启动开始**   
**需求：WordCount统计 3秒一个批次，窗口6秒，滑步3秒。**  
```scala
object Spark08_State_window {
  def main(args: Array[String]): Unit = {
    //创建SparkConf
    val sparkConf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")

    //创建StreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(3))

    //设置检查点路径  用于保存状态
    ssc.checkpoint("D:\\dev\\workspace\\my-bak\\spark-bak\\cp")

    //创建DStream
    val lineDStream: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop202", 9999)

    //扁平映射
    val flatMapDS: DStream[String] = lineDStream.flatMap(_.split(" "))

    //设置窗口大小，滑动的步长
    val windowDS: DStream[String] = flatMapDS.window(Seconds(6),Seconds(3))

    //结构转换
    val mapDS: DStream[(String, Int)] = windowDS.map((_,1))

    //聚合
   val reduceDS: DStream[(String, Int)] = mapDS.reduceByKey(_+_)
    reduceDS.print()

    //启动
    ssc.start()
    ssc.awaitTermination()
  }
}
```    

#### 2）关于Window的操作还有如下方法   
**1)window(windowLength, slideInterval)**   
&emsp; 基于对源DStream窗化的批次进行计算返回一个新的Dstream    
**2)countByWindow(windowLength, slideInterval)**   
&emsp; 返回一个滑动窗口计数流中的元素个数  
**3)countByValueAndWindow()**    
&emsp; 返回的DStream则包含窗口中每个值的个数  
**4)reduceByWindow(func, windowLength, slideInterval)**   
&emsp; 通过使用自定义函数整合滑动区间流元素来创建一个新的单元素流   
**5)reduceByKeyAndWindow(func, windowLength, slideInterval, [numTasks])**   
&emsp; 当在一个(K,V)对的DStream上调用此函数，会返回一个新(K,V)对的DStream，此处通过对滑动窗口中批次数据使用reduce函数来整合每个key的value值  
**6)reduceByKeyAndWindow(func, invFunc, windowLength, slideInterval, [numTasks])**  
&emsp; 这个函数是上述函数的变化版本，每个窗口的reduce值都是通过用前一个窗的reduce值来递增计算。通过reduce进入到滑动窗口数据并”反向reduce”离开窗口的旧数据来实现这个操作。如果把3秒的时间窗口当成一个池塘，池塘每一秒都会有鱼游进或者游出，那么第一个函数表示每由进来一条鱼，就在该类鱼的数量上累加。而第二个函数是，每由出去一条鱼，就将该鱼的总数减去一。  

### 二、DStream输出     
&emsp; 输出操作指定了对流数据经转化操作得到的数据所要执行的操作(例如把结果推入外部数据库或输出到屏幕上)。与RDD中的惰性求值类似，如果一个DStream及其派生出的DStream都没有被执行输出操作，那么这些DStream就都不会被求值。如果StreamingContext中没有设定输出操作，整个context就都不会启动。   

### 1、常用输出操作   
**1)print()**  
&emsp; 在运行流程序的驱动结点上打印DStream中每一批次数据的最开始10个元素。这用于开发和调试。在Python API中，同样的操作叫print()。  
**2)saveAsTextFiles(prefix, [suffix])**  
&emsp; 以text文件形式存储这个DStream的内容。每一批次的存储文件名基于参数中的prefix和suffix。”prefix-Time_IN_MS[.suffix]”。  
**3)saveAsObjectFiles(prefix, [suffix])**   
&emsp; 以Java对象序列化的方式将Stream中的数据保存为SequenceFiles。每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]". Python中目前不可用。   
**4)saveAsHadoopFiles(prefix, [suffix])**   
&emsp; 将Stream中的数据保存为 Hadoop files。每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]"。Python API中目前不可用。   
**5)foreachRDD(func)**    
&emsp; 这是最通用的输出操作，即将函数func用于产生于stream的每一个RDD。其中参数传入的函数func应该实现将每一个RDD中数据推送到外部系统，如将RDD存入文件或者通过网络将其写入数据库。   
&emsp; 通用的输出操作foreachRDD()，它用来对DStream中的RDD运行任意计算。这和transform()有些类似，都可以让我们访问任意RDD。在foreachRDD()中，可以重用我们在Spark中实现的所有行动操作。比如，常见的用例之一是把数据写到诸如MySQL的外部数据库中，但是在使用的时候需要注意以下几点   
&emsp; 连接不能写在driver层面（序列化）；  
&emsp; 如果写在foreach则每个RDD中的每一条数据都创建，得不偿失；   
&emsp; 增加foreachPartition，在分区创建（获取）。   

### 三、累加器和广播变量    
&emsp; 和RDD中的累加器和广播变量的用法完全一样，RDD中怎么用, 这里就怎么用   
### 1、DataFrame and SQL Operations  
&emsp; 你可以很容易地在流数据上使用DataFrames和SQL，你必须使用SparkContext来创建StreamingContext要用的SQLContext。此外，这一过程可以在驱动失效后重启。我们通过创建一个实例化的SQLContext单实例来实现这个工作。   
&emsp; 如下例所示，我们对前例WordCount进行修改从而使用DataFrames和SQL来实现。每个RDD被转换为DataFrame，以临时表格配置并用SQL进行查询。   
```scala
val spark = SparkSession.builder.config(conf).getOrCreate()
import spark.implicits._
mapDS.foreachRDD(rdd =>{
  val df: DataFrame = rdd.toDF("word", "count")
  df.createOrReplaceTempView("words")
  spark.sql("select * from words").show
})
```   

### 2、Caching/Persistence   
&emsp; 和RDDs类似，DStreams同样允许开发者将流数据保存在内存中。也就是说，在DStream上使用persist()方法将会自动把DStreams中的每个RDD保存在内存中。    
&emsp; 当DStream中的数据要被多次计算时，这个非常有用（如在同样数据上的多次操作）。对于像reduceByWindow和reduceByKeyAndWindow以及基于状态的(updateStateByKey)这种操作，保存是隐含默认的。因此，即使开发者没有调用persist()，由基于窗操作产生的DStreams也会自动保存在内存中。    
