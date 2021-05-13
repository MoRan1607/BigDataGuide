Spark SQL数据的加载与保存  
---  
### 1、通用的加载与保存方式  
&emsp; 1）**spark.read.load**是加载数据的通用方法  
&emsp; 2）**df.write.save** 是保存数据的通用方法  

#### 1.1 数据加载
&emsp; 1)read直接加载数据  
```scala
scala> spark.read.

csv   format   jdbc   json   load   option   options   orc   parquet   schema   table   text   textFile
```  
&emsp; **注意**：加载数据的相关参数需写到上述方法中，如：textFile需传入加载数据的路径，jdbc需传入JDBC相关参数。    
&emsp; 例如：直接加载Json数据    

```scala
scala> spark.read.json("/opt/module/spark-local/people.json").show
+---+--------+
|age|    name|
+---+--------+
| 18|qiaofeng|
| 19|  duanyu|
| 20|   xuzhu|
```  
&emsp; 2)format指定加载数据类型  
```scala
scala> spark.read.format("…")[.option("…")].load("…")
```  
&emsp; 用法详解：  
&emsp; （1）format("…")：指定加载的数据类型，包括"csv"、"jdbc"、"json"、"orc"、"parquet"和"textFile"  
&emsp; （2）load("…")：在"csv"、"jdbc"、"json"、"orc"、"parquet"和"textFile"格式下需要传入加载数据的路径  
&emsp; （3）option("…")：在"jdbc"格式下需要传入JDBC相应参数，url、user、password和dbtable  
&emsp; 例如：使用format指定加载Json类型数据  
```scala
scala> spark.read.format("json").load ("/opt/module/spark-local/people.json").show
+---+--------+
|age|    name|
+---+--------+
| 18|qiaofeng|
| 19|  duanyu|
| 20|   xuzhu|
```   
&emsp; 3)在文件上直接运行SQL  
&emsp; 前面的是使用read API先把文件加载到DataFrame然后再查询，也可以直接在文件上进行查询。   

```scala
scala>  spark.sql("select * from json.`/opt/module/spark-local/people.json`").show

+---+--------+
|age|    name|
+---+--------+
| 18|qiaofeng|
| 19|  duanyu|
| 20|   xuzhu|
+---+--------+|
```   

&emsp; 说明:json表示文件的格式. 后面的文件具体路径需要用反引号括起来。

#### 1.2 保存数据  
&emsp; 1)write直接保存数据  
```scala
scala> df.write.
csv  jdbc   json  orc   parquet textFile… …
```  
&emsp; 注意：保存数据的相关参数需写到上述方法中。如：textFile需传入加载数据的路径，jdbc需传入JDBC相关参数。    
&emsp; 例如：直接将df中数据保存到指定目录  
```scala
//默认保存格式为parquet
scala> df.write.save("/opt/module/spark-local/output")
//可以指定为保存格式，直接保存，不需要再调用save了
scala> df.write.json("/opt/module/spark-local/output")
```  
&emsp; 2)format指定保存数据类型  
```scala
scala> df.write.format("…")[.option("…")].save("…")
```   
&emsp; 用法详解：  
&emsp; （1）format("…")：指定保存的数据类型，包括"csv"、"jdbc"、"json"、"orc"、"parquet"和"textFile"。   
&emsp; （2）save ("…")：在"csv"、"orc"、"parquet"和"textFile"格式下需要传入保存数据的路径。  
&emsp; （3）option("…")：在"jdbc"格式下需要传入JDBC相应参数，url、user、password和dbtable

&emsp; 3)文件保存选项  
&emsp; 保存操作可以使用 SaveMode, 用来指明如何处理数据，使用mode()方法来设置。有一点很重要: 这些 SaveMode 都是没有加锁的, 也不是原子操作。  
&emsp; SaveMode是一个枚举类，其中的常量包括：  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20SQL/3/2021-05-13_183502.png"/>  
<p align="center">
</p>
</p>  

