Spark运行模式
---
### 一、集群角色
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/Local%E6%A8%A1%E5%BC%8F/%E9%9B%86%E7%BE%A4%E6%A8%A1%E5%BC%8F.png"/>  
<p align="center">
</p>
</p>  

&emsp; 从**物理部署层面**上来看，Spark主要分为两种类型的节点：**Master节点和Worker节**点。**Master节点**主要运行集群管理器的中心化部分，所承载的作用是分配Application到Worker节点，维护Worker节点，Driver，Application的状态。**Worker节点**负责具体的业务运行。  
&emsp; 从**Spark程序运行层面**来看，Spark主要分为**驱动器节点和执行器节点**。

**1、Driver（驱动器节点）**  
&emsp; Spark的驱动器是执行开发程序中的main方法的进程。它负责开发人员编写的用来**创建SparkContext、创建RDD，以及进行RDD的转化操作和行动操作代码的执行**。如果你是用spark shell，那么当你启动Spark shell的时候，系统后台自启了一个Spark驱动器程序，就是在Spark shell中预加载的一个叫作 sc的SparkContext对象。如果驱动器程序终止，那么Spark应用也就结束了。主要负责：  
&emsp; 1）把用户程序转为作业（JOB）  
&emsp; 2）跟踪Executor的运行状况  
&emsp; 3）为执行器节点调度任务  
&emsp; 4）UI展示应用运行状况  
**2、Executor（执行器节点）**  
&emsp; Spark Executor是**一个工作进程，负责在 Spark 作业中运行任务，任务间相互独立**。Spark应用启动时，Executor节点被同时启动，并且始终伴随着整个Spark应用的生命周期而存在。如果有Executor节点发生了故障或崩溃，Spark应用也可以继续执行，会将出错节点上的任务调度到其他Executor节点上继续运行。主要负责：  
&emsp; 1）负责运行组成Spark应用的任务，并将结果返回给驱动器进程  
&emsp; 2）通过自身的块管理器（Block Manager）为用户程序中要求缓存的RDD提供内存式存储。RDD是直接缓存在Executor进程内的，因此任务可以在运行时充分利用缓存数据加速运算。  

### 二、运行模式
#### 1、Local模式  
1）概述  
&emsp; Local模式：Local模式就是运行在一台计算机上的模式，通常就是用于在本机上练手和测试。它可以通过以下集中方式设置master。  
&emsp; local：所有计算都运行在一个线程当中，没有任何并行计算，通常我们在本机执行一些测试代码，或者练手，就用这种模式;  
&emsp; local[K]：指定使用几个线程来运行计算，比如local[4]就是运行4个worker线程。通常我们的cpu有几个core，就指定几个线程，最大化利用cpu的计算能力;  
&emsp; local[*]：这种模式直接帮你按照cpu最多cores来设置线程数了。  

2）安装使用  
（1）上传并解压安装包  
```xml
[drift@hadoop102 sorfware]$ tar -zxvf spark-2.1.1-bin-hadoop2.7.tgz -C /opt/module/
[drift@hadoop102 module]$ mv spark-2.1.1-bin-hadoop2.7 spark
```  
（2）测试官方用例  
```xml
[drift@hadoop102 spark]$ bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--executor-memory 1G \
--total-executor-cores 2 \
./examples/jars/spark-examples_2.11-2.1.1.jar \
100    ==> 该数字代表迭代次数
```  

计算圆周率：  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/Local%E6%A8%A1%E5%BC%8F/%E5%9C%86%E5%91%A8%E7%8E%871.png"/>  
<p align="center">
</p>
</p>  

计算结果：  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/Local%E6%A8%A1%E5%BC%8F/%E5%9C%86%E5%91%A8%E7%8E%871%E7%BB%93%E6%9E%9C.png"/>  
<p align="center">
</p>
</p>  

（3）WordCount案例（直接在Spark-Shell模式运行）  
1.WordCount思路  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/Local%E6%A8%A1%E5%BC%8F/wordcount%E6%80%9D%E8%B7%AF.png"/>  
<p align="center">
</p>
</p>  

2.代码实现  
&emsp; 新建input文件夹，创建相应的txt文档，并在文档中写入适量单词  
```xml
scala> sc.textFile("input").flatMap(_.split(" ")).map
((_,1)).reduceByKey(_+_).collect
```  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/Local%E6%A8%A1%E5%BC%8F/spark-shell%E7%BB%93%E6%9E%9C.png"/>  
<p align="center">
</p>
</p>  

3.数据流分析：  
&emsp; textFile("input")：读取本地文件input文件夹数据；  
&emsp; flatMap(_.split(" "))：压平操作，按照空格分割符将一行数据映射成一个个单词；  
&emsp; map((_,1))：对每一个元素操作，将单词映射为元组；  
&emsp; reduceByKey(_+_)：按照key将值进行聚合，相加；  
&emsp; collect：将数据收集到Driver端展示。  

