## 企业级调优

### 1、Fetch抓取

Fetch抓取是指，**Hive中对某些情况的查询可以不必使用MapReduce计算**。例如：SELECT * FROM employees;在这种情况下，Hive可以简单地读取employee对应的存储目录下的文件，然后输出查询结果到控制台。

在hive-default.xml.template文件中hive.fetch.task.conversion默认是more，老版本hive默认是**minimal**，**该属性修改为more以后，在全局查找、字段查找、limit查找等都不走mapreduce**。

```xml
<property>
    <name>hive.fetch.task.conversion</name>
    <value>more</value>
    <description>
      Expects one of [none, minimal, more].
      Some select queries can be converted to single FETCH task minimizing latency.
      Currently the query should be single sourced not having any subquery and should not have any aggregations or distincts (which incurs RS), lateral views and joins.
      0. none : disable hive.fetch.task.conversion
      1. minimal : SELECT STAR, FILTER on partition columns, LIMIT only
      2. more  : SELECT, FILTER, LIMIT only (support TABLESAMPLE and virtual columns)
    </description>
</property>
```

1）案例实操

（1）把hive.fetch.task.conversion设置成none，然后执行查询语句，都会执行mapreduce程序。

```sql
hive (default)> set hive.fetch.task.conversion=none;
hive (default)> select * from emp;
hive (default)> select ename from emp;
hive (default)> select ename from emp limit 3;
```

（2）把hive.fetch.task.conversion设置成more，然后执行查询语句，如下查询方式都不会执行mapreduce程序。

```sql
hive (default)> set hive.fetch.task.conversion=more;
hive (default)> select * from emp;
hive (default)> select ename from emp;
hive (default)> select ename from emp limit 3;
```

### 2、表的优化

#### 2.1 小表、大表Join

将key相对分散，并且数据量小的表放在join的左边，这样可以有效减少内存溢出错误发生的几率；再进一步，可以使用map join让小的维度表（1000条以下的记录条数）先进内存。在map端完成reduce。

实际测试发现：新版的hive已经对小表JOIN大表和大表JOIN小表进行了优化。小表放在左边和右边已经没有明显区别。

案例实操

1）需求

测试大表JOIN小表和小表JOIN大表的效率

2）建大表、小表和JOIN后表的语句

```sql
// 创建大表
create table bigtable(id bigint, t bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';
// 创建小表
create table smalltable(id bigint, t bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';
// 创建join后表的语句
create table jointable(id bigint, t bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';
```

3）分别向大表和小表中导入数据

```sql
hive (default)> load data local inpath '/opt/module/datas/bigtable' into table bigtable;
hive (default)>load data local inpath '/opt/module/datas/smalltable' into table smalltable;
```

4）关闭mapjoin功能（默认是打开的）

```sql
set hive.auto.convert.join = false;
```

5）执行小表JOIN大表语句

```sql
insert overwrite table jointable
select b.id, b.t, b.uid, b.keyword, b.url_rank, b.click_num, b.click_url
from smalltable s
join bigtable  b
on b.id = s.id;

Time taken: 35.921 seconds
No rows affected (44.456 seconds)
```

6）执行大表JOIN小表语句

```sql
insert overwrite table jointable
select b.id, b.t, b.uid, b.keyword, b.url_rank, b.click_num, b.click_url
from bigtable  b
join smalltable  s
on s.id = b.id;

Time taken: 34.196 seconds
No rows affected (26.287 seconds)
```

#### 2.2 大表Join小表

1）空key过滤

有时join超时是因为某些key对应的数据太多，而相同key对应的数据都会发送到相同的reducer上，从而导致内存不够。此时我们应该仔细分析这些异常的key，很多情况下，这些key对应的数据是异常数据，我们需要在SQL语句中进行过滤。例如key对应的字段为空，操作如下：

案例实操

（1）配置历史服务器

配置mapred-site.xml

```xml
<property>
<name>mapreduce.jobhistory.address</name>
<value>hadoop102:10020</value>
</property>
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop102:19888</value>
</property>
```

启动历史服务器

```sql
sbin/mr-jobhistory-daemon.sh start historyserver
```

查看jobhistory：http://hadoop102:19888/jobhistory

（2）创建原始数据表、空id表、合并后数据表

```sql
// 创建原始表
create table ori(id bigint, t bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';
// 创建空id表
create table nullidtable(id bigint, t bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';
// 创建join后表的语句
create table jointable(id bigint, t bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';
```

（3）分别加载原始数据和空id数据到对应表中

```sql
hive (default)> load data local inpath '/opt/module/datas/ori' into table ori;
hive (default)> load data local inpath '/opt/module/datas/nullid' into table nullidtable;
```

（4）测试不过滤空id

```sql
hive (default)> insert overwrite table jointable select n.* from nullidtable n
left join ori o on n.id = o.id;

Time taken: 42.038 seconds
Time taken: 37.284 seconds
```

（5）测试过滤空id

```sql
hive (default)> insert overwrite table jointable select n.* from (select * from nullidtable where id is not null ) n  left join ori o on n.id = o.id;

Time taken: 31.725 seconds
Time taken: 28.876 seconds
```

2）空key转换

有时虽然某个key为空对应的数据很多，但是相应的数据不是异常数据，必须要包含在join的结果中，此时我们可以表a中key为空的字段赋一个随机的值，使得数据随机均匀地分不到不同的reducer上。例如：

