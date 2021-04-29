Zookeeper内部原理
---   
### 1、结点类型  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/ZK%E6%96%87%E6%A1%A3Pics/ZK%E7%BB%93%E7%82%B9%E7%B1%BB%E5%9E%8B.png"/>  
<p align="center">
</p>
</p>  
**持久（Persistent）**：客户端和服务器端断开连接后，创建的节点不删除  
**短暂（Ephermeral）**：客户端和服务器端断开连接后，创建的节点自己删除  
（1）持久化目录节点  
客户端与Zookeeper断开连接后，该节点依旧存在。  
（2）持久化顺序标号目录节点  
客户端与Zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号。  
（3）临时目录节点  
客户端与Zookeeper断开连接后，该节点被删除。  
（4）临时顺序编号目录节点  
客户端与Zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号。  

### 2、Stat结构体  
1）**czxid-创建节点的事务zxid**  
&emsp; 每次修改ZooKeeper状态都会收到一个zxid形式的时间戳，也就是ZooKeeper事务ID。  
&emsp; 事务ID是ZooKeeper中所有修改总的次序。每个修改都有唯一的zxid，如果zxid1小于zxid2，那么zxid1在zxid2之前发生。  
2）ctime - znode被创建的毫秒数(从1970年开始)  
3）mzxid - znode最后更新的事务zxid  
4）mtime - znode最后修改的毫秒数(从1970年开始)  
5）pZxid-znode最后更新的子节点zxid  
6）cversion - znode子节点变化号，znode子节点修改次数  
7）**dataversion - znode数据变化号**  
8）aclVersion - znode访问控制列表的变化号  
9）ephemeralOwner- 如果是临时节点，这个是znode拥有者的session id。如果不是临时节点则是0。  
10）**dataLength- znode的数据长度**  
11）**numChildren - znode子节点数量**  

### 3、监听器原理  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/ZK%E6%96%87%E6%A1%A3Pics/ZK%E7%9B%91%E5%90%AC%E5%99%A8%E5%8E%9F%E7%90%86.png"/>  
<p align="center">
</p>
</p>  

**监听器原理详解：**    
1）首先要有一个main()线程。  
2）在main线程中创建Zookeeper客户端，这时就会创建两个线程，一个负责网络连接通信(connet)，一个负责监听(listener)。  
3）通过connect线程将注册的监听事件发送给Zookeepers。  
4）在Zookeeper的注册监听器列表中将注册的监听事件添加到列表中。5)Zookeeper监听到有数据或路径变化，就会将这个消息发送给listener线程。  
6）listener线程内部调用了process()方法。  
**常见的监听：**  
1）监听节点数据的变化  
&emsp; get path[watch]  
2）监听子节点增减的变化  
&emsp; ls path[watch]  

### 4、选举机制   
1）**半数机制：集群中半数以上机器存活，集群可用。所以Zookeeper适合安装奇数台服务器。**  
2）Zookeeper虽然在配置文件中并没有指定Master和Slave。但是，Zookeeper工作时，是有一个节点为Leader，其他则为Follower，Leader是通过内部的选举机制临时产生的。  
3）以一个简单的例子来说明整个选举的过程。  
&emsp; 假设有五台服务器组成的Zookeeper集群，它们的id从1-5，同时它们都是最新启动的，也就是没有历史数据，在存放数据量这一点上，都是一样的。假设这些服务器依序启动，来看看会发生什么。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/ZK%E6%96%87%E6%A1%A3Pics/ZK%E9%80%89%E4%B8%BE%E6%9C%BA%E5%88%B6.png"/>  
<p align="center">
</p>
</p>  
1）服务器1启动，发起一次选举。服务器1投自己一票。此时服务器1票数一票，不够半数以上（3票），选举无法完成，服务器1状态保持为LOOKING；  
2）服务器2启动，再发起一次选举。服务器1和2分别投自己一票并交换选票信息：此时服务器1发现服务器2的ID比自己目前投票推举的（服务器1）大，更改选票为推举服务器2。此时服务器1票数0票，服务器2票数2票，没有半数以上结果，选举无法完成，服务器1，2状态保持LOOKING；  
3）服务器3启动，发起一次选举。此时服务器1和2都会更改选票为服务器3。此次投票结果：服务器1为0票，服务器2为0票，服务器3为3票。此时服务器3的票数已经超过半数，服务器3当选Leader。服务器1，2更改状态为FOLLOWING，服务器3更改状态为LEADING；  
4）服务器4启动，发起一次选举。此时服务器1，2，3已经不是LOOKING状态，不会更改选票信息。交换选票信息结果：服务器3为3票，服务器4为1票。此时服务器4服从多数，更改选票信息为服务器3，并更改状态为FOLLOWING；  
5）服务器5启动，同4一样当小弟。  

### 5、写数据流程  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/ZK%E6%96%87%E6%A1%A3Pics/ZK%E5%86%99%E6%95%B0%E6%8D%AE%E6%B5%81%E7%A8%8B.png"/>  
<p align="center">
</p>
</p>  
