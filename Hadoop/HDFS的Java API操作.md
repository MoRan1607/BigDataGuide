## HDFS的Java API操作
### 一、HDFS客户端环境准备
1）根据自己电脑的操作系统拷贝对应的编译后的hadoop jar包到非中文路径  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E6%96%87%E6%A1%A3Pics/hadoop%20jar%E5%8C%85.png"/>  
<p align="center">
</p>
</p>  

2）配置HADOOP_HOME环境变量和path路径  
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E6%96%87%E6%A1%A3Pics/%E9%85%8D%E7%BD%AEHADOOP_HOME%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E5%92%8Cpath%E8%B7%AF%E5%BE%841.png"/>  
<p align="center">
</p>
</p>  

<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hadoop%E6%96%87%E6%A1%A3Pics/%E9%85%8D%E7%BD%AEHADOOP_HOME%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E5%92%8Cpath%E8%B7%AF%E5%BE%842.png"/>  
<p align="center">
</p>
</p>  

### 二、HDFS的API操作
**新建Maven工程并添加依赖**  
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
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.7.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>2.7.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>2.7.2</version>
    </dependency>
</dependencies>
```  

**API操作代码**  
#### 1、HDFS文件上传
```java  
    @Test
    public void testPut() throws Exception {  
        Configuration configuration = new Configuration();  
        FileSystem fileSystem = FileSystem.get(  
                new URI("hdfs://hadoop102:9000"),  
                configuration,  
                "drift");  

        fileSystem.copyFromLocalFile(  
                new Path("f:/hello.txt"), new Path("/0308_666/hello1.txt"));  
        fileSystem.close();  
    }  
```  

#### 2、HDFS文件下载
```java
    @Test
    public void testDownload() throws Exception {
        // 1 获取文件系统
        Configuration configuration = new Configuration();
        FileSystem fileSystem = FileSystem.get(
                new URI("hdfs://hadoop102:9000"), configuration, "drift");
        // 2 执行下载操作
        fileSystem.copyToLocalFile(
                false,
                new Path("/0308_666/hello.txt"),
                new Path("f:/hello1.txt"),
                true);
        // 3 关闭资源
        fileSystem.close();
        System.out.println("over");
    }
```  

#### 3、HDFS文件夹删除
```java  
    @Test
    public void delete() throws Exception {
        // 1 获取文件系统
        Configuration configuration = new Configuration();
        FileSystem fileSystem = FileSystem.get(
                new URI("hdfs://hadoop102:9000"), configuration, "drift");
        // 2 执行删除操作
        fileSystem.delete(new Path("/0308_777"), true);
        // 3 关闭资源
        fileSystem.close();
        System.out.println("over");
    }
```  

#### 4、HDFS文件夹名更改
```java  
    @Test
    public void testRename() throws Exception {
        // 1 获取文件系统
        Configuration configuration = new Configuration();
        FileSystem fileSystem = FileSystem.get(
                new URI("hdfs://hadoop102:9000"), configuration, "drift");
        // 2 执行重命名操作
        fileSystem.rename(new Path("/0308_666/hello.txt"), new Path("/0308_666/hello2.txt"));
        // 3 关闭资源
        fileSystem.close();
        System.out.println("over");
    }
```  

#### 5、HDFS文件详情查看
```java  
    @Test
    public void testLS1() throws Exception {
        // 1 获取文件系统
        Configuration configuration = new Configuration();
        FileSystem fileSystem = FileSystem.get(
                new URI("hdfs://hadoop102:9000"), configuration, "drift");
        // 2 查询文件信息
        RemoteIterator<LocatedFileStatus> listFiles = fileSystem.listFiles(new Path("/"), true);
        while (listFiles.hasNext()) {
            LocatedFileStatus fileStatus = listFiles.next();
            // 文件的长度
            System.out.println(fileStatus.getLen());
            // 文件的名字
            System.out.println(fileStatus.getPath().getName());
            // 文件的权限
            System.out.println(fileStatus.getPermission());
            BlockLocation[] locations = fileStatus.getBlockLocations();
            for (BlockLocation location : locations) {
                String[] hosts = location.getHosts();
                for (String host : hosts) {
                    System.out.println(host);
                }
            }
            System.out.println("---------------分割线---------------");
        }
        // 3 关闭资源
        fileSystem.close();
    }
