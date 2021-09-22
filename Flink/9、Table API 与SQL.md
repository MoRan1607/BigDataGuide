# Table API 与SQL

> Table API是流处理和批处理通用的关系型API，Table API可以基于流输入或者批输入来运行而不需要进行任何修改。Table API是SQL语言的超集并专门为Apache Flink设计的，Table API是Scala 和Java语言集成式的API。与常规SQL语言中将查询指定为字符串不同，Table API查询是以Java或Scala中的语言嵌入样式来定义的，具有IDE支持如：自动完成和语法检测。

### 一、需要引入的pom依赖

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-planner_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-api-scala-bridge_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
```

### 二、简单了解TableAPI

```scala
def main(args: Array[String]): Unit = {
  val env = StreamExecutionEnvironment.getExecutionEnvironment
  env.setParallelism(1)

  val inputStream = env.readTextFile("..\\sensor.txt")
  val dataStream = inputStream
    .map( data => {
      val dataArray = data.split(",")
      SensorReading(dataArray(0).trim, dataArray(1).trim.toLong, dataArray(2).trim.toDouble)
    }
    )
  // 基于env创建 tableEnv
val settings: EnvironmentSettings = EnvironmentSettings.newInstance().useOldPlanner().inStreamingMode().build()
val tableEnv: StreamTableEnvironment = StreamTableEnvironment.create(env, settings)

  // 从一条流创建一张表
  val dataTable: Table = tableEnv.fromDataStream(dataStream)

  // 从表里选取特定的数据
  val selectedTable: Table = dataTable.select('id, 'temperature)
    .filter("id = 'sensor_1'")

  val selectedStream: DataStream[(String, Double)] = selectedTable
    .toAppendStream[(String, Double)]

  selectedStream.print()

  env.execute("table test")

}
```

#### 2.1 动态表

如果流中的数据类型是case class可以直接根据case class的结构生成table

```scala
tableEnv.fromDataStream(dataStream)  
```

或者根据字段顺序单独命名

```scala
tableEnv.fromDataStream(dataStream,’id,’timestamp  .......)  
```

最后的动态表可以转换为流进行输出

```scala
table.toAppendStream[(String,String)]
```

#### 2.2 字段

用一个单引放到字段前面来标识字段名, 如 ‘name , ‘id ,’amount 等

### 三、TableAPI 的窗口聚合操作

#### 3.1 通过一个例子了解TableAPI 

```scala
// 统计每10秒中每个传感器温度值的个数
def main(args: Array[String]): Unit = {
  val env = StreamExecutionEnvironment.getExecutionEnvironment
  env.setParallelism(1)
  env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

  val inputStream = env.readTextFile("..\\sensor.txt")
  val dataStream = inputStream
    .map( data => {
      val dataArray = data.split(",")
      SensorReading(dataArray(0).trim, dataArray(1).trim.toLong, dataArray(2).trim.toDouble)
    }
    )
    .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[SensorReading](Time.seconds(1)) {
      override def extractTimestamp(element: SensorReading): Long = element.timestamp * 1000L
    })
  // 基于env创建 tableEnv
val settings: EnvironmentSettings = EnvironmentSettings.newInstance().useOldPlanner().inStreamingMode().build()
val tableEnv: StreamTableEnvironment = StreamTableEnvironment.create(env, settings)

  // 从一条流创建一张表，按照字段去定义，并指定事件时间的时间字段
  val dataTable: Table = tableEnv.fromDataStream(dataStream, 'id, 'temperature, 'ts.rowtime)

  // 按照时间开窗聚合统计
  val resultTable: Table = dataTable
    .window( Tumble over 10.seconds on 'ts as 'tw )
    .groupBy('id, 'tw)
    .select('id, 'id.count)

  val selectedStream: DataStream[(Boolean, (String, Long))] = resultTable
    .toRetractStream[(String, Long)]

  selectedStream.print()

  env.execute("table window test")
}
```

#### 3.2 关于group by

如果了使用 groupby，table转换为流的时候只能用toRetractDstream

```scala
val dataStream: DataStream[(Boolean, (String, Long))] = table.toRetractStream[(String,Long)]
```

toRetractDstream 得到的第一个boolean型字段标识 true就是最新的数据(Insert)，false表示过期老数据(Delete)

```scala
val dataStream: DataStream[(Boolean, (String, Long))] = table.toRetractStream[(String,Long)]
dataStream.filter(_._1).print()
```

如果使用的api包括时间窗口，那么窗口的字段必须出现在groupBy中。

```scala
val resultTable: Table = dataTable
    .window( Tumble over 10.seconds on 'ts as 'tw )
    .groupBy('id, 'tw)
    .select('id, 'id.count)
```

#### 3.3 关于时间窗口

用到时间窗口，必须提前声明时间字段，如果是processTime直接在创建动态表时进行追加就可以。

```scala
val dataTable: Table = tableEnv.fromDataStream(dataStream, 'id, 'temperature, 'ps.proctime)
```

如果是EventTime要在创建动态表时声明

```scala
val dataTable: Table = tableEnv.fromDataStream(dataStream, 'id, 'temperature, 'ts.rowtime)
```

滚动窗口可以使用Tumble over 10000.millis on 来表示

```scala
val resultTable: Table = dataTable
    .window( Tumble over 10.seconds on 'ts as 'tw )
    .groupBy('id, 'tw)
    .select('id, 'id.count)
```

### 四、SQL如何编写

```scala
// 统计每10秒中每个传感器温度值的个数
def main(args: Array[String]): Unit = {
  val env = StreamExecutionEnvironment.getExecutionEnvironment
  env.setParallelism(1)
  env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

  val inputStream = env.readTextFile("..\\sensor.txt")
  val dataStream = inputStream
    .map( data => {
      val dataArray = data.split(",")
      SensorReading(dataArray(0).trim, dataArray(1).trim.toLong, dataArray(2).trim.toDouble)
    }
    )
    .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[SensorReading](Time.seconds(1)) {
      override def extractTimestamp(element: SensorReading): Long = element.timestamp * 1000L
    })
  // 基于env创建 tableEnv
val settings: EnvironmentSettings = EnvironmentSettings.newInstance().useOldPlanner().inStreamingMode().build()
val tableEnv: StreamTableEnvironment = StreamTableEnvironment.create(env, settings)

  // 从一条流创建一张表，按照字段去定义，并指定事件时间的时间字段
  val dataTable: Table = tableEnv.fromDataStream(dataStream, 'id, 'temperature, 'ts.rowtime)

  // 直接写sql完成开窗统计 
  val resultSqlTable: Table = tableEnv.sqlQuery("select id, count(id) from "
  + dataTable + " group by id, tumble(ts, interval '15' second)")

  val selectedStream: DataStream[(Boolean, (String, Long))] = resultSqlTable.toRetractStream[(String, Long)]

  selectedStream.print()

  env.execute("table window test")
}
```