&emsp; 例如：使用指定format指定保存类型进行保存  
```scala
df.write.mode("append").json("/opt/module/spark-local/output")  
```   

#### 1.3 默认数据源  
&emsp; Spark SQL的默认数据源为Parquet格式。数据源为Parquet文件时，Spark SQL可以方便的执行所有的操作，不需要使用format。修改配置项spark.sql.sources.default，可修改默认数据源格式。  
&emsp; 1)加载数据    
```scala
val df = spark.read.load("/opt/module/spark-local/examples/src/main/resources/users.parquet").show

+------+--------------+----------------+
|  name|favorite_color|favorite_numbers|
+------+--------------+----------------+
|Alyssa|          null|  [3, 9, 15, 20]|
|   Ben|           red|              []|
+------+--------------+----------------+

df: Unit = ()
```   

&emsp; 2)保存数据  
```scala
scala> var df = spark.read.json("/opt/module/spark-local/people.json")
//保存为parquet格式
scala> df.write.mode("append").save("/opt/module/spark-local/output")
```   

### 2、JSON文件  
&emsp; Spark SQL能够自动推测JSON数据集的结构，并将它加载为一个Dataset[Row]。可以通过SparkSession.read.json()去加载一个一个JSON文件。  
&emsp; 注意：这个JSON文件不是一个传统的JSON文件，每一行都得是一个JSON串。格式如下：  
```json
{"name":"Michael"}
{"name":"Andy","age":30}
{"name":"Justin","age":19}
```   
&emsp; 1)导入隐式转换  
```json
import spark.implicits._
```   
&emsp; 2)加载JSON文件  
```json
val path = "/opt/module/spark-local/people.json"
val peopleDF = spark.read.json(path)
```   
&emsp; 3)创建临时表  
```json
peopleDF.createOrReplaceTempView("people")
```   
&emsp; 4)数据查询  
```json
val teenagerNamesDF = spark.sql("SELECT name FROM people WHERE age BETWEEN 13 AND 19")
teenagerNamesDF.show()
+------+
|  name|
+------+
|Justin|
+------+
```   

### 3、MySQL  
&emsp; Spark SQL可以通过JDBC从关系型数据库中读取数据的方式创建DataFrame，通过对DataFrame一系列的计算后，还可以将数据再写回关系型数据库中。  
&emsp; **如果使用spark-shell操作，可在启动shell时指定相关的数据库驱动路径或者将相关的数据库驱动放到spark的类路径下。 **  
```json
bin/spark-shell 
--jars mysql-connector-java-5.1.27-bin.jar
```  
&emsp; 这里演示在Idea中通过JDBC对Mysql进行操作  
#### 3.1 导入依赖  
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.27</version>
</dependency>
```    
#### 3.2 从JDBC读数据  
```scala
object SparkSQL02_Datasource {
  def main(args: Array[String]): Unit = {
    //创建上下文环境配置对象
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL01_Demo")

    //创建SparkSession对象
    val spark: SparkSession = SparkSession.builder().config(conf).getOrCreate()

    import spark.implicits._

    //方式1：通用的load方法读取
    spark.read.format("jdbc")
      .option("url", "jdbc:mysql://hadoop202:3306/test")
      .option("driver", "com.mysql.jdbc.Driver")
      .option("user", "root")
      .option("password", "123456")
      .option("dbtable", "user")
      .load().show

    
    //方式2:通用的load方法读取 参数另一种形式
    spark.read.format("jdbc")
      .options(Map("url"->"jdbc:mysql://hadoop202:3306/test?user=root&password=123456",
        "dbtable"->"user","driver"->"com.mysql.jdbc.Driver")).load().show

    //方式3:使用jdbc方法读取
    val props: Properties = new Properties()
    props.setProperty("user", "root")
    props.setProperty("password", "123456")
    val df: DataFrame = spark.read.jdbc("jdbc:mysql://hadoop202:3306/test", "user", props)
    df.show

    //释放资源
    spark.stop()
  }
}
```   

#### 3.3 向JDBC写数据  

```scala
object SparkSQL03_Datasource {
  def main(args: Array[String]): Unit = {
    //创建上下文环境配置对象
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL01_Demo")

    //创建SparkSession对象
    val spark: SparkSession = SparkSession.builder().config(conf).getOrCreate()
    import spark.implicits._

    val rdd: RDD[User2] = spark.sparkContext.makeRDD(List(User2("lisi", 20), User2("zs", 30)))
    val ds: Dataset[User2] = rdd.toDS
    //方式1：通用的方式  format指定写出类型
    ds.write
      .format("jdbc")
      .option("url", "jdbc:mysql://hadoop202:3306/test")
      .option("user", "root")
      .option("password", "123456")
      .option("dbtable", "user")
      .mode(SaveMode.Append)
      .save()

    //方式2：通过jdbc方法
    val props: Properties = new Properties()
    props.setProperty("user", "root")
    props.setProperty("password", "123456")
    ds.write.mode(SaveMode.Append).jdbc("jdbc:mysql://hadoop202:3306/test", "user", props)

    //释放资源
    spark.stop()
  }
}