```  

#### 6、HDFS文件和文件夹判断
```java  
    @Test
    public void testLS2() throws Exception {
        // 1 获取文件系统
        Configuration configuration = new Configuration();
        FileSystem fileSystem = FileSystem.get(
                new URI("hdfs://hadoop102:9000"), configuration, "drift");
        // 2 文件和文件夹的判断
        FileStatus[] fileStatuses = fileSystem.listStatus(new Path("/"));
        for (FileStatus fileStatus : fileStatuses) {
            if (fileStatus.isFile()) {
                System.out.println("F:" + fileStatus.getPath().getName());
            } else {
                System.out.println("D:" + fileStatus.getPath().getName());
            }
        }
        // 3 关闭资源
        fileSystem.close();
    }
```  

**HDFS的I/O流操作**  
#### 1、HDFS文件上传  
```java  
    @Test
    public void testPut2() throws Exception {
        //1.获取hdfs的客户端
        Configuration configuration = new Configuration();
        FileSystem fileSystem = FileSystem.get(
                new URI("hdfs://hadoop102:9000"), configuration, "drift");

        //2.创建输入流
        FileInputStream fileInputStream = new FileInputStream(new File("f:/hello2.txt"));

        //3.创建输出流
        FSDataOutputStream outputStream = fileSystem.create(new Path("/0308_666/hello3.txt"));

        //4.流的拷贝
        IOUtils.copyBytes(fileInputStream, outputStream, configuration);

        //5.关闭资源
        fileSystem.close();
    }
```  

#### 2、HDFS文件下载  
```java  
    @Test
    public void testDownload2() throws Exception {
        //1.获取hdfs的客户端
        Configuration configuration = new Configuration();
        FileSystem fileSystem = FileSystem.get(
                new URI("hdfs://hadoop102:9000"), configuration, "drift");

        //2.创建输入流
        FSDataInputStream inputStream = fileSystem.open(new Path("/0308_666/hello3.txt"));

        //3.创建输出流
        FileOutputStream outputStream = new FileOutputStream(new File("f:/hello3.txt"));

        //4.流的拷贝
        IOUtils.copyBytes(inputStream, outputStream, configuration);

        //5.关闭资源
        fileSystem.close();
        System.out.println("over");
    }
```   

#### 3、定位文件读取  
**需求：分块读取HDFS上的大文件**  
```java  
    /**
     * 文件的下载：
     *  1.下载第一块
     */
    @Test
    public void testSeek1() throws Exception {
        //1.获取hdfs的客户端
        Configuration configuration = new Configuration();
        FileSystem fileSystem = FileSystem.get(
                new URI("hdfs://hadoop102:9000"), configuration, "drift");

        //2.创建输入流
        FSDataInputStream inputStream = fileSystem.open(new Path("/user/drift/hadoop-2.7.2.tar.gz"));

        //3.创建输出流
        FileOutputStream outputStream = new FileOutputStream(new File("f:/hadoop-2.7.2.tar.gz.part1"));

        //4.流的拷贝
        byte[] buf = new byte[1024];

        for (int i = 0; i < 1024 * 128; i++) {
            inputStream.read(buf);
            outputStream.write(buf);
        }

        //5.关闭资源
        IOUtils.closeStream(inputStream);
        IOUtils.closeStream(outputStream);
        fileSystem.close();
        System.out.println("over");
    }

    /**
     * 文件的下载：
     *  2.下载第二块
     */
    @Test
    public void testSeek2() throws Exception {
        //1.获取hdfs的客户端
        Configuration configuration = new Configuration();
        FileSystem fileSystem = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "drift");

        //2.创建输入流
        FSDataInputStream inputStream = fileSystem.open(new Path("/user/drift/hadoop-2.7.2.tar.gz"));

        //3.创建输出流
        FileOutputStream outputStream = new FileOutputStream(new File("f:/hadoop-2.7.2.tar.gz.part2"));

        //4.流的拷贝
        inputStream.seek(1024 * 1024 * 128);
        IOUtils.copyBytes(inputStream, outputStream, configuration);

        //5.关闭资源
        IOUtils.closeStream(inputStream);
        IOUtils.closeStream(outputStream);
        fileSystem.close();
        System.out.println("over");
    }

}
```  


