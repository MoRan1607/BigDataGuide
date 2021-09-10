# Flink流处理API

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E6%B5%81%E5%A4%84%E7%90%86API/%E5%9B%BE%E7%89%871.png" alt="img;" />

### 一、Environment

#### 1.1 getExecutionEnvironment

创建一个执行环境，表示当前执行程序的上下文。 如果程序是独立调用的，则此方法返回本地执行环境；如果从命令行客户端调用程序以提交到集群，则此方法返回此集群的执行环境，也就是说，getExecutionEnvironment会根据查询运行的方式决定返回什么样的运行环境，是最常用的一种创建执行环境的方式。

```scala
val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment
val env = StreamExecutionEnvironment.getExecutionEnvironment
```

如果没有设置并行度，会以flink-conf.yaml中的配置为准，默认是1。

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E6%B5%81%E5%A4%84%E7%90%86API/%E5%9B%BE%E7%89%872.png" alt="img;" />

#### 1.2 createLocalEnvironment

返回本地执行环境，需要在调用时指定默认的并行度。

```scala
val env = StreamExecutionEnvironment.createLocalEnvironment(1)
```

#### 1.3 createRemoteEnvironment

返回集群执行环境，将Jar提交到远程服务器。需要在调用时指定JobManager的IP和端口号，并指定要在集群中运行的Jar包。

```scala
val env = ExecutionEnvironment.createRemoteEnvironment("jobmanage-hostname",6123,"YOURPATH//wordcount.jar")
```

### 二、Source

#### 2.1 从集合读取数据

```scala
// 定义样例类，传感器id，时间戳，温度
case class SensorReading(id: String, timestamp: Long, temperature: Double)

object Sensor {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream1 = env
      .fromCollection(List(
        SensorReading("sensor_1", 1547718199, 35.8),
        SensorReading("sensor_6", 1547718201, 15.4),
        SensorReading("sensor_7", 1547718202, 6.7),
        SensorReading("sensor_10", 1547718205, 38.1)
      ))

    stream1.print("stream1:").setParallelism(1)

    env.execute()
  }
}
```

#### 2.2 从文件读取数据

```scala
val stream2 = env.readTextFile("YOUR_FILE_PATH")
```

#### 2.3 以kafka消息队列的数据作为来源

需要引入kafka连接器的依赖：

`pom.xml`

```xml
<!-- https://mvnrepository.com/artifact/org.apache.flink/flink-connector-kafka-0.11 -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.11_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
```

具体代码如下：

```scala
val properties = new Properties()
properties.setProperty("bootstrap.servers", "localhost:9092")
properties.setProperty("group.id", "consumer-group")
properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
properties.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
properties.setProperty("auto.offset.reset", "latest")

val stream3 = env.addSource(new FlinkKafkaConsumer011[String]("sensor", new SimpleStringSchema(), properties))
```

#### 2.4 自定义Source

除了以上的source数据来源，我们还可以自定义source。需要做的，只是传入一个SourceFunction就可以。具体调用如下：

```scala
val stream4 = env.addSource( new MySensorSource() )
```

我们希望可以随机生成传感器数据，MySensorSource具体的代码实现如下：

```scala
class MySensorSource extends SourceFunction[SensorReading]{

	// flag: 表示数据源是否还在正常运行
	var running: Boolean = true

	override def cancel(): Unit = {
		running = false
	}

	override def run(ctx: SourceFunction.SourceContext[SensorReading]): Unit = {
		// 初始化一个随机数发生器
		val rand = new Random()

		var curTemp = 1.to(10).map(
			i => ( "sensor_" + i, 65 + rand.nextGaussian() * 20 )
		)

		while(running){
			// 更新温度值
			curTemp = curTemp.map(
				t => (t._1, t._2 + rand.nextGaussian() )
			)
			// 获取当前时间戳
			val curTime = System.currentTimeMillis()

			curTemp.foreach(
				t => ctx.collect(SensorReading(t._1, curTime, t._2))
			)
			Thread.sleep(100)
		}
	}
}
```

### 三、Transform

**`转换算子`**

#### 3.1 map

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E6%B5%81%E5%A4%84%E7%90%86API/%E5%9B%BE%E7%89%873.png" alt="img;" />

```scala
val streamMap = stream.map { x => x * 2 }
```

#### 3.2 flatMap

