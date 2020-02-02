一、Hive概述
---
### 1、什么是Hive  
&emsp; Hive是由Facebook开源用于解决海量结构化日志的数据统计。  
&emsp; Hive是基于Hadoop的**一个数据仓库工具**，可以将结构化的数据文件映射为一张表，并**提供类SQL查询功能**。   
&emsp; 本质是：**将HQL转化成MapReduce程序**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hive%E6%96%87%E6%A1%A3Pics/HQL%E8%BD%AC%E5%8C%96%E6%88%90MapReduce.png"/>  
<p align="center">
</p>
</p>  

&emsp; 1）Hive处理的数据存储在HDFS  
&emsp; 2）Hive分析数据底层的实现是MapReduce   
&emsp; 3）执行程序运行在Yarn上  

### 2、Hive优缺点
**`优点`**：  
&emsp; 1) 操作接口采用类SQL语法，提供快速开发的能力（简单、容易上手）。  
&emsp; 2) 避免了去写MapReduce，减少开发人员的学习成本。   
&emsp; 3) Hive的执行延迟比较高，因此Hive常用于数据分析，对实时性要求不高的场合。   
&emsp; 4) Hive优势在于处理大数据，对于处理小数据没有优势，因为Hive的执行延迟比较高。   
&emsp; 5) Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数。  
**`缺点`**：  
&emsp; 1）Hive的HQL表达能力有限  
&emsp; &emsp; （1）迭代式算法无法表达   
&emsp; &emsp; （2）数据挖掘方面不擅长  
&emsp; 2）Hive的效率比较低   
&emsp; &emsp; （1）Hive自动生成的MapReduce作业，通常情况下不够智能化  
&emsp; &emsp; （2）Hive调优比较困难，粒度较粗  

### 3、Hive架构原理
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hive%E6%96%87%E6%A1%A3Pics/Hive%E6%9E%B6%E6%9E%84.png"/>  
<p align="center">
</p>
</p>  

1）用户接口：Client  
&emsp; CLI（command-line interface）、JDBC/ODBC(jdbc访问hive)、WEBUI（浏览器访问hive）  
2）元数据：Metastore  
&emsp; 元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等；  
&emsp; 默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore  
3）Hadoop  
&emsp; 使用HDFS进行存储，使用MapReduce进行计算。  
4）驱动器：Driver  
&emsp; （1）解析器（SQL Parser）：将SQL字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr；对AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。  
&emsp; （2）编译器（Physical Plan）：将AST编译生成逻辑执行计划。  
&emsp; （3）优化器（Query Optimizer）：对逻辑执行计划进行优化。  
&emsp; （4）执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于Hive来说，就是MR/Spark。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hive%E6%96%87%E6%A1%A3Pics/Hive%E8%BF%90%E8%A1%8C%E6%9C%BA%E5%88%B6.png"/>  
<p align="center">
</p>
</p>  

&emsp; Hive通过给用户提供的一系列交互接口，接收到用户的指令(SQL)，使用自己的Driver，结合元数据(MetaStore)，将这些指令翻译成MapReduce，提交到Hadoop中执行，最后，将执行返回的结果输出到用户交互接口。 