case class User2(name: String, age: Long)
```    

### 4、Hive  
&emsp; Apache Hive 是 Hadoop 上的 SQL 引擎，Spark SQL编译时可以包含 Hive 支持，也可以不包含。  
&emsp; 包含 Hive 支持的 Spark SQL 可以支持 Hive 表访问、UDF (用户自定义函数)以及 Hive 查询语言(HiveQL/HQL)等。需要强调的一点是，如果要在 Spark SQL 中包含Hive 的库，并不需要事先安装 Hive。一般来说，最好还是在编译Spark SQL时引入Hive支持，这样就可以使用这些特性了。如果你下载的是二进制版本的 Spark，它应该已经在编译时添加了 Hive 支持。  
&emsp; 若要把 Spark SQL 连接到一个部署好的 Hive 上，你必须把 hive-site.xml 复制到 Spark的配置文件目录中($SPARK_HOME/conf)。即使没有部署好 Hive，Spark SQL 也可以运行，需要注意的是，如果你没有部署好Hive，Spark SQL 会在当前的工作目录中创建出自己的 Hive 元数据仓库，叫作 metastore_db。此外，对于使用部署好的Hive，如果你尝试使用 HiveQL 中的 CREATE TABLE (并非 CREATE EXTERNAL TABLE)语句来创建表，这些表会被放在你默认的文件系统中的 /user/hive/warehouse 目录中(如果你的 classpath 中有配好的 hdfs-site.xml，默认的文件系统就是 HDFS，否则就是本地文件系统)。  
&emsp; **spark-shell默认是Hive支持的；代码中是默认不支持的，需要手动指定（加一个参数即可）。**  

#### 4.1 使用内嵌Hive  
&emsp; 如果使用 Spark 内嵌的 Hive, 则什么都不用做, 直接使用即可。  
&emsp; Hive 的元数据存储在 derby 中, 仓库地址:$SPARK_HOME/spark-warehouse。   
```scala
scala> spark.sql("show tables").show
+--------+---------+-----------+
|database|tableName|isTemporary|
+--------+---------+-----------+
+--------+---------+-----------+

scala> spark.sql("create table aa(id int)")
19/02/09 18:36:10 WARN HiveMetaStore: Location: file:/opt/module/spark-local/spark-warehouse/aa specified for non-external table:aa
res2: org.apache.spark.sql.DataFrame = []

scala> spark.sql("show tables").show
+--------+---------+-----------+
|database|tableName|isTemporary|
+--------+---------+-----------+
| default|       aa|      false|
+--------+---------+-----------+
```    
&emsp; 向表中加载本地数据数据  
```scala
scala> spark.sql("load data local inpath './ids.txt' into table aa")
res8: org.apache.spark.sql.DataFrame = []