flatMap的函数签名：def flatMap[A,B](as: List[A])(f: A ⇒ List[B]): List[B]
例如：flatMap(List(1,2,3))(i ⇒ List(i,i))
结果是List(1,1,2,2,3,3), 
而List("a b", "c d").flatMap(line ⇒ line.split(" "))
结果是List(a, b, c, d)。

```scala
val streamFlatMap = stream.flatMap{
    x => x.split(" ")
}
```

#### 3.3 Filter

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E6%B5%81%E5%A4%84%E7%90%86API/%E5%9B%BE%E7%89%874.png" alt="img;" />

```scala
val streamFilter = stream.filter{
    x => x == 1
}
```

#### 3.4 KeyBy

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E6%B5%81%E5%A4%84%E7%90%86API/%E5%9B%BE%E7%89%875.png" alt="img;" />

**DataStream** → **KeyedStream**：逻辑地将一个流拆分成不相交的分区，每个分区包含具有相同key的元素，在内部以hash的形式实现的。

#### 3.5 滚动聚合算子（Rolling Aggregation）

这些算子可以针对KeyedStream的每一个支流做聚合。

- sum()
- min()
- max()
- minBy()
- maxBy()

#### 3.6 Reduce

**KeyedStream** → **DataStream**：一个分组数据流的聚合操作，合并当前的元素和上次聚合的结果，产生一个新的值，返回的流中包含每一次聚合的结果，而不是只返回最后一次聚合的最终结果。

```scala
val stream2 = env.readTextFile("YOUR_PATH\\sensor.txt")
  .map( data => {
    val dataArray = data.split(",")
    SensorReading(dataArray(0).trim, dataArray(1).trim.toLong, dataArray(2).trim.toDouble)
  })
  .keyBy("id")
  .reduce( (x, y) => SensorReading(x.id, x.timestamp + 1, y.temperature) )
```

#### 3.7 Split 和 Select

`Split`

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E6%B5%81%E5%A4%84%E7%90%86API/%E5%9B%BE%E7%89%876.png" alt="img;" />

**DataStream** → **SplitStream**：根据某些特征把一个DataStream拆分成两个或者多个DataStream。

`Select`

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E6%B5%81%E5%A4%84%E7%90%86API/%E5%9B%BE%E7%89%877.png" alt="img;" />

**SplitStream**→**DataStream**：从一个SplitStream中获取一个或者多个DataStream。

需求：传感器数据按照温度高低（以30度为界），拆分成两个流。

```scala
val splitStream = stream2
  .split( sensorData => {
    if (sensorData.temperature > 30) Seq("high") else Seq("low")
  } )

val high = splitStream.select("high")
val low = splitStream.select("low")
val all = splitStream.select("high", "low")
```

#### 3.8 Connect和 CoMap

`Connnect`

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E6%B5%81%E5%A4%84%E7%90%86API/%E5%9B%BE%E7%89%878.png" alt="img;" />

**DataStream,DataStream** → **ConnectedStreams**：连接两个保持他们类型的数据流，两个数据流被Connect之后，只是被放在了一个同一个流中，内部依然保持各自的数据和形式不发生任何变化，两个流相互独立。
`CoMap,CoFlatMap`

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E6%B5%81%E5%A4%84%E7%90%86API/%E5%9B%BE%E7%89%879.png" alt="img;" />

**ConnectedStreams** → **DataStream**：作用于ConnectedStreams上，功能与map和flatMap一样，对ConnectedStreams中的每一个Stream分别进行map和flatMap处理。

```scala
val warning = high.map( sensorData => (sensorData.id, sensorData.temperature) )
val connected = warning.connect(low)

val coMap = connected.map(
    warningData => (warningData._1, warningData._2, "warning"),
    lowData => (lowData.id, "healthy")
)
```

#### 3.9 Union

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E6%B5%81%E5%A4%84%E7%90%86API/%E5%9B%BE%E7%89%8710.png" alt="img;" />

DataStream → DataStream：对两个或者两个以上的DataStream进行union操作，产生一个包含所有DataStream元素的新DataStream。

```scala
//合并以后打印
val unionStream: DataStream[StartUpLog] = appStoreStream.union(otherStream)
unionStream.print("union:::")
```

**Connect与 Union 区别：**

1. Union之前两个流的类型必须是一样，Connect可以不一样，在之后的coMap中再去调整成为一样的。
2. Connect只能操作两个流，Union可以操作多个。

### 四、支持的数据类型

