Zookeeper实战
---  
### 1、IDEA环境搭建   
1）创建一个Maven Module  
2）添加pom文件  
```xml
<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-core</artifactId>
			<version>2.8.2</version>
		</dependency>
		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.5.7</version>
		</dependency>
</dependencies>
```  
3）拷贝log4j.properties文件到项目根目录   
&emsp; 需要在项目的src/main/resources目录下，新建一个文件，命名为“log4j.properties”，在文件中填入。
```xml
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```
### 2、初始化ZooKeeper客户端  
```java
public class Zookeeper {

    private String connectString;
    private int sessionTimeout;
private ZooKeeper zkClient;

    @Before   //获取客户端对象
public void init() throws IOException {

        connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";
        int sessionTimeout = 10000;
        
       //参数解读 1集群连接字符串  2连接超时时间 单位:毫秒  3当前客户端默认的监控器
        zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
            }
        });
    }

    @After //关闭客户端对象
    public void close() throws InterruptedException {
        zkClient.close();
    }
}
```  
### 3、获取子节点列表,不监听  
```java
@Test
public void ls() throws IOException, KeeperException, InterruptedException {
    //用客户端对象做各种操作
    List<String> children = zkClient.getChildren("/", false);
    System.out.println(children);
}
```  
### 4、获取子节点列表,并监听  
```java
@Test
public void lsAndWatch() throws KeeperException, InterruptedException {
    List<String> children = zkClient.getChildren("/atguigu", new Watcher() {
        @Override
        public void process(WatchedEvent event) {
            System.out.println(event);
        }
    });
System.out.println(children);

    //因为设置了监听,所以当前线程不能结束
    Thread.sleep(Long.MAX_VALUE);
}
```   
### 5、创建子节点  
```java
@Test
public void create() throws KeeperException, InterruptedException {
//参数解读 1节点路径  2节点存储的数据  
//3节点的权限(使用Ids选个OPEN即可) 4节点类型 短暂 持久 短暂带序号 持久带序号
     String path = zkClient.create("/atguigu", "shanguigu".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

    //创建临时节点
//String path = zkClient.create("/atguigu2", "shanguigu".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);

System.out.println(path);

    //创建临时节点的话,需要线程阻塞
    //Thread.sleep(10000);
}
```   
### 6、判断Znode是否存在     
```java
@Test
public void exist() throws Exception {

	Stat stat = zkClient.exists("/atguigu", false);

	System.out.println(stat == null ? "not exist" : "exist");
}
```   
### 7、获取子节点存储的数据,不监听    
```java
@Test
public void get() throws KeeperException, InterruptedException {
    //判断节点是否存在
    Stat stat = zkClient.exists("/atguigu", false);
    if (stat == null) {
        System.out.println("节点不存在...");
        return;
    }

    byte[] data = zkClient.getData("/atguigu", false, stat);
    System.out.println(new String(data));
}
```
### 8、获取子节点存储的数据,并监听    
```java
@Test
public void getAndWatch() throws KeeperException, InterruptedException {
    //判断节点是否存在
    Stat stat = zkClient.exists("/atguigu", false);
    if (stat == null) {
        System.out.println("节点不存在...");
        return;
    }

    byte[] data = zkClient.getData("/atguigu", new Watcher() {
        @Override
        public void process(WatchedEvent event) {
            System.out.println(event);
        }
    }, stat);
    System.out.println(new String(data));
    //线程阻塞
    Thread.sleep(Long.MAX_VALUE);
}
```   
### 9、设置节点的值   
```java
@Test
public void set() throws KeeperException, InterruptedException {
    //判断节点是否存在
    Stat stat = zkClient.exists("/atguigu", false);
    if (stat == null) {
        System.out.println("节点不存在...");
        return;
    }
    //参数解读 1节点路径 2节点的值 3版本号
    zkClient.setData("/atguigu", "sgg".getBytes(), stat.getVersion());
}
```   
### 10、删除空节点    
```java
@Test
public void delete() throws KeeperException, InterruptedException {
    //判断节点是否存在
    Stat stat = zkClient.exists("/aaa", false);
    if (stat == null) {
        System.out.println("节点不存在...");
        return;
    }
    zkClient.delete("/aaa", stat.getVersion());
}
```    
### 11、删除非空节点,递归实现   
```java
//封装一个方法,方便递归调用
public void deleteAll(String path, ZooKeeper zk) throws KeeperException, InterruptedException {
    //判断节点是否存在
    Stat stat = zkClient.exists(path, false);
    if (stat == null) {
        System.out.println("节点不存在...");
        return;
    }
    //先获取当前传入节点下的所有子节点
    List<String> children = zk.getChildren(path, false);
    if (children.isEmpty()) {
        //说明传入的节点没有子节点,可以直接删除
        zk.delete(path, stat.getVersion());
    } else {
        //如果传入的节点有子节点,循环所有子节点
        for (String child : children) {
            //删除子节点,但是不知道子节点下面还有没有子节点,所以递归调用
            deleteAll(path + "/" + child, zk);  
        }
        //删除完所有子节点以后,记得删除传入的节点
        zk.delete(path, stat.getVersion());
    }
}
//测试deleteAll
@Test
public void testDeleteAll() throws KeeperException, InterruptedException {
    deleteAll("/atguigu",zkClient);
}
```   
