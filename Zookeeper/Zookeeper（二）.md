Zookeeper单机和分布式安装
---  
## 单机模式
### 1、安装前准备  
&emsp; 1）虚拟机安装好JDK  
&emsp; 2）将Zookeeper安装包拷贝到Linux系统下  
&emsp; 3）将Zookpeer解压到指定目录  
```  
[atguigu@hadoop102 software]$ tar -zxvf zookeeper-3.5.7.tar.gz -C /opt/module/
```  
### 2、配置修改  
&emsp; 1）将/opt/module/zookeeper-3.5.7/conf这个路径下的zoo_sample.cfg修改为zoo.cfg  
```
[atguigu@hadoop102 conf]$ mv zoo_sample.cfg zoo.cfg
```  
&emsp; 2）打开zoo.cfg文件，修改dataDir路径  
```  
[atguigu@hadoop102 zookeeper-3.5.7]$ vim zoo.cfg
```  
&emsp; 修改如下内容：  
```  
dataDir=/opt/module/zookeeper-3.5.7/zkData
```  
&emsp; 3）在/opt/module/zookeeper-3.5.7/这个目录上创建zkData文件夹  
```  
[atguigu@hadoop102 zookeeper-3.5.7]$ mkdir zkData
```  
### 3、启动Zookeeper  
&emsp; 1）启动Zookeeper  
```  
[atguigu@hadoop102 zookeeper-3.5.7]$ bin/zkServer.sh start
```  
&emsp; 2）查看进程是否启动   
```  
[atguigu@hadoop102 zookeeper-3.5.7]$ jps  
4020 Jps  
4001 QuorumPeerMain   
```  
3）查看状态  
```  
[atguigu@hadoop102 zookeeper-3.5.7]$ bin/zkServer.sh status
```  
&emsp; 4）启动客户端  
```  
[atguigu@hadoop102 zookeeper-3.5.7]$ bin/zkCli.sh
```  
&emsp; 5）退出客户端  
```  
[zk: localhost:2181(CONNECTED) 0] quit
```  
&emsp; 6）停止Zookeeper  
```  
[atguigu@hadoop102 zookeeper-3.5.7]$ bin/zkServer.sh stop
```  
### 4、配置参数解读  
Zookeeper中的配置文件zoo.cfg中参数含义解读如下：  
1）tickTime =2000：通信心跳数，Zookeeper服务器与客户端心跳时间，单位毫秒  
&emsp; Zookeeper使用的基本时间，服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime时间就会发送一个心跳，时间单位为毫秒。  
&emsp; 它用于心跳机制，并且设置最小的session超时时间为两倍心跳时间。(session的最小超时时间是2xtickTime)  
2）initLimit =10：LF初始通信时限  
&emsp; 集群中的Follower跟随者服务器与Leader领导者服务器之间初始连接时能容忍的最多心跳数（tickTime的数量），用它来限定集群中的Zookeeper服务器连接到Leader的时限。  
3）syncLimit =5：LF同步通信时限  
&emsp; 集群中Leader与Follower之间的最大响应时间单位，假如响应超过syncLimit * tickTime，Leader认为Follwer死掉，从服务器列表中删除Follwer。  
4）dataDir：数据文件目录+数据持久化路径  
&emsp; 主要用于保存Zookeeper中的数据。  
5）clientPort =2181：客户端连接端口  
&emsp; 监听客户端连接的端口。    

分布式安装  
---
**1、集群规划**  
&emsp; 在hadoop102、hadoop103和hadoop104三个节点上部署Zookeeper。  
**2、解压安装**  
&emsp; 1）解压Zookeeper安装包到/opt/module/目录下  
```  
[atguigu@hadoop102 software]$ tar -zxvf zookeeper-3.5.7.tar.gz -C /opt/module/
```  
&emsp; 2）同步/opt/module/zookeeper-3.5.7目录内容到hadoop103、hadoop104  
```  
[atguigu@hadoop102 module]$ xsync zookeeper-3.5.7/
```  
**3、配置服务器编号**  
&emsp; 1）在/opt/module/zookeeper-3.5.7/这个目录下创建zkData  
```  
[atguigu@hadoop102 zookeeper-3.5.7]$ mkdir -p zkData
```  
&emsp; 2）在/opt/module/zookeeper-3.5.7/zkData目录下创建一个myid的文件  
```  
[atguigu@hadoop102 zkData]$ touch myid
```  
&emsp; **添加myid文件，注意一定要在linux里面创建，在notepad++里面很可能乱码**  
&emsp; 3）编辑myid文件   
```  
[atguigu@hadoop102 zkData]$ vim myid
```  
&emsp; 在文件中添加与server对应的编号：  
```  
2
```  
&emsp; 4）拷贝配置好的zookeeper到其他机器上  
```  
[atguigu@hadoop102 zkData]$ xsync myid
```  
&emsp; **并分别在hadoop103、hadoop104上修改myid文件中内容为3、4**  
**4、配置zoo.cfg文件**  
&emsp; 1）重命名/opt/module/zookeeper-3.5.7/conf这个目录下的zoo_sample.cfg为zoo.cfg  
```  
[atguigu@hadoop102 conf]$ mv zoo_sample.cfg zoo.cfg
```  
&emsp; 2）打开zoo.cfg文件  
```  
[atguigu@hadoop102 conf]$ vim zoo.cfg
```  
&emsp; 修改数据存储路径配置  
```  
dataDir=/opt/module/zookeeper-3.5.7/zkData
```  
&emsp; 增加如下配置  
```  
#######################cluster##########################  
server.2=hadoop102:2888:3888  
server.3=hadoop103:2888:3888  
server.4=hadoop104:2888:3888  
```   
&emsp; 3）同步zoo.cfg配置文件  
```  
[atguigu@hadoop102 conf]$ xsync zoo.cfg
```  
&emsp; 4）配置参数解读  
```  
server.A=B:C:D。
```  
&emsp; A是一个数字，表示这个是第几号服务器；  
&emsp; 集群模式下配置一个文件myid，这个文件在dataDir目录下，这个文件里面有一个数据就是A的值，**Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。**  
&emsp; B是这个服务器的地址；  
&emsp; C是这个服务器Follower与集群中的Leader服务器交换信息的端口；  
&emsp; D是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。  
**5、集群操作**  
&emsp; 1）分别启动Zookeeper  
```  
[atguigu@hadoop102 zookeeper-3.5.7]$ bin/zkServer.sh start  
[atguigu@hadoop103 zookeeper-3.5.7]$ bin/zkServer.sh start  
[atguigu@hadoop104 zookeeper-3.5.7]$ bin/zkServer.sh start  
```  
&emsp; 2）查看状态（三台机器）  
```  
[atguigu@hadoop102 zookeeper-3.5.7]# bin/zkServer.sh status  
```  
