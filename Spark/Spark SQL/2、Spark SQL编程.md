Spark SQL编程
---  
### 1、Spark Session新的起始点  
&emsp; 在老的版本中，SparkSQL提供两种SQL查询起始点：一个叫SQLContext，用于Spark自己提供的SQL查询；一个叫HiveContext，用于连接Hive的查询。  
&emsp; **SparkSession是Spark最新的SQL查询起始点**，实质上是SQLContext和HiveContext的组合，所以在SQLContex和HiveContext上可用的API在SparkSession上同样是可以使用的。SparkSession内部封装了sparkContext，所以计算实际上是由sparkContext完成的。当我们使用 spark-shell 的时候, spark 会自动的创建一个叫做spark的SparkSession, 就像我们以前可以自动获取到一个sc来表示SparkContext   
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20SQL/2/%E5%9B%BE%E7%89%871.png"/>  
<p align="center">
</p>
</p>  

### 2、DataFrame  
&emsp; Spark SQL的DataFrame API 允许我们使用 DataFrame 而不用必须去注册临时表或者生成SQL表达式。DataFrame API 既有transformation操作也有action操作，DataFrame的转换从本质上来说更具有关系， 而 DataSet API 提供了更加函数式的 API。  
#### 2.1 创建DataFrame  
&emsp; 在Spark SQL中SparkSession是创建DataFrame和执行SQL的入口，创建DataFrame有三种方式：**通过Spark的数据源进行创建**；**从一个存在的RDD进行转换**；**还可以从Hive Table进行查询返回**。  
#### 2.2 SQL风格语法  
&emsp; SQL语法风格是指我们查询数据的时候使用SQL语句来查询，这种风格的查询必须要有临时视图或者全局视图来辅助  
&emsp; 1）创建一个DataFrame    
```scala
scala> val df = spark.read.json("/opt/module/spark-local/people.json")
df: org.apache.spark.sql.DataFrame = [age: bigint， name: string]
```   
&emsp; 2）对DataFrame创建一个临时表  
```scala
scala> df.createOrReplaceTempView("people")
```   
&emsp; 3）通过SQL语句实现查询全表  
```scala
scala> val sqlDF = spark.sql("SELECT * FROM people")
sqlDF: org.apache.spark.sql.DataFrame = [age: bigint， name: string]
```   
&emsp; 4）结果展示  
```scala
scala> sqlDF.show
+---+--------+
|age|    name|
+---+--------+
| 18|qiaofeng|
| 19|  duanyu|
| 20|   xuzhu|
+---+--------+
```   
&emsp;**注意**：普通临时表是Session范围内的，如果想应用范围内有效，可以使用全局临时表。使用全局临时表时需要全路径访问，如：global_temp.people  

&emsp; 5）对于DataFrame创建一个全局表  
```scala
scala> df.createGlobalTempView("people")
```  
&emsp; 6）通过SQL语句实现查询全表  
```scala
scala> spark.sql("SELECT * FROM global_temp.people").show()
+---+--------+
|age|    name|
+---+--------+
| 18|qiaofeng|
| 19|  duanyu|
| 20|   xuzhu|
+---+--------+

scala> spark.newSession().sql("SELECT * FROM global_temp.people").show()
+---+--------+
|age|    name|
+---+--------+
| 18|qiaofeng|
| 19|  duanyu|
| 20|   xuzhu|
+---+--------+
```    