Flink流应用程序处理的是以数据对象表示的事件流。所以在Flink内部，我们需要能够处理这些对象。它们需要被序列化和反序列化，以便通过网络传送它们；或者从状态后端、检查点和保存点读取它们。为了有效地做到这一点，Flink需要明确知道应用程序所处理的数据类型。Flink使用类型信息的概念来表示数据类型，并为每个数据类型生成特定的序列化器、反序列化器和比较器。

Flink还具有一个类型提取系统，该系统分析函数的输入和返回类型，以自动获取类型信息，从而获得序列化器和反序列化器。但是，在某些情况下，例如lambda函数或泛型类型，需要显式地提供类型信息，才能使应用程序正常工作或提高其性能。

Flink支持Java和Scala中所有常见数据类型。使用最广泛的类型有以下几种。

#### 4.1 基础数据类型

Flink支持所有的Java和Scala基础数据类型，Int, Double, Long, String, …

```scala
val numbers: DataStream[Long] = env.fromElements(1L, 2L, 3L, 4L)
numbers.map( n => n + 1 )
```

#### 4.2 Java和Scala元组（Tuples）

```scala
val persons: DataStream[(String, Integer)] = env.fromElements( 
	("Adam", 17), 
	("Sarah", 23) ) 
persons.filter(p => p._2 > 18)
```

#### 4.3 Scala样例类（case classes）

```scala
case class Person(name: String, age: Int) 
val persons: DataStream[Person] = env.fromElements(
	Person("Adam", 17), 
	Person("Sarah", 23) )
persons.filter(p => p.age > 18)
```

#### 4.4 Java简单对象（POJOs）

```scala
public class Person {
	public String name;
	public int age;
	public Person() {}
	public Person(String name, int age) { 
		this.name = name;      
		this.age = age;  
	}
}

DataStream<Person> persons = env.fromElements(   
	new Person("Alex", 42),   
	new Person("Wendy", 23));
```

#### 4.5 其它（Arrays, Lists, Maps, Enums, 等等）

Flink对Java和Scala中的一些特殊目的的类型也都是支持的，比如Java的ArrayList，HashMap，Enum等等。

### 五、实现UDF函数——更细粒度的控制流

#### 5.1 函数类（Function Classes）

Flink暴露了所有udf函数的接口(实现方式为接口或者抽象类)。例如MapFunction, FilterFunction, ProcessFunction等等。

下面例子实现了FilterFunction接口：

```scala
class FilterFilter extends FilterFunction[String] {
      override def filter(value: String): Boolean = {
        value.contains("flink")
      }
}
val flinkTweets = tweets.filter(new FlinkFilter)
```

还可以将函数实现成匿名类

```scala
val flinkTweets = tweets.filter(
	new RichFilterFunction[String] {
		override def filter(value: String): Boolean = {
			value.contains("flink")
		}
	}
)
```

我们filter的字符串"flink"还可以当作参数传进去。

```scala
val tweets: DataStream[String] = ...
val flinkTweets = tweets.filter(new KeywordFilter("flink"))

class KeywordFilter(keyWord: String) extends FilterFunction[String] {
	override def filter(value: String): Boolean = {
		value.contains(keyWord)
	}
}
```

#### 5.2 匿名函数（Lambda Functions）

```scala
val tweets: DataStream[String] = ...
val flinkTweets = tweets.filter(_.contains("flink"))
```

#### 5.3 富函数（Rich Functions）

“富函数”是DataStream API提供的一个函数类的接口，所有Flink函数类都有其Rich版本。它与常规函数的不同在于，可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现更复杂的功能。

- RichMapFunction

- RichFlatMapFunction

- RichFilterFunction

- …

Rich Function有一个生命周期的概念。典型的生命周期方法有：

- open()方法是rich function的初始化方法，当一个算子例如map或者filter被调用之前open()会被调用。

- close()方法是生命周期中的最后一个调用的方法，做一些清理工作。

- getRuntimeContext()方法提供了函数的RuntimeContext的一些信息，例如函数执行的并行度，任务的名字，以及state状态。

```scala
class MyFlatMap extends RichFlatMapFunction[Int, (Int, Int)] {
	var subTaskIndex = 0

	override def open(configuration: Configuration): Unit = {
		subTaskIndex = getRuntimeContext.getIndexOfThisSubtask
		// 以下可以做一些初始化工作，例如建立一个和HDFS的连接
	}

	override def flatMap(in: Int, out: Collector[(Int, Int)]): Unit = {
		if (in % 2 == subTaskIndex) {
			out.collect((subTaskIndex, in))
		}
	}

	override def close(): Unit = {
		// 以下做一些清理工作，例如断开和HDFS的连接。
	}
}
```

