Spark SQL实战
---   
### 1、数据准备  
&emsp; Spark-sql操作所有的数据均来自Hive，首先在Hive中创建表,并导入数据。一共有3张表：1张用户行为表，1张城市表，1张产品表。   

```xml
CREATE TABLE `user_visit_action`(
  `date` string,
  `user_id` bigint,
  `session_id` string,
  `page_id` bigint,
  `action_time` string,
  `search_keyword` string,
  `click_category_id` bigint,
  `click_product_id` bigint,
  `order_category_ids` string,
  `order_product_ids` string,
  `pay_category_ids` string,
  `pay_product_ids` string,
  `city_id` bigint)
row format delimited fields terminated by '\t';
load data local inpath '/opt/module/data/user_visit_action.txt' into table sparkpractice.user_visit_action;

CREATE TABLE `product_info`(
  `product_id` bigint,
  `product_name` string,
  `extend_info` string)
row format delimited fields terminated by '\t';
load data local inpath '/opt/module/data/product_info.txt' into table sparkpractice.product_info;

CREATE TABLE `city_info`(
  `city_id` bigint,
  `city_name` string,
  `area` string)
row format delimited fields terminated by '\t';
load data local inpath '/opt/module/data/city_info.txt' into table sparkpractice.city_info;
```     

### 2、需求  
#### 2.1 需求简介  
&emsp; 这里的热门商品是从点击量的维度来看的，计算各个区域前三大热门商品，并备注上每个商品在主要城市中的分布比例，超过两个城市用其他显示。   
&emsp; 例如：  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%20SQL/4/2021-05-13_184130.png"/>  
<p align="center">
</p>
</p>  

#### 2.2 思路分析  
&emsp; 1）使用sql来完成，碰到复杂的需求，可以使用udf或udaf  
&emsp; 2）查询出来所有的点击记录，并与city_info表连接，得到每个城市所在的地区，与Product_info表连接得到产品名称  
&emsp; 3）按照地区和商品名称分组，统计出每个商品在每个地区的总点击次数  
&emsp; 4）每个地区内按照点击次数降序排列  
&emsp; 5）只取前三名，并把结果保存在数据库中  
&emsp; 6）城市备注需要自定义UDAF函数   

#### 2.3 代码实现  
&emsp; 1）UDAF函数定义   
```scala
class AreaClickUDAF extends UserDefinedAggregateFunction {
  // 输入数据的类型:  北京  String
  override def inputSchema: StructType = {
    StructType(StructField("city_name", StringType) :: Nil)
    //        StructType(Array(StructField("city_name", StringType)))
  }

  // 缓存的数据的类型: 北京->1000, 天津->5000  Map,  总的点击量  1000/?
  override def bufferSchema: StructType = {
    // MapType(StringType, LongType) 还需要标注 map的key的类型和value的类型
    StructType(StructField("city_count", MapType(StringType, LongType)) :: StructField("total_count", LongType) :: Nil)
  }

  // 输出的数据类型  "北京21.2%，天津13.2%，其他65.6%"  String
  override def dataType: DataType = StringType

  // 相同的输入是否应用有相同的输出.
  override def deterministic: Boolean = true

  // 给存储数据初始化
  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    //初始化map缓存
    buffer(0) = Map[String, Long]()
    // 初始化总的点击量
    buffer(1) = 0L
  }

  // 分区内合并 Map[城市名, 点击量]
  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    // 首先拿到城市名, 然后把城市名作为key去查看map中是否存在, 如果存在就把对应的值 +1, 如果不存在, 则直接0+1
    val cityName = input.getString(0)
    //        val map: collection.Map[String, Long] = buffer.getMap[String, Long](0)
    val map: Map[String, Long] = buffer.getAs[Map[String, Long]](0)
    buffer(0) = map + (cityName -> (map.getOrElse(cityName, 0L) + 1L))
    // 碰到一个城市, 则总的点击量要+1
    buffer(1) = buffer.getLong(1) + 1L
  }

  // 分区间的合并
  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    val map1 = buffer1.getAs[Map[String, Long]](0)
    val map2 = buffer2.getAs[Map[String, Long]](0)

    // 把map1的键值对与map2中的累积, 最后赋值给buffer1
    buffer1(0) = map1.foldLeft(map2) {
      case (map, (k, v)) =>
        map + (k -> (map.getOrElse(k, 0L) + v))
    }

    buffer1(1) = buffer1.getLong(1) + buffer2.getLong(1)
  }

  // 最终的输出. "北京21.2%，天津13.2%，其他65.6%"
  override def evaluate(buffer: Row): Any = {
    val cityCountMap = buffer.getAs[Map[String, Long]](0)
    val totalCount = buffer.getLong(1)

    var citysRatio: List[CityRemark] = cityCountMap.toList.sortBy(-_._2).take(2).map {
      case (cityName, count) => {
        CityRemark(cityName, count.toDouble / totalCount)
      }
    }
    // 如果城市的个数超过2才显示其他
    if (cityCountMap.size > 2) {
      citysRatio = citysRatio :+ CityRemark("其他", citysRatio.foldLeft(1D)(_ - _.cityRatio))
    }
    citysRatio.mkString(", ")
  }
}


case class CityRemark(cityName: String, cityRatio: Double) {
  val formatter = new DecimalFormat("0.00%")
  override def toString: String = s"$cityName:${formatter.format(cityRatio)}"
}
```    

&emsp; 2)具体实现   
```scala
object SparkSQL04_TopN {
  def main(args: Array[String]): Unit = {
    val spark: SparkSession = SparkSession
      .builder()
      .master("local[2]")
      .appName("AreaClickApp")
      .enableHiveSupport()
      .getOrCreate()
    spark.sql("use sparkpractice")
    // 0 注册自定义聚合函数
    spark.udf.register("city_remark", new AreaClickUDAF)
    // 1. 查询出所有的点击记录,并和城市表产品表做内连接
    spark.sql(
      """
        |select
        |    c.*,
        |    v.click_product_id,
        |    p.product_name
        |from user_visit_action v join city_info c join product_info p on v.city_id=c.city_id and v.click_product_id=p.product_id
        |where click_product_id>-1
      """.stripMargin).createOrReplaceTempView("t1")

    // 2. 计算每个区域, 每个产品的点击量
    spark.sql(
      """
        |select
        |    t1.area,
        |    t1.product_name,
        |    count(*) click_count,
        |    city_remark(t1.city_name)
        |from t1
        |group by t1.area, t1.product_name
      """.stripMargin).createOrReplaceTempView("t2")

    // 3. 对每个区域内产品的点击量进行倒序排列
    spark.sql(
      """
        |select
        |    *,
        |    rank() over(partition by t2.area order by t2.click_count desc) rank
        |from t2
      """.stripMargin).createOrReplaceTempView("t3")

    // 4. 每个区域取top3

    spark.sql(
      """
        |select
        |    *
        |from t3
        |where rank<=3
      """.stripMargin).show

    //释放资源
    spark.stop()

  }
}
```     
