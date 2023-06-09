### Hive分区和分桶的区别

可回答：Hive分区和分桶的逻辑

问过的一些公司：字节，小米，阿里云社招，京东x2，猿辅导，竞技世界，美团，抖音

参考答案：

**1、定义上**

**分区**

- Hive的分区使用HDFS的子目录功能实现。每一个子目录包含了分区对应的列名和每一列的值。

- Hive的分区方式：由于Hive实际是存储在HDFS上的抽象，Hive的一个分区名对应一个目录名，子分区名就是子目录名，并不是一个实际字段。

- 所以可以这样理解，当我们在插入数据的时候指定分区，其实就是新建一个目录或者子目录，或者在原有的目录上添加数据文件。

**注意：partitned by子句中定义的列是表中正式的列（分区列），但是数据文件内并不包含这些列。**

```mysql
# 创建分区表
create table student(
id int,
name string,
age int,
address string
)
partitioned by (dt string,type string)                 # 制定分区
row format delimited fields terminated by '\t'         # 指定字段分隔符为tab
collection items terminated by ','                     # 指定数组中字段分隔符为逗号
map keys terminated by ':'                             # 指定字典中KV分隔符为冒号
lines terminated by '\n'                               # 指定行分隔符为回车换行
stored as textfile                                     # 指定存储类型为文件
;

# 将数据加载到表中（此时时静态分区）
load data local inpath '/root/student.txt' into  test.student partition(class='一班');
```

**分桶：**

- 分桶表是在表或者分区表的基础上，进一步对表进行组织，Hive使用 对分桶所用的值；
- 进行hash，并用hash结果除以桶的个数做取余运算的方式来分桶，保证了每个桶中都有数据，但每个桶中的数据条数不一定相等。

注意：

创建分区表时：

- 可以使用distribute by(sno) sort by(sno asc) 或是使用clustered by(字段)

- 当排序和分桶的字段相同的时候使用cluster by， 就等同于分桶+排序(sort)

```mysql
# 创建分桶表
create table student(
id int,
name string,
age int,
address string
)
clustered by(id) sorted by(age) into 4 buckets
row format delimited fields terminated by '\t'
stored as textfile;

# 开启分桶
set hive.enforce.bucketing = true;
# 插入数据
insert overwrite table studentselect id ,name ,age ,address from employees;
# 也可以用另一种插入方式
load data local inpath '/root/student.txt' into test.student;
```

**2、数据类型上**

分桶随机分割数据库，分区是非随机分割数据库。因为分桶是按照列的哈希函数进行分割的，相对比较平均；而分区是按照列的值来进行分割的，容易造成数据倾斜。

分桶是对应不同的文件（细粒度），分区是对应不同的文件夹（粗粒度）。桶是更为细粒度的数据范围划分，分桶的比分区获得更高的查询处理效率，使取样更高效。

注意：普通表（外部表、内部表）、分区表这三个都是对应HDFS上的目录，桶表对应是目录里的文件。

**欢迎加入知识星球，获取《大数据面试题 V4.0》以及更多大数据开发内容**   
<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/Pics/%E6%98%9F%E7%90%83%E4%BC%98%E6%83%A0%E5%88%B8%20(21).png"  width="290" height="387"/>  
<p align="center">
</p>   
