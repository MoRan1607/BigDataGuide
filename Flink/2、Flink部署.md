## Flink部署

### 一、standalone模式

#### 1.1 安装

解压缩`flink-1.10.0-bin-scala_2.11.tgz`，进入conf目录中。

**1） 修改 flink/conf/flink-conf.yaml 文件**

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%871.png" alt="img;" />

**2）修改 /conf/slaves文件**

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%872.png" alt="img;" />

**3）分发给另外两台机子**

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%873.png" alt="img;" />

**4）启动**

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%874.png" alt="img;" />

访问http://localhost:8081可以对flink集群和任务进行监控管理。

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%875.png" alt="img;" />

#### 1.2 提交任务

**1）准备数据文件**

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%876.png" alt="img;" />

**2）把含数据文件的文件夹，分发到taskmanage机器中**

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%877.png" alt="img;" />

由于读取数据是从本地磁盘读取，实际任务会被分发到taskmanage的机器中，所以要把目标文件分发。

**3）执行程序**

```shell
./flink run -c com.atguigu.wc.StreamWordCount –p 2 FlinkTutorial-1.0-SNAPSHOT-jar-with-dependencies.jar --host lcoalhost –port 7777 
```

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%878.png" alt="img;" />

**4）到目标文件夹中查看计算结果**

注意：计算结果根据会保存到taskmanage的机器下，不会在jobmanage下。

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%879.png" alt="img;" />

**5）在控制台查看计算过程**

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%8710.png" alt="img;" />

### 二、YARN模式

> 以Yarn模式部署Flink任务时，要求Flink是有Hadoop支持的版本，Hadoop环境需要保证版本在2.2以上，并且集群中安装有HDFS服务。

#### 2.1 Flink on Yarn

Flink提供了两种在yarn上运行的模式，分别为`Session-Cluster`和`Per-Job-Cluster`模式。

##### **2.1.1 Session-cluster 模式**

 <img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%8711.png" alt="img;" />

Session-Cluster模式需要先启动集群，然后再提交作业，接着会向yarn申请一块空间后，资源永远保持不变。如果资源满了，下一个作业就无法提交，只能等到yarn中的其中一个作业执行完成后，释放了资源，下个作业才会正常提交。所有作业共享Dispatcher和ResourceManager；共享资源；适合规模小执行时间短的作业。

<font color=red>在yarn中初始化一个flink集群，开辟指定的资源，以后提交任务都向这里提交。这个flink集群会常驻在yarn集群中，除非手工停止。</font>

##### 2.1.2 Per-Job-Cluster 模式

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%8712.png" alt="img;" />

一个Job会对应一个集群，每提交一个作业会根据自身的情况，都会单独向yarn申请资源，直到作业执行完成，一个作业的失败与否并不会影响下一个作业的正常提交和运行。独享Dispatcher和ResourceManager，按需接受资源申请；适合规模大长时间运行的作业。

<font color=red>每次提交都会创建一个新的flink集群，任务之间互相独立，互不影响，方便管理。任务执行完成之后创建的集群也会消失。</font>

#### 2.2 Session Cluster

1）启动hadoop集群（略）

2）启动yarn-session

```shell
./yarn-session.sh -n 2 -s 2 -jm 1024 -tm 1024 -nm test -d
```

其中：

`-n(--container)`：TaskManager的数量。

`-s(--slots)`：	每个TaskManager的slot数量，默认一个slot一个core，默认每个taskmanager的slot的个数为1，有时可以多一些taskmanager，做冗余。

`-jm`：JobManager的内存（单位MB)。

`-tm`：每个taskmanager的内存（单位MB)。

`-nm`：yarn 的appName(现在yarn的ui上的名字)。 

`-d`：后台执行。

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%8713.png" alt="img;" />

3）执行任务

```shell
./flink run -c com.atguigu.wc.StreamWordCount  FlinkTutorial-1.0-SNAPSHOT-jar-with-dependencies.jar --host lcoalhost –port 7777
```

4）去yarn控制台查看任务状态

<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Flink%E6%96%87%E6%A1%A3Pics/Flink%E9%83%A8%E7%BD%B2/%E5%9B%BE%E7%89%8714.png" alt="img;" />

5）取消yarn-session

```shell
yarn application --kill application_1577588252906_0001
```

#### 2.3  Per Job Cluster

1）启动hadoop集群（略）

2）<font color=red>不启动yarn-session</font>，直接执行job

```shell
./flink run –m yarn-cluster -c com.atguigu.wc.StreamWordCount  FlinkTutorial-1.0-SNAPSHOT-jar-with-dependencies.jar --host lcoalhost –port 7777
```

### 三、Kubernetes部署

容器化部署时目前业界很流行的一项技术，基于Docker镜像运行能够让用户更加方便地对应用进行管理和运维。容器管理工具中最为流行的就是Kubernetes（k8s），而Flink也在最近的版本中支持了k8s部署模式。

1）搭建Kubernetes集群（略）

2）配置各组件的yaml文件

在k8s上构建Flink Session Cluster，需要将Flink集群的组件对应的docker镜像分别在k8s上启动，包括JobManager、TaskManager、JobManagerService三个镜像服务。每个镜像服务都可以从中央镜像仓库中获取。

3）启动Flink Session Cluster

```shell
// 启动jobmanager-service 服务
kubectl create -f jobmanager-service.yaml

// 启动jobmanager-deployment服务
kubectl create -f jobmanager-deployment.yaml

// 启动taskmanager-deployment服务
kubectl create -f taskmanager-deployment.yaml
```

4）访问Flink UI页面

集群启动后，就可以通过JobManagerServicers中配置的WebUI端口，用浏览器输入以下url来访问Flink UI页面了：

`http://{JobManagerHost:Port}/api/v1/namespaces/default/services/flink-jobmanager:ui/proxy`



