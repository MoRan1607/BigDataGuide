Zookeeper客户端命令
---  
### 1、启动客户端  
```  
[atguigu@hadoop103 zookeeper-3.5.7]$ bin/zkCli.sh  
```  
### 2、显示所有操作命令  
```  
[zk: localhost:2181(CONNECTED) 1] help  
```  
### 3、查看当前znode中所包含的内容  
```  
[zk: localhost:2181(CONNECTED) 0] ls /  
[zookeeper]    
```  
### 4、查看当前节点详细数据  
```  
[zk: localhost:2181(CONNECTED) 1] ls -s /
```  
### 5、分别创建2个普通节点    
```   
[zk: localhost:2181(CONNECTED) 3] create /sanguo "diaochan"    
```  
### 6、获得节点的值  
```   
[zk: localhost:2181(CONNECTED) 5] get /sanguo    
[zk: localhost:2181(CONNECTED) 6] get -s /sanguo  
[zk: localhost:2181(CONNECTED) 7] get -s /sanguo/shuguo  
```  
### 7、创建临时节点  
```   
[zk: localhost:2181(CONNECTED) 7] create -e /sanguo/wuguo "zhouyu"
```  
&emsp; 1）在当前客户端是能查看到的  
```   
[zk: localhost:2181(CONNECTED) 3] ls /sanguo 
```  
&emsp; 2）退出当前客户端然后再重启客户端   
```   
[zk: localhost:2181(CONNECTED) 12] quit
```  
``` 
[atguigu@hadoop104 zookeeper-3.5.7]$ bin/zkCli.sh
```  
&emsp; 3）再次查看根目录下短暂节点已经删除  
``` 
[zk: localhost:2181(CONNECTED) 0] ls /sanguo
```  
### 8、创建带序号的节点  
&emsp; 1）先创建一个普通的根节点/sanguo/weiguo  
``` 
[zk: localhost:2181(CONNECTED) 1] create /sanguo/weiguo "caocao"
```  
&emsp; 2）创建带序号的节点  
``` 
[zk: localhost:2181(CONNECTED) 2] create /sanguo/weiguo "caocao"
```  
``` 
[zk: localhost:2181(CONNECTED) 3] create -s /sanguo/weiguo
```  
``` 
[zk: localhost:2181(CONNECTED) 4] create -s /sanguo/weiguo
```  
``` 
[zk: localhost:2181(CONNECTED) 5] create -s /sanguo/weiguo
```  
``` 
[zk: localhost:2181(CONNECTED) 6] ls /sanguo
```  
&emsp; 如果节点下原来没有子节点，序号从0开始依次递增。如果原节点下已有2个节点，则再排序时从2开始，以此类推。    
### 9、修改节点数据值  
``` 
[zk: localhost:2181(CONNECTED) 6] set /sanguo/weiguo "caopi"
```  
### 10、节点的值变化监听   
&emsp; 1）在hadoop104主机上注册监听/sanguo节点数据变化  
```  
[zk: localhost:2181(CONNECTED) 26] [zk: localhost:2181(CONNECTED) 8] get -w /sanguo  
```   
&emsp; 2）在hadoop103主机上修改/sanguo节点的数据   
``` 
[zk: localhost:2181(CONNECTED) 1] set /sanguo "xishi"
```  
&emsp; 3）观察hadoop104主机收到数据变化的监听  
### 11、节点的子节点变化监听（路径变化）  
&emsp; 1）在hadoop104主机上注册监听/sanguo节点的子节点变化  
``` 
[zk: localhost:2181(CONNECTED) 1] ls -w /sanguo
```  
&emsp; 2）在hadoop103主机/sanguo节点上创建子节点   
``` 
[zk: localhost:2181(CONNECTED) 2] create /sanguo/jin "simayi"
```  
&emsp; 3）观察hadoop104主机收到子节点变化的监听   
### 12、删除节点  
``` 
[zk: localhost:2181(CONNECTED) 4] delete /sanguo/jin
```  
### 13、递归删除节点  
``` 
[zk: localhost:2181(CONNECTED) 15] deleteall /sanguo/shuguo
```  
### 14、查看节点状态  
``` 
[zk: localhost:2181(CONNECTED) 17] stat /sanguo
```  
