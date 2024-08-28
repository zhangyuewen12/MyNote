# 问题1. HDFS的基本架构

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20211015095531804.png" alt="image-20211015095531804" style="zoom:50%;" />

```
1)NameNode(nn):就是Master，它
是一个主管、管理者。
(1)管理HDFS的名称空间; (2)配置副本策略; (3)管理数据块(Block)映射信息; (4)处理客户端读写请求。

2)DataNode:就是Slave。NameNode 下达命令，DataNode执行实际的操作。
(1)存储实际的数据块; (2)执行数据块的读/写操作。

3)Client:就是客户端。 
(1)文件切分。文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行上传; 
(2)与NameNode交互，获取文件的位置信息;
(3)与DataNode交互，读取或者写入数据;
(4)Client提供一些命令来管理HDFS，比如NameNode格式化;
(5)Client可以通过一些命令来访问HDFS，比如对HDFS增删查改操作;

4)Secondary NameNode:并非NameNode的热备。当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务。
(1)辅助NameNode，分担其工作量，比如定期合并Fsimage和Edits，并推送给NameNode ; (2)在紧急情况下，可辅助恢复NameNode。
```

```
架构原理
HDFS采用master/slave架构。一个HDFS集群包含一个单独的NameNode和多个DataNode。

NameNode作为master服务，它负责管理文件系统的命名空间和客户端对文件的访问。NameNode会保存文件系统的具体信息，包括文件信息、文件被分割成具体block块的信息、以及每一个block块归属的DataNode的信息。对于整个集群来说，HDFS通过NameNode对用户提供了一个单一的命名空间。

DataNode作为slave服务，在集群中可以存在多个。通常每一个DataNode都对应于一个物理节点。DataNode负责管理节点上它们拥有的存储，它将存储划分为多个block块，管理block块信息，同时周期性的将其所有的block块信息发送给NameNode。
```



## 问题2. HDFS的写数据流程

![image-20211015100014612](/Users/zyw/Library/Application Support/typora-user-images/image-20211015100014612.png)



```
(1)客户端通过 Distributed FileSystem 模块向 NameNode 请求上传文件，NameNode 检 查目标文件是否已存在，父目录是否存在。
(2)NameNode 返回是否可以上传。
(3)客户端请求第一个 Block 上传到哪几个 DataNode 服务器上。
(4)NameNode 返回 3 个 DataNode 节点，分别为 dn1、dn2、dn3。
(5)客户端通过 FSDataOutputStream 模块请求 dn1 上传数据，dn1 收到请求会继续调用dn2，然后 dn2 调用 dn3，将这个通信管道建立完成。
(6)dn1、dn2、dn3 逐级应答客户端。
(7)客户端开始往 dn1 上传第一个 Block(先从磁盘读取数据放到一个本地内存缓存)，
以 Packet 为单位，dn1 收到一个 Packet 就会传给 dn2，dn2 传给 dn3;dn1 每传一个 packet 会放入一个应答队列等待应答。
(8)当一个 Block 传输完成之后，客户端再次请求 NameNode 上传第二个 Block 的服务 器。(重复执行 3-7 步)。
```

> ### Replication Pipelining
>
> When a client is writing data to an HDFS file with a replication factor of three, the NameNode retrieves a list of DataNodes using a replication target choosing algorithm. This list contains the DataNodes that will host a replica of that block. The client then writes to the first DataNode. The first DataNode starts receiving the data in portions, writes each portion to its local repository and transfers that portion to the second DataNode in the list. The second DataNode, in turn starts receiving each portion of the data block, writes that portion to its repository and then flushes that portion to the third DataNode. Finally, the third DataNode writes the data to its local repository. Thus, a DataNode can be receiving data from the previous one in the pipeline and at the same time forwarding data to the next one in the pipeline. Thus, the data is pipelined from one DataNode to the next.
>
> 客户机将数据写入HDFS文件时，其数据首先写入本地文件，如前一节所述。假设HDFS文件的复制因子为3。当本地文件累积完整的用户数据块时，客户端从NameNode检索数据节点列表。此列表包含将承载该块副本的数据节点。然后，客户端将数据块刷新到第一个数据节点。第一个数据节点开始接收小部分的数据，将每个部分写入其本地存储库，并将该部分传输到列表中的第二个数据节点。第二个数据节点依次开始接收数据块的每个部分，将该部分写入其存储库，然后将该部分刷新到第三个数据节点。最后，第三个数据节点将数据写入其本地存储库。因此，数据节点可以从管道中的前一个节点接收数据，同时将数据转发到管道中的下一个节点。因此，数据是从一个数据节点到下一个数据节点的流水线

# 问题3. HDFS读数据流程

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20211015100714978.png" alt="image-20211015100714978" style="zoom:50%;" />

```
(1)客户端通过 DistributedFileSystem 向 NameNode 请求下载文件，NameNode 通过查 询元数据，找到文件块所在的 DataNode 地址。
(2)挑选一台 DataNode(就近原则，然后随机)服务器，请求读取数据。
(3)DataNode 开始传输数据给客户端(从磁盘里面读取数据输入流，以 Packet 为单位 来做校验)。
(4)客户端以 Packet 为单位接收，先在本地缓存，然后写入目标文件。
```

## 问题4. HDFS在写入过程中如何保证packet传输的一致性