#### 2.3 DSL风格语法  
&emsp; DataFrame提供一个特定领域语言(domain-specific language, DSL)去管理结构化的数据，可以在Scala, Java, Python和R中使用DSL，使用DSL语法风格不必去创建临时视图了。    
&emsp; 1）创建一个DataFrame  
```scala
scala> val df = spark.read.json("/opt/module/spark-local /people.json")
df: org.apache.spark.sql.DataFrame = [age: bigint， name: string]
```    
&emsp; 2）查看DataFrame的Schema信息  
```scala
scala> df.printSchema
root
 |-- age: Long (nullable = true)
 |-- name: string (nullable = true)
```    
&emsp; 3）只查看”name”列数据  
```scala
scala> df.select("name").show()
+--------+
|    name|
+--------+
|qiaofeng|
|  duanyu|
|   xuzhu|
+--------+
```    
&emsp; 4）查看所有列  
```scala
scala> df.select("*").show
+--------+---------+
|    name |age|
+--------+---------+
|qiaofeng|       18|
|  duanyu|       19|
|   xuzhu|       20|
+--------+---------+
```    
&emsp; 5）查看”name”列数据以及”age+1”数据   
注意:涉及到运算的时候, 每列都必须使用$  
```scala
scala> df.select($"name",$"age" + 1).show
+--------+---------+
|    name|(age + 1)|
+--------+---------+
|qiaofeng|       19|
|  duanyu|       20|
|   xuzhu|       21|
+--------+---------+
```    
&emsp; 6）查看”age”大于”19”的数据  
```scala
scala> df.filter($"age">19).show
+---+-----+
|age| name|
+---+-----+
| 20|xuzhu|
+---+-----+
```    
&emsp; 7）按照”age”分组，查看数据条数   
```scala
scala> df.groupBy("age").count.show
+---+-----+
|age|count|
+---+-----+
| 19|    1|
| 18|    1|
| 20|    1|
+---+-----+
```    
#### 2.4 RDD转换为DataFrame  
&emsp; 在 IDEA 中开发程序时，如果需要RDD 与DF 或者DS 之间互相操作，那么需要引入import spark.implicits._。  
&emsp; 这里的spark不是Scala中的包名，而是创建的sparkSession 对象的变量名称，所以必须先创建 SparkSession 对象再导入。这里的 spark 对象不能使用var 声明，因为 Scala 只支持val 修饰的对象的引入。  
&emsp; spark-shell 中无需导入，自动完成此操作。  
```scala
scala> val idRDD = sc.textFile("data/id.txt") scala> idRDD.toDF("id").show
+---+
| id|
+---+
| 1|
| 2|
| 3|
| 4|
+---+
```    
&emsp; **实际开发中，一般通过样例类将RDD转换为DataFrame。**  
```scala
scala> case class User(name:String, age:Int) defined class User
scala> sc.makeRDD(List(("zhangsan",30), ("lisi",40))).map(t=>User(t._1, t._2)).toDF.show
+--------+---+
|	name|age|
+--------+---+
```    
#### 2.5 DataFrame转换为RDD  
&emsp; DataFrame其实就是对RDD的封装，所以可以直接获取内部的RDD  
```scala
scala> val df = sc.makeRDD(List(("zhangsan",30), ("lisi",40))).map(t=>User(t._1, t._2)).toDF
df: org.apache.spark.sql.DataFrame = [name: string, age: int]

scala> val rdd = df.rdd
rdd: org.apache.spark.rdd.RDD[org.apache.spark.sql.Row] = MapPartitionsRDD[46] at rdd at <console>:25

scala> val array = rdd.collect
array: Array[org.apache.spark.sql.Row] = Array([zhangsan,30], [lisi,40])
```    
&emsp; **注意：此时得到的RDD存储类型为Row**  
```scala
scala> array(0)
res28: org.apache.spark.sql.Row = [zhangsan,30] scala> array(0)(0)
res29: Any = zhangsan
scala> array(0).getAs[String]("name") res30: String = zhangsan
```    

### 3、DataSet  
&emsp; DataSet是具有强类型的数据集合，需要提供对应的类型信息。  
#### 3.1 创建DataSet  
&emsp; 1）使用样例类序列创建DataSet  
```scala
scala> case class Person(name: String, age: Long)
defined class Person

scala> val caseClassDS = Seq(Person("wangyuyan",2)).toDS()

caseClassDS: org.apache.spark.sql.Dataset[Person] = [name: string, age: Long]

scala> caseClassDS.show
+---------+---+
|     name|age|
+---------+---+
|wangyuyan|  2|
+---------+---+
```    
&emsp; 2）使用基本类型的序列创建DataSet  
```scala
scala> val ds = Seq(1,2,3,4,5,6).toDS
ds: org.apache.spark.sql.Dataset[Int] = [value: int]

scala> ds.show
+-----+
|value|
+-----+
|    1|
|    2|
|    3|
|    4|
|    5|
|    6|
+-----+
```     

**注意:在实际使用的时候，很少用到把序列转换成DataSet，更多是通过RDD来得到DataSet。**

#### 3.2 RDD转换为DataSet  
&emsp; SparkSQL能够自动将包含有样例类的RDD转换成DataSet，样例类定义了table的结构，样例类属性通过反射变成了表的列名。样例类可以包含诸如Seq或者Array等复杂的结构。  

&emsp; 1)创建一个RDD  
```scala 
scala> val peopleRDD = sc.textFile("/opt/module/spark-local/people.txt")

peopleRDD: org.apache.spark.rdd.RDD[String] = /opt/module/spark-local/people.txt MapPartitionsRDD[19] at textFile at <console>:24
```    

