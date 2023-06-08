参考答案：

一个job含有多个stage，一个stage含有多个task。action 的触发会生成一个job，Job会提交给DAGScheduler，分解成Stage。

**Job**

- 包含很多task的并行计算，可以认为**是Spark RDD 里面的action**,每个action的计算会生成一个job。

- 用户提交的Job会提交给DAGScheduler，Job会被分解成Stage和Task。

**Stage**

- DAGScheduler根据shuffle将job划分为不同的stage，同一个stage中包含多个task，这些tasks有相同的shuffle dependencies。

- 一个Job会被拆分为多组Task，每组任务被称为一个Stage，就像Map Stage，Reduce Stage。

**Task**

- 即stage下的一个任务执行单元，一般来说，一个rdd有多少个partition，就会有多少个task，因为每一个task只是处理一个partition上的数据。

- 每个executor执行的task的数目，可以由submit时，--num-executors(on yarn)来指定。

Job->Stage->Task每一层都是1对n的关系。

**欢迎加入知识星球，获取《大数据面试题 V4.0》以及更多大数据开发内容**   
<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/Pics/%E6%98%9F%E7%90%83%E4%BC%98%E6%83%A0%E5%88%B8%20(21).png"  width="290" height="387"/>  
<p align="center">
</p>   
