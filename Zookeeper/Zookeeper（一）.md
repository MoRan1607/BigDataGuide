Zookeeper介绍
---
&emsp; Zookeeper是一个开源的分布式的，为分布式应用提供协调服务的Apache项目。  
  
### 1、工作机制  
&emsp; Zookeeper从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它**负责存储和管理大家都关心的数据**，然后**接受观察者的注册**，一旦这些数据的状态发生变化，Zookeeper就将**负责通知已经在Zookeeper上注册的那些观察者做出相应的反应**，从而实现集群中类似Master/Slave管理模式  
&emsp; **Zookeeper=文件系统（配置文件保存）+通知机制（在ZK注册多个节点的共同配置文件保存在ZK，当配置文件被修改后，ZK会通知每个节点做出反应）**  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/ZK%E6%96%87%E6%A1%A3Pics/%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6.png"/>  
<p align="center">
</p>
</p>  

### 2、特点
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/ZK%E6%96%87%E6%A1%A3Pics/%E7%89%B9%E7%82%B9.png"/>  
<p align="center">
</p>
</p>  

&emsp; 1）Zookeeper：一个领导者（leader），多个跟随者（follower）组成的集群。  
&emsp; 2）Leader：负责进行投票的发起和决议，更新系统状态。   
&emsp; 3）Follower：用于接收客户请求并向客户端返回结果，在选举Leader过程中参与投票。   
&emsp; 4）**集群中只要有半数以上节点存活，Zookeeper集群就能正常服务**。   
&emsp; 5）全局数据一致：每个server保存一份相同的数据副本，client无论连接到哪个server，数据都是一致的。    
&emsp; 6）更新请求顺序进行，来自同一个client的更新请求按其发送顺序依次执行。    
&emsp; 7）数据更新原子性，一次数据更新要么成功，要么失败。   
&emsp; 8）实时性，在一定时间范围内，client能读到最新数据。  
  
### 3、数据结构
&emsp; ZooKeeper数据模型的结构**与Unix文件系统很类似**，整体上可以看作是一棵树，每个节点称做一个ZNode。每**一个znode默认能够存储1MB的数据**，每个ZNode都可以**通过其路径唯一标识**。   
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/ZK%E6%96%87%E6%A1%A3Pics/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.png"/>  
<p align="center">
</p>
</p>  

### 4、应用场景
&emsp; 提供的服务包括：**`统一命名服务`**、**`统一配置管理`**、**`统一集群管理`**、**`服务器节点动态上下线`**、**`软负载均衡`**等。  

1）**`统一命名服务`**  
&emsp; 在分布式环境下，经常需要对应用/服务进行统一命名，便于识别不同服务。  
&emsp; &emsp; （1）类似于域名与ip之间对应关系，ip不容易记住，而域名容易记住。   
&emsp; &emsp; （2）通过名称来获取资源或服务的地址，提供者等信息。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/ZK%E6%96%87%E6%A1%A3Pics/%E7%BB%9F%E4%B8%80%E5%91%BD%E5%90%8D%E6%9C%8D%E5%8A%A1.png"/>  
<p align="center">
</p>
</p>  

2）**`统一配置管理`**  
&emsp; 分布式环境下，配置文件管理和同步是一个常见问题。  
&emsp; &emsp; （1）一个集群中，所有节点的配置信息是一致的，比如 Hadoop 集群。   
&emsp; &emsp; （2）对配置文件修改后，希望能够快速同步到各个节点上。  
&emsp; 配置管理可交由ZooKeeper实现。  
&emsp; &emsp; （1）可将配置信息写入ZooKeeper上的一个Znode。   
&emsp; &emsp; （2）各个节点监听这个Znode。   
&emsp; &emsp; （3）一旦Znode中的数据被修改，ZooKeeper将通知各个节点。  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/ZK%E6%96%87%E6%A1%A3Pics/%E7%BB%9F%E4%B8%80%E9%85%8D%E7%BD%AE%E7%AE%A1%E7%90%86.png"/>  
<p align="center">
</p>
</p>  

3）**`统一集群管理`**  
&emsp; 分布式环境中，实时掌握每个节点的状态是必要的。  
&emsp; &emsp; （1）可根据节点实时状态做出一些调整。   
&emsp; 可交由ZooKeeper实现。   
&emsp; &emsp; （1）可将节点信息写入ZooKeeper上的一个Znode。   
&emsp; &emsp; （2）监听这个Znode可获取它的实时状态变化。   
&emsp; 典型应用   
&emsp; &emsp; （1）HBase中Master状态监控与选举。   
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/ZK%E6%96%87%E6%A1%A3Pics/%E7%BB%9F%E4%B8%80%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.png"/>  
<p align="center">
</p>
</p>  

4）**`服务器动态上下线`**  
&emsp; 客户端能实时洞察到服务器上下线的变化  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/ZK%E6%96%87%E6%A1%A3Pics/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8A%A8%E6%80%81%E4%B8%8A%E4%B8%8B%E7%BA%BF.png"/>  
<p align="center">
</p>
</p>  

5）**`软负载均衡`**  
&emsp; 在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/ZK%E6%96%87%E6%A1%A3Pics/%E8%BD%AF%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1.png"/>  
<p align="center">
</p>
</p>  




