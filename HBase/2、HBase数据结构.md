二、HBase数据结构
---
### 1、RowKey
&emsp; 与nosql数据库们一样,RowKey是用来检索记录的主键。访问HBASE table中的行，只有三种方式：  
&emsp; 1）通过单个RowKey访问(get)  
&emsp; 2）通过RowKey的range（正则）(like)  
&emsp; 3）全表扫描(scan)  
&emsp; RowKey行键 (RowKey)可以是任意字符串(最大长度是64KB，实际应用中长度一般为 10-100bytes)，在HBASE内部，RowKey保存为字节数组。存储时，数据按照RowKey的字典序(byte order)排序存储。设计RowKey时，要充分排序存储这个特性，将经常一起读取的行存储放到一起。(位置相关性)  

### 2、Column Family
&emsp; 列族：HBASE表中的每个列，都归属于某个列族。列族是表的schema的一部 分(而列不是)，必须在使用表之前定义。列名都以列族作为前缀。例如 courses:history，courses:math都属于courses 这个列族。  

### 3、Cell
&emsp; 由{rowkey, column Family:columu, version} 唯一确定的单元。cell中的数据是没有类型的，全部是字节码形式存贮。  
&emsp; 关键字：无类型、字节码  

### 4、Time Stamp
&emsp; HBASE 中通过rowkey和columns确定的为一个存贮单元称为cell。每个 cell都保存 着同一份数据的多个版本。版本通过时间戳来索引。时间戳的类型是 64位整型。时间戳可以由HBASE(在数据写入时自动 )赋值，此时时间戳是精确到毫秒 的当前系统时间。时间戳也可以由客户显式赋值。如果应用程序要避免数据版 本冲突，就必须自己生成具有唯一性的时间戳。每个 cell中，不同版本的数据按照时间倒序排序，即最新的数据排在最前面。  
&emsp; 为了避免数据存在过多版本造成的的管理 (包括存贮和索引)负担，HBASE提供 了两种数据版本回收方式。一是保存数据的最后n个版本，二是保存最近一段 时间内的版本（比如最近七天）。用户可以针对每个列族进行设置。  

### 5、命名空间
&emsp; 命名空间结构如下：  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/HBase%E6%96%87%E6%A1%A3Pics/HBase%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4%E7%BB%93%E6%9E%84.png"/>  
<p align="center">
</p>
</p>  

1) Table：表，所有的表都是命名空间的成员，即表必属于某个命名空间，如果没有指定，则在default默认的命名空间中。
2) RegionServer group：一个命名空间包含了默认的RegionServer Group。
3) Permission：权限，命名空间能够让我们来定义访问控制列表ACL（Access Control List）。例如，创建表，读取表，删除，更新等等操作。
4) Quota：限额，可以强制一个命名空间可包含的region的数量。


















