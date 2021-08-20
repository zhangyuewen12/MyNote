# Kafka Connect

Kafaka connect 是一种用于在Kafka和其他系统之间可扩展的、可靠的流式传输数据的工具。它使得能够快速定义将大量数据集合移入和移出Kafka的连接器变得简单。Kafka Connect可以从数据库或应用程序服务器收集数据到Kafka topic，使数据可用于低延迟的流处理。导出作业可以将数据从Kafka topic传输到二次存储和查询系统，或者传递到批处理系统以进行离线分析。

## **Kafaka connect的核心组件：**

**Source：**负责将外部数据写入到kafka的topic中。
**Sink：**负责从kafka中读取数据到自己需要的地方去，比如读取到HDFS，hbase等。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210507182214121.png" alt="image-20210507182214121" style="zoom:50%;" />

在Kafka Connect中有两个重要的概念：Task和Work。

Task是Kafka Connect数据模型的主角，每一个Connector都会协调一系列的Task去执行任务，Connector 可以把一项工作分割成许多task,然后把task分发到各个work进程中去执行（分布式模型下），Task不保存自己的状态信息，而是交给特定的Kafka主题去保存。Connector和Task都是逻辑工作单位，必须在进程中运行，这个进程就是worker。



**Connectors ：**通过管理任务来协调数据流的高级抽象

**Tasks：**数据写入kafk和从kafka中读出数据的具体实现，source和sink使用时都需要Task

**Workers：**运行connectors和tasks的进程。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210507183127961.png" alt="image-20210507183127961" style="zoom:50%;" />

**Converters：**kafka connect和其他存储系统直接发送或者接受数据之间转换数据。

converter会把bytes数据转换成kafka connect内部的格式，也可以把kafka connect内部存储格式的数据转变成bytes，converter对connector来说是解耦的，所以其他的connector都可以重用，例如，使用了avro converter，那么jdbc connector可以写avro格式的数据到kafka，当然，hdfs connector也可以从kafka中读出avro格式的数据。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210507183031761.png" alt="image-20210507183031761" style="zoom:50%;" />

**Transforms：**一种轻量级数据调整的工具。



## **Kafka connect 工作模式：**

**Kafka connect 有两种工作模式：**
**standalone：**在standalone模式中，所有的worker都在一个独立的进程中完成。

**distributed：**distributed模式具有高扩展性，以及提供自动容错机制。你可以使用一个group.ip来启动很多worker进程，在有效的worker进程中它们会自动的去协调执行connector和task，如果你新加了一个worker或者挂了一个worker，其他的worker会检测到然后在重新分配connector和task。

### standalone 模式 demo：

将数据从一个文件读取，写到另一个文件。

需要两个配置文件: 一个用于Worker进程运行的相关配置文件，另一个是执行Source连接器和Sink连接器的配置文件，可以同时指定多个连接器配置，每个连接器配置文件对应一个连接器。因此要保证连接器名称全局唯一，连接器名称通过name参数指定。

1. 首先修改Worker进程运行的配置文件$Kafka_home/config/connect-standalone.properties

   bootstrap.servers=localhost:9092

   

   \# The converters specify the format of data in Kafka and how to translate it into Connect data. Every Connect user will

   \# need to configure these based on the format they want their data in when loaded from or stored into Kafka

   key.converter=org.apache.kafka.connect.json.JsonConverter

   value.converter=org.apache.kafka.connect.json.JsonConverter

   

   \# Converter-specific settings can be passed in by prefixing the Converter's setting with the converter we want to apply it to

   key.converter.schemas.enable=true

   value.converter.schemas.enable=true

   #用来指定保存偏移量的文件路径

   offset.storage.file.filename=/tmp/connect.offsets

   offset.flush.interval.ms=10000

