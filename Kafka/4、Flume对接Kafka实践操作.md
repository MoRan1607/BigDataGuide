四、Flume对接Kafka
---
&emsp; 一般实际应用中，会通过Flume+Kafka来对产生的数据进行采集，也可以在Kafka中对数据进行一个初步的处理，用于后续Spark或MapReduce的使用，
这个案例主要是实现Flume和Kafka的对接，
#### 1、配置flume(flume-kafka.conf)
```xml
# define
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F -c +0 /opt/module/datas/flume.log
a1.sources.r1.shell = /bin/bash -c

# sink
a1.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
a1.sinks.k1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092,hadoop104:9092
a1.sinks.k1.kafka.topic = first
a1.sinks.k1.kafka.flumeBatchSize = 20
a1.sinks.k1.kafka.producer.acks = 1
a1.sinks.k1.kafka.producer.linger.ms = 1

# channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# bind
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```  

#### 2、启动kafkaIDEA消费者（上一篇文章有）
#### 3、进入flume根目录下，启动flume
```xml
bin/flume-ng agent -c conf/ -n a1 -f jobs/flume-kafka.conf
```  
#### 4、向/opt/module/datas/flume.log里追加数据，查看kafka消费者消费情况
```xml
echo hello >> /opt/module/datas/flume.log
```   