### 六、Sink

Flink没有类似于spark中foreach方法，让用户进行迭代的操作。虽有对外的输出操作都要利用Sink完成。最后通过类似如下方式完成整个任务最终输出操作。

```scala
stream.addSink(new MySink(xxxx)) 
```

官方提供了一部分的框架的sink。除此以外，需要用户自定义实现sink。

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E6%B5%81%E5%A4%84%E7%90%86API/%E5%9B%BE%E7%89%8711.png" alt="img;" />

#### 6.1 Kafka

`pom.xml`

```xml
<!-- https://mvnrepository.com/artifact/org.apache.flink/flink-connector-kafka-0.11 -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.11_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
```

主函数中添加sink：

```scala
val union = high.union(low).map(_.temperature.toString)

union.addSink(new FlinkKafkaProducer011[String]("localhost:9092", "test", new SimpleStringSchema()))
```

#### 6.2 Redis

`pom.xml`

```xml
<!-- https://mvnrepository.com/artifact/org.apache.bahir/flink-connector-redis -->
<dependency>
    <groupId>org.apache.bahir</groupId>
    <artifactId>flink-connector-redis_2.11</artifactId>
    <version>1.0</version>
</dependency>
```

定义一个redis的mapper类，用于定义保存到redis时调用的命令：

```scala
class MyRedisMapper extends RedisMapper[SensorReading]{
  override def getCommandDescription: RedisCommandDescription = {
    new RedisCommandDescription(RedisCommand.HSET, "sensor_temperature")
  }
  override def getValueFromData(t: SensorReading): String = t.temperature.toString

  override def getKeyFromData(t: SensorReading): String = t.id
}
```

在主函数中调用：

```scala
val conf = new FlinkJedisPoolConfig.Builder().setHost("localhost").setPort(6379).build()
dataStream.addSink( new RedisSink[SensorReading](conf, new MyRedisMapper) )
```

#### 6.3 Elasticsearch

`pom.xml`

```scala
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-elasticsearch6_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
```

在主函数中调用：

```scala
val httpHosts = new util.ArrayList[HttpHost]()
httpHosts.add(new HttpHost("localhost", 9200))

val esSinkBuilder = new ElasticsearchSink.Builder[SensorReading]( httpHosts, new ElasticsearchSinkFunction[SensorReading] {
  override def process(t: SensorReading, runtimeContext: RuntimeContext, requestIndexer: RequestIndexer): Unit = {
    println("saving data: " + t)
    val json = new util.HashMap[String, String]()
    json.put("data", t.toString)
    val indexRequest = Requests.indexRequest().index("sensor").`type`("readingData").source(json)
    requestIndexer.add(indexRequest)
    println("saved successfully")
  }
} )
dataStream.addSink( esSinkBuilder.build() )
```

#### 6.4 JDBC 自定义sink

`pom.xml`

```scala
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.44</version>
</dependency>
```

添加MyJdbcSink

```scala
class MyJdbcSink() extends RichSinkFunction[SensorReading]{
  var conn: Connection = _
  var insertStmt: PreparedStatement = _
  var updateStmt: PreparedStatement = _

  // open 主要是创建连接
  override def open(parameters: Configuration): Unit = {
    super.open(parameters)

    conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "123456")
    insertStmt = conn.prepareStatement("INSERT INTO temperatures (sensor, temp) VALUES (?, ?)")
    updateStmt = conn.prepareStatement("UPDATE temperatures SET temp = ? WHERE sensor = ?")
  }
  // 调用连接，执行sql
  override def invoke(value: SensorReading, context: SinkFunction.Context[_]): Unit = {
    
updateStmt.setDouble(1, value.temperature)
    updateStmt.setString(2, value.id)
    updateStmt.execute()

    if (updateStmt.getUpdateCount == 0) {
      insertStmt.setString(1, value.id)
      insertStmt.setDouble(2, value.temperature)
      insertStmt.execute()
    }
  }

  override def close(): Unit = {
    insertStmt.close()
    updateStmt.close()
    conn.close()
  }
}
```

在main方法中增加，把明细保存到mysql中

```scala
dataStream.addSink(new MyJdbcSink())
```