案例实操：

不随机分布空null值：

（1）设置5个reduce个数

```sql
set mapreduce.job.reduces = 5;
```

（2）JOIN两张表

```sql
insert overwrite table jointable 
select n.* from nullidtable n left join ori b on n.id = b.id;
```

结果：如下图所示，可以看出来，出现了数据倾斜，某些reducer的资源消耗远大于其他reducer。

![image-20220404230341921](Pictures/image-20220404230341921.png)

随机分布空null值

（1）设置5个reduce个数

```sql
set mapreduce.job.reduces = 5;
```

（2）JOIN两张表

```sql
insert overwrite table jointable
select n.* from nullidtable n full join ori o on 
nvl(n.id,rand()) = o.id;
```

结果：如下图所示，可以看出来，消除了数据倾斜，负载均衡reducer的资源消耗

![image-20220404230433056](Pictures/image-20220404230433056.png)

#### 2.3 MapJoin（MR引擎）

如果不指定MapJoin或者不符合MapJoin的条件，那么Hive解析器会将Join操作转换成Common Join，即：在Reduce阶段完成join。容易发生数据倾斜。可以用MapJoin把小表全部加载到内存在map端进行join，避免reducer处理。

1）开启MapJoin参数设置

（1）设置自动选择Mapjoin

```sql
set hive.auto.convert.join = true;   // 默认为true
```

（2）大表小表的阈值设置（默认25M一下认为是小表）：

```sql
set hive.mapjoin.smalltable.filesize=25000000;
```

2）MapJoin工作机制

<img src="Pictures/image-20220404230750310.png" alt="image-20220404230750310" style="zoom:85%;" />

3）案例实操

（1）开启Mapjoin功能

```sql
set hive.auto.convert.join = true;
```

（2）执行小表JOIN大表语句

```sql
insert overwrite table jointable
select b.id, b.t, b.uid, b.keyword, b.url_rank, b.click_num, b.click_url
from smalltable s
left join bigtable  b
on s.id = b.id;

Time taken: 24.594 seconds
```

（3）执行大表JOIN小表语句

```sql
insert overwrite table jointable
select b.id, b.t, b.uid, b.keyword, b.url_rank, b.click_num, b.click_url
from bigtable  b
left join smalltable  s
on s.id = b.id;

Time taken: 24.315 seconds
```

#### 2.4 Group By

默认情况下，Map阶段同一Key数据分发给一个reduce，当一个key数据过大时就倾斜了。

![image-20220404231433631](Pictures/image-20220404231433631.png)

并不是所有的聚合操作都需要在Reduce端完成，很多聚合操作都可以先在Map端进行部分聚合，最后在Reduce端得出最终结果。

1）开启Map端聚合参数设置

（1）是否在Map端进行聚合，默认为True

```sql
set hive.map.aggr = true
```

（2）在Map端进行聚合操作的条目数目

```sql
set hive.groupby.mapaggr.checkinterval = 100000
```

（3）有数据倾斜的时候进行负载均衡（默认是false）

```sql
set hive.groupby.skewindata = true
```

**当选项设定为 true，生成的查询计划会有两个MR Job**。第一个MR Job中，Map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是**相同的Group By Key有可能被分发到不同的Reduce中**，从而达到负载均衡的目的；第二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中（这个过程可以保证相同的Group By Key被分布到同一个Reduce中），最后完成最终的聚合操作。

```sql
hive (default)> select deptno from emp group by deptno;
Stage-Stage-1: Map: 1  Reduce: 5   Cumulative CPU: 23.68 sec   HDFS Read: 19987 HDFS Write: 9 SUCCESS
Total MapReduce CPU Time Spent: 23 seconds 680 msec
OK
deptno
10
20
30
```

优化以后

```sql
hive (default)> set hive.groupby.skewindata = true;
hive (default)> select deptno from emp group by deptno;
Stage-Stage-1: Map: 1  Reduce: 5   Cumulative CPU: 28.53 sec   HDFS Read: 18209 HDFS Write: 534 SUCCESS
Stage-Stage-2: Map: 1  Reduce: 5   Cumulative CPU: 38.32 sec   HDFS Read: 15014 HDFS Write: 9 SUCCESS
Total MapReduce CPU Time Spent: 1 minutes 6 seconds 850 msec
OK
deptno
10
20
30
```

#### 2.5 笛卡尔积

尽量避免笛卡尔积，join的时候不加on条件，或者无效的on条件，Hive只能使用1个reducer来完成笛卡尔积。

#### 2.6 行列过滤

列处理：在SELECT中，只拿需要的列，如果有，尽量使用分区过滤，少用SELECT *。

行处理：在分区剪裁中，当使用外关联时，如果将副表的过滤条件写在Where后面，那么就会先全表关联，之后再过滤，比如：

案例实操：

1）测试先关联两张表，再用where条件过滤

```sql
hive (default)> select o.id from bigtable b
join ori o on o.id = b.id
where o.id <= 10;

Time taken: 34.406 seconds, Fetched: 100 row(s)
```

2）通过子查询后，再关联表

```sql
hive (default)> select b.id from bigtable b
join (select id from ori where id <= 10 ) o on b.id = o.id;

Time taken: 30.058 seconds, Fetched: 100 row(s)
```

### 3、合理设置Map及Reduce数（MR引擎）