&emsp; 2)创建一个样例类  
```scala
scala> case class Person(name:String,age:Int)
defined class Person
&emsp; 3)将RDD转化为DataSet  
scala> peopleRDD.map(line => {val fields = line.split(",");Person(fields(0),fields(1). toInt)}).toDS

res0: org.apache.spark.sql.Dataset[Person] = [name: string, age: Long]
```    
#### 3.3DataSet转换为RDD  
&emsp; 调用rdd方法即可。  
&emsp; 1)创建一个DataSet  
```scala
scala> val DS = Seq(Person("zhangcuishan", 32)).toDS()

DS: org.apache.spark.sql.Dataset[Person] = [name: string, age: Long]
```   
&emsp; 2)将DataSet转换为RDD  
```scala
scala> DS.rdd

res1: org.apache.spark.rdd.RDD[Person] = MapPartitionsRDD[6] at rdd at <console>:28
```   

### 4、DataFrame与DataSet的互操作  
#### 4.1 DataFrame转为DataSet  
&emsp; 1）创建一个DateFrame  
```scala
scala> val df = spark.read.json("/opt/module/spark-local/people.json")

df: org.apache.spark.sql.DataFrame = [age: bigint, name: string]
```  
&emsp; 2)创建一个样例类   
```scala
scala> case class Person(name: String,age: Long)
defined class Person
```  
&emsp; 3)将DataFrame转化为DataSet     
```scala
scala> df.as[Person]

res5: org.apache.spark.sql.Dataset[Person] = [age: bigint, name: string]
```  

&emsp; 这种方法就是在给出每一列的类型后，使用as方法，转成Dataset，这在数据类型是DataFrame又需要针对各个字段处理时极为方便。在使用一些特殊的操作时，一定要加上 import spark.implicits._ 不然toDF、toDS无法使用。    

#### 4.2Dataset转为DataFrame  
&emsp; 1)创建一个样例类  
```scala
scala> case class Person(name: String,age: Long)
defined class Person
```  
&emsp; 2)创建DataSet  
```scala
scala> val ds = Seq(Person("zhangwuji",32)).toDS()

ds: org.apache.spark.sql.Dataset[Person] = [name: string, age: bigint]
```  
&emsp; 3)将DataSet转化为DataFrame  
```scala
scala> var df = ds.toDF
df: org.apache.spark.sql.DataFrame = [name: string, age: bigint]
```  
&emsp; 4)展示  
```scala
scala> df.show
+---------+---+
|     name|age|
+---------+---+
|zhangwuji| 32|
+---------+---+
```  

#### 5、IDEA实践  
&emsp; 1）Maven工程添加依赖  
```xml
<dependency>
	<groupId>org.apache.spark</groupId>
	<artifactId>spark-sql_2.11</artifactId>
	<version>2.1.1</version>
</dependency>
```  

&emsp; 2）代码实现   
```scala
object SparkSQL01_Demo {
  def main(args: Array[String]): Unit = {
    //创建上下文环境配置对象
    val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL01_Demo")

    //创建SparkSession对象
    val spark: SparkSession = SparkSession.builder().config(conf).getOrCreate()
    //RDD=>DataFrame=>DataSet转换需要引入隐式转换规则，否则无法转换
    //spark不是包名，是上下文环境对象名
    import spark.implicits._

    //读取json文件 创建DataFrame  {"username": "lisi","age": 18}
    val df: DataFrame = spark.read.json("D:\\dev\\workspace\\spark-bak\\spark-bak-00\\input\\test.json")
    //df.show()

    //SQL风格语法
    df.createOrReplaceTempView("user")
    //spark.sql("select avg(age) from user").show

    //DSL风格语法
    //df.select("username","age").show()

    //*****RDD=>DataFrame=>DataSet*****
    //RDD
    val rdd1: RDD[(Int, String, Int)] = spark.sparkContext.makeRDD(List((1,"qiaofeng",30),(2,"xuzhu",28),(3,"duanyu",20)))

    //DataFrame
    val df1: DataFrame = rdd1.toDF("id","name","age")
    //df1.show()

    //DateSet
    val ds1: Dataset[User] = df1.as[User]
    //ds1.show()

    //*****DataSet=>DataFrame=>RDD*****
    //DataFrame
    val df2: DataFrame = ds1.toDF()

    //RDD  返回的RDD类型为Row，里面提供的getXXX方法可以获取字段值，类似jdbc处理结果集，但是索引从0开始
    val rdd2: RDD[Row] = df2.rdd
    //rdd2.foreach(a=>println(a.getString(1)))

    //*****RDD=>DataSe*****
    rdd1.map{
      case (id,name,age)=>User(id,name,age)
    }.toDS()

    //*****DataSet=>=>RDD*****
    ds1.rdd

    //释放资源
    spark.stop()
  }
}
case class User(id:Int,name:String,age:Int)
```   
