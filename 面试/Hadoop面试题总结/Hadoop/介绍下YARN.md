介绍YARN，可以先考虑下面两个问题

1）如何管理集群资源？

2）如何给任务合理分配资源？

YARN是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个**分布式的操作系统平台**，而**MapReduce等运算程序则相当于运行于操作系统之上的应用程序**。

YARN 作为一个资源管理、任务调度的框架，主要包含ResourceManager、NodeManager、ApplicationMaster和Container模块。

**YARN基础架构**

<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/%E9%9D%A2%E8%AF%95/Hadoop%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93/Hadoop/Pics/Hadoop-%E4%BB%8B%E7%BB%8D%E4%B8%8BYARN01.png" />  
<p align="center">
</p>
</p>  

1）ResourceManager（RM）主要作用如下：

- 处理客户端请求

- 监控NodeManager

- 启动或监控ApplicationMaster

- 资源的分配与调度

2）NodeManager（NM）主要作用如下：

- 管理单个节点上的资源

- 处理来自ResourceManager的命令

- 处理来自ApplicationMaster的命令

3）ApplicationMaster（AM）作用如下：

- 为应用程序申请资源并分配给内部的任务

- 任务的监督与容错

4）Container

- Container是YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等。

可以结合“YARN有什么优势，能解决什么问题？”一起回答