scala> spark.sql("select * from aa").show
+---+
| id|
+---+
|100|
|101|
|102|
|103|
|104|
|105|
|106|
+---+
```    
&emsp; 在实际使用中, 几乎没有任何人会使用内置的 Hive。  

#### 4.2 外部Hive应用  
&emsp; 如果Spark要接管Hive外部已经部署好的Hive，需要通过以下几个步骤。  
&emsp; (1)确定原有Hive是正常工作的  
&emsp; (2)需要把hive-site.xml拷贝到spark的conf/目录下  
&emsp; (3)如果以前hive-site.xml文件中，配置过Tez相关信息，注释掉  
&emsp; (4)把Mysql的驱动copy到Spark的jars/目录下  
&emsp; (5)需要提前启动hive服务，hive/bin/hiveservices.sh start  
&emsp; (6)如果访问不到hdfs，则需把core-site.xml和hdfs-site.xml拷贝到conf/目录  
&emsp; **启动 spark-shell**  
```scala
scala> spark.sql("show tables").show
+--------+---------+-----------+
|database|tableName|isTemporary|
+--------+---------+-----------+
| default|      emp|      false|
+--------+---------+-----------+

scala> spark.sql("select * from emp").show
19/02/09 19:40:28 WARN LazyStruct: Extra bytes detected at the end of the row! Ignoring similar problems.
+-----+-------+---------+----+----------+------+------+------+
|empno|  ename|      job| mgr|  hiredate|   sal|  comm|deptno|
+-----+-------+---------+----+----------+------+------+------+
| 7369|  SMITH|    CLERK|7902|1980-12-17| 800.0|  null|    20|
| 7499|  ALLEN| SALESMAN|7698| 1981-2-20|1600.0| 300.0|    30|
| 7521|   WARD| SALESMAN|7698| 1981-2-22|1250.0| 500.0|    30|
| 7566|  JONES|  MANAGER|7839|  1981-4-2|2975.0|  null|    20|
| 7654| MARTIN| SALESMAN|7698| 1981-9-28|1250.0|1400.0|    30|
| 7698|  BLAKE|  MANAGER|7839|  1981-5-1|2850.0|  null|    30|
| 7782|  CLARK|  MANAGER|7839|  1981-6-9|2450.0|  null|    10|
| 7788|  SCOTT|  ANALYST|7566| 1987-4-19|3000.0|  null|    20|
| 7839|   KING|PRESIDENT|null|1981-11-17|5000.0|  null|    10|
| 7844| TURNER| SALESMAN|7698|  1981-9-8|1500.0|   0.0|    30|
| 7876|  ADAMS|    CLERK|7788| 1987-5-23|1100.0|  null|    20|
| 7900|  JAMES|    CLERK|7698| 1981-12-3| 950.0|  null|    30|
| 7902|   FORD|  ANALYST|7566| 1981-12-3|3000.0|  null|    20|
| 7934| MILLER|    CLERK|7782| 1982-1-23|1300.0|  null|    10|
| 7944|zhiling|    CLERK|7782| 1982-1-23|1300.0|  null|    50|
+-----+-------+---------+----+----------+------+------+------+
```    

#### 4.3 运行Spark SQL CLI  
&emsp; Spark SQLCLI可以很方便的在本地运行Hive元数据服务以及从命令行执行查询任务。在Spark目录下执行如下命令启动Spark SQ LCLI，直接执行SQL语句，类似Hive窗口。  
```scala
bin/spark-sql
```   

#### 4.4 代码中操作Hive    
&emsp; 1）添加依赖   
```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-hive_2.11</artifactId>
    <version>2.1.1</version>
</dependency>
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>1.2.1</version>
</dependency>
```   
&emsp; 2）拷贝hive-site.xml到resources目录  
&emsp; 3）代码实现  
```scala
object SparkSQL08_Hive{
 def main(args: Array[String]): Unit = {
    //创建上下文环境配置对象
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL01_Demo")
    val spark: SparkSession = SparkSession
      .builder()
      .enableHiveSupport()
      .master("local[*]")
      .appName("SQLTest")
      .getOrCreate()
    spark.sql("show tables").show()
    //释放资源
    spark.stop()
  }
}
```  
