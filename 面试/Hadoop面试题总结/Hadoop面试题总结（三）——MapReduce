## Hadoop面试题总结（三）——MapReduce  

### 1、谈谈Hadoop序列化和反序列化及自定义bean对象实现序列化?  
1）序列化和反序列化  
&emsp; （1）序列化就是把内存中的对象，转换成字节序列（或其他数据传输协议）以便于存储（持久化）和网络传输。   
&emsp; （2）反序列化就是将收到字节序列（或其他数据传输协议）或者是硬盘的持久化数据，转换成内存中的对象。  
&emsp; （3）Java的序列化是一个重量级序列化框架（Serializable），一个对象被序列化后，会附带很多额外的信息（各种校验信息，header，继承体系等），不便于在网络中高效传输。所以，hadoop自己开发了一套序列化机制（Writable），精简、高效。  
2）自定义bean对象要想序列化传输步骤及注意事项：  
&emsp; （1）必须实现Writable接口  
&emsp; （2）反序列化时，需要反射调用空参构造函数，所以必须有空参构造  
&emsp; （3）重写序列化方法  
&emsp; （4）重写反序列化方法  
&emsp; （5）注意反序列化的顺序和序列化的顺序完全一致  
&emsp; （6）要想把结果显示在文件中，需要重写toString()，且用"\t"分开，方便后续用  
&emsp; （7）如果需要将自定义的bean放在key中传输，则还需要实现comparable接口，因为mapreduce框中的shuffle过程一定会对key进行排序  

## 2、FileInputFormat切片机制（☆☆☆☆☆）
job提交流程源码详解
waitForCompletion()
submit();
// 1、建立连接
    connect();    
        // 1）创建提交job的代理
        new Cluster(getConfiguration());
            // （1）判断是本地yarn还是远程
            initialize(jobTrackAddr, conf);
// 2、提交job
submitter.submitJobInternal(Job.this, cluster)
    // 1）创建给集群提交数据的Stag路径
    Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);
    // 2）获取jobid ，并创建job路径
    JobID jobId = submitClient.getNewJobID();
    // 3）拷贝jar包到集群
copyAndConfigureFiles(job, submitJobDir);    
    rUploader.uploadFiles(job, jobSubmitDir);
    // 4）计算切片，生成切片规划文件
writeSplits(job, submitJobDir);
    maps = writeNewSplits(job, jobSubmitDir);
        input.getSplits(job);
    // 5）向Stag路径写xml配置文件
writeConf(conf, submitJobFile);
    conf.writeXml(out);
    // 6）提交job,返回提交状态
status = submitClient.submitJob(jobId, submitJobDir.toString(), job.getCredentials());