#### 2、Standalone模式 
&emsp; 构建一个由Master+Slave构成的Spark集群，Spark运行在集群中。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/Standalone%E6%A8%A1%E5%BC%8F/Standalone%E6%A8%A1%E5%BC%8F%20.png"/>  
<p align="center">
</p>
</p>  

1）安装使用  
进入spark的conf目录下，修改三个文件名  
&emsp; 将slaves.template复制为slaves  
&emsp; 将spark-env.sh.template复制为spark-env.sh  
&emsp; 将spark-defaults.conf.template复制为spark-defaults.conf.sh  

修改slaves，添加work节点  
```xml
hadoop102
hadoop103
hadoop104
```  

修改spark-env.sh  
```xml
SPARK_MASTER_HOST=hadoop102
SPARK_MASTER_PORT=7077
```  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/Standalone%E6%A8%A1%E5%BC%8F/spark-env.sh.png"/>  
<p align="center">
</p>
</p>  

修改sbin下面的spark-config.sh  
```xml
export JAVA_HOME=/opt/module/jdk1.8.0_144
```  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/Standalone%E6%A8%A1%E5%BC%8F/spark-config.sh.png"/>  
<p align="center">
</p>
</p>  

分发spark到集群各个节点，之后启动Spark  

2）官方案例：求π  
```xml
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop102:7077 \
--executor-memory 1G \
--total-executor-cores 2 \
./examples/jars/spark-examples_2.11-2.1.1.jar \
100
```  

结果：  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/Standalone%E6%A8%A1%E5%BC%8F/%E5%AE%98%E6%96%B9%E6%A1%88%E4%BE%8B%EF%BC%9A%E6%B1%82%CF%80.png"/>  
<p align="center">
</p>
</p>  

3）Spark-Shell模式测试
```xml
scala> sc.textFile("input").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect
```   

**`注意：`**  
提交应用程序概述  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/Standalone%E6%A8%A1%E5%BC%8F/%E6%8F%90%E4%BA%A4%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E6%A6%82%E8%BF%B0.jpg"/>  
<p align="center">
</p>
</p>  

#### 3、Yarn模式
&emsp; Spark客户端直接连接Yarn，不需要额外构建Spark集群。有**yarn-client和yarn-cluster两种模式**，主要区别在于：Driver程序的运行节点。  
&emsp; **`yarn-client`**：Driver程序运行在客户端，适用于交互、调试，希望立即看到app的输出。  
&emsp; **`yarn-cluster`**：Driver程序运行在由RM（ResourceManager）启动的AP（APPMaster）适用于生产环境。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/YARN%E6%A8%A1%E5%BC%8F/YARN%E6%A8%A1%E5%BC%8F.png"/>  
<p align="center">
</p>
</p>  

1）安装使用  
&emsp; **注意：在提交任务之前需启动HDFS以及YARN集群。**  
（1）修改hadoop配置文件yarn-site.xml,添加如下内容：  
```xml
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>
<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property>
```  

（2）修改spark-env.sh，添加如下配置：  
```xml
HADOOP_CONF_DIR=/opt/module/hadoop-2.7.2/etc/hadoop
YARN_CONF_DIR=/opt/module/hadoop-2.7.2/etc/hadoop
```   
**配置完之后分发yarn-site.xml、spark-env.sh（其实不分发也可以）**  

2)官方案例  
```xml
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode client \
./examples/jars/spark-examples_2.11-2.1.1.jar \
100
```  

执行结果：  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/YARN%E6%A8%A1%E5%BC%8F/%E5%9C%86%E5%91%A8%E7%8E%87%E7%BB%93%E6%9E%9C.png"/>  
<p align="center">
</p>
</p>  

#### 4、Mesos模式（了解即可）
&emsp; Spark客户端直接连接Mesos，不需要额外构建Spark集群。国内应用比较少，更多的是运用yarn调度。  

#### 5、几种模式比较
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/%E5%AF%B9%E6%AF%94%E5%92%8C%E7%BB%93%E6%9E%9C/%E5%87%A0%E7%A7%8D%E6%A8%A1%E5%BC%8F%E6%AF%94%E8%BE%83.png"/>  
<p align="center">
</p>
</p>  

#### 6、总结
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/%E5%AF%B9%E6%AF%94%E5%92%8C%E7%BB%93%E6%9E%9C/local.png"/>  
<p align="center">
</p>
</p>  
  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Spark%E6%96%87%E6%A1%A3Pics/Spark%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/%E5%AF%B9%E6%AF%94%E5%92%8C%E7%BB%93%E6%9E%9C/spark%E7%9A%84%E8%A7%92%E8%89%B2.png"/>  
<p align="center">
</p>
</p>  

&emsp; 如果想Driver运行在客户端，则采用Yarn-Client模式（客户端模式）  
&emsp; 如果想Driver运行按照集群资源分配，则用Yarn-Cluster模式（集群模式）  