2. 修改Source连接器的配置文件$Kafka_home/config/connect-file.source.properties

   #连接器名称

   name=local-file-source

   #连接器类名

   connector.class=FileStreamSource

   #Task数量

   tasks.max=1

   #指定该连接器数据源文件路径

   file=/Users/zyw/software/kafka/test.txt

   #设置连接器把数据导入哪个主题，如果不存在会自动创建。

   topic=connect-test

3. 创建topic

   ```shell
   zywdeMacBook-Pro:kafka zyw$ bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic connect-test --replication-factor 1 --partitions 1
   Created topic connect-test.
   ```

   

4. 启动Source连接器

   ```she
   zywdeMacBook-Pro:kafka zyw$ bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties 
   ```

5. 查看传入到Kafka的消息

   ```she
   zywdeMacBook-Pro:kafka zyw$ bin/kafka-dump-log.sh --files /tmp/kafka-logs/connect-test-0/00000000000000000000.log --print-data-log
   
   Dumping /tmp/kafka-logs/connect-test-0/00000000000000000000.log
   Starting offset: 0
   baseOffset: 0 lastOffset: 0 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 0 CreateTime: 1620385172643 size: 130 magic: 2 compresscodec: NONE crc: 1788838844 isvalid: true
   | offset: 0 CreateTime: 1620385172643 keysize: -1 valuesize: 61 sequence: -1 headerKeys: [] payload: {"schema":{"type":"string","optional":false},"payload":"sdf"}
   baseOffset: 1 lastOffset: 1 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 130 CreateTime: 1620385299151 size: 149 magic: 2 compresscodec: NONE crc: 2591062069 isvalid: true
   | offset: 1 CreateTime: 1620385299151 keysize: -1 valuesize: 79 sequence: -1 headerKeys: [] payload: {"schema":{"type":"string","optional":false},"payload":"hello kafka connector"}
   ```

   

### 分布式模式 demo

与独立模式不同，分布式模式天然集成了kafka提供的负载均衡和故障转移功能，能够自动在多个节点机器上平衡负载。不过以分布式模式启动的连接器并不支持在启动时通过加载连接器配置文件来创建一个连接器，只能通过访问rest api来创建连接器。

1. 同样要修改Worker进程的相关配置文件 $Kafka_home/config/connect-distributed.properties

   bootstap.servers =localhost1:9092,localhost2:9092..

   ····

2. 发送rest api 请求来创建连接器

## kafka connect rest api

kafka connect的目的是作为一个服务运行，默认情况下，此服务运行于端口8083。它支持rest管理，用来获取 Kafka Connect 状态，管理 Kafka Connect 配置，Kafka Connect 集群内部通信，常用命令如下：



GET /connectors 返回一个活动的connect列表
POST /connectors 创建一个新的connect；请求体是一个JSON对象包含一个名称字段和连接器配置参数

GET /connectors/{name} 获取有关特定连接器的信息
GET /connectors/{name}/config 获得特定连接器的配置参数
PUT /connectors/{name}/config 更新特定连接器的配置参数
GET /connectors/{name}/tasks 获得正在运行的一个连接器的任务的列表

DELETE /connectors/{name} 删除一个连接器，停止所有任务，并删除它的配置

GET /connectors 返回一个活动的connect列表

POST /connectors 创建一个新的connect；请求体是一个JSON对象包含一个名称字段和连接器配置参数

GET /connectors/{name} 获取有关特定连接器的信息
GET /connectors/{name}/config 获得特定连接器的配置参数
PUT /connectors/{name}/config 更新特定连接器的配置参数
GET /connectors/{name}/tasks 获得正在运行的一个连接器的任务的列表

DELETE /connectors/{name} 删除一个连接器，停止所有任务，并删除它的配置





```
docker run -it --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my-connect-configs -e OFFSET_STORAGE_TOPIC=my-connect-offsets -e ADVERTISED_HOST_NAME=$(echo $DOCKER_HOST | cut -f3  -d'/' | cut -f1 -d':') -e BOOTSTRAP_SERVERS=172.31.81.0:9092 debezium/connect
```