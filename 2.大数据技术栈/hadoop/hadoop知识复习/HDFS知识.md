# HDFS基本原理

HDFS是Hadoop的分布式文件系统（Hadoop Distributed File System），实现大规模数据可靠的分布式读写。**HDFS针对的使用场景是数据读写具有“一次写，多次读”的特征，而数据“写”操作是顺序写，也就是在文件创建时的写入或者在现有文件之后的添加操作。HDFS保证一个文件在一个时刻只被一个调用者执行写操作，而可以被多个调用者执行读操作。**

## HDFS结构

HDFS包含主、备NameNode和多个DataNode，如[图1](https://support.huaweicloud.com/productdesc-mrs/mrs_08_000701.html#mrs_08_000701__fig1245232216814)所示。

HDFS是一个Master/Slave的架构，在Master上运行NameNode，而在每一个Slave上运行DataNode，ZKFC需要和NameNode一起运行。

NameNode和DataNode之间的通信都是建立在TCP/IP的基础之上的。NameNode、DataNode、ZKFC和JournalNode能部署在运行Linux的服务器上。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221006172256793.png" alt="image-20221006172256793" style="zoom:40%;" />

| **名称**       | **描述**                                                     |
| -------------- | ------------------------------------------------------------ |
| NameNode       | 用于管理文件系统的命名空间、目录结构、元数据信息以及提供备份机制等，分为：  <br>Active NameNode：管理文件系统的命名空间、维护文件系统的目录结构树以及元数据信息；记录写入的每个“数据块”与其归属文件的对应关系。  <br> Standby NameNode：与Active NameNode中的数据保持同步；随时准备在Active NameNode出现异常时接管其服务。<br>Observer NameNode：与Active NameNode中的数据保持同步，处理来自客户端的读请求。 |
| DataNode       | 用于存储每个文件的“数据块”数据，并且会周期性地向NameNode报告该DataNode的数据存放情况。 |
| JournalNode    | HA集群下，用于同步主备NameNode之间的元数据信息。             |
| ZKFC           | ZKFC是需要和NameNode一一对应的服务，即每个NameNode都需要部署ZKFC。它负责监控NameNode的状态，并及时把状态写入ZooKeeper。ZKFC也有选择谁作为Active NameNode的权利。 |
| ZK Cluster     | ZooKeeper是一个协调服务，帮助ZKFC执行主NameNode的选举。      |
| HttpFS gateway | HttpFS是个单独无状态的gateway进程，对外提供webHDFS接口，对HDFS使用FileSystem接口对接。可用于不同Hadoop版本间的数据传输，及用于访问在防火墙后的HDFS（HttpFS用作gateway）。 |

- HDFS HA架构

  HA即为High Availability，用于解决NameNode单点故障问题，该特性通过主备的方式为主NameNode提供一个备用者，一旦主NameNode出现故障，可以迅速切换至备NameNode，从而不间断对外提供服务。

  在一个典型HDFS HA场景中，通常由两个NameNode组成，一个处于Active状态，另一个处于Standby状态。

  为了能实现Active和Standby两个NameNode的元数据信息同步，需提供一个共享存储系统。本版本提供基于QJM（Quorum Journal Manager）的HA解决方案，如[图2](https://support.huaweicloud.com/productdesc-mrs/mrs_08_000701.html#mrs_08_000701__fig1517714182104)所示。主备NameNode之间通过一组JournalNode同步元数据信息。

  通常配置奇数个（2N+1个）JournalNode，且最少要运行3个JournalNode。这样，一条元数据更新消息只要有N+1个JournalNode写入成功就认为数据写入成功，此时最多容忍N个JournalNode写入失败。比如，3个JournalNode时，最多允许1个JournalNode写入失败，5个JournalNode时，最多允许2个JournalNode写入失败。

  由于JournalNode是一个轻量级的守护进程，可以与Hadoop其它服务共用机器。建议将JournalNode部署在控制节点上，以避免数据节点在进行大数据量传输时引起JournalNode写入失败。

图2 基于QJM的HDFS架构

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221006173033568.png" alt="image-20221006173033568" style="zoom:50%;" />



## HDFS原理

MRS使用HDFS的副本机制来保证数据的可靠性，HDFS中每保存一个文件则自动生成1个备份文件，即共2个副本。HDFS副本数可通过**“dfs.replication”**参数查询。

- 当MRS集群中Core节点规格选择为非本地盘（hdd）时，若集群中只有一个Core节点，则HDFS默认副本数为1。若集群中Core节点数大于等于2，则HDFS默认副本数为2。
- 当MRS集群中Core节点规格选择为本地盘（hdd）时，若集群中只有一个Core节点，则HDFS默认副本数为1。若集群中有两个Core节点，则HDFS默认副本数为2。若集群中Core节点数大于等于3，则HDFS默认副本数为3。

图3 HDFS架构

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221006173200912.png" alt="image-20221006173200912" style="zoom:50%;" />





NameNode保存HDFS的命名空间。

NameNode在内存中保存着整个文件系统的命名空间和文件数据块映射的映射。

Datanode将HDFS数据以文件的形式存储在本地的文件系统中。

文件写入流程：

1. 客户端向NameNode发送文件写入请求；
2. NameNode根据文件和文件块的配置情况，返回给客户端所管理部分DataNode信息。
3. 客户端将文件划分为多个快，根据DataNode的地址信息，按顺序写入DataNode中。



## HDFS健壮性

1. 磁盘数据错误，心跳检测和重复复制。

   每个DataNode节点周期性向NameNode发送心跳信息。

2. 集群均衡

   HDFS的架构支持数据均衡策略。

3. 数据完整性

   ```
   从某个DataNode获取的数据块可能是损坏的，损坏可能是由DataNode的存储设备错误、网络错误等造成的。HDFS客户端实现了对HDFS文件内容的校验和检查。当客户端创建一个新的HDFS文件时，会计算这个文件每个数据块的校验和，并将校验和作为一个单独的隐藏文件保存在同一个HDFS命名空间下。当客户端获取文件后，他会检查从DataNode读取的数据和相应的校验和与文件中的检验和是否匹配。如果不匹配，客户端从其他副本上读取。
   
   客户端向HDFS写数据的时候
   1、假设客户端发送2KB的数据
   2、客户端会以字节的方式往datanode发送，所以客户端会计算发送的数据有多少个，而这个单位就是chunk，它一般情况是512字节，也就是说，每512字节就称为一个chunk。
   3、客户端可以计算出checksum值，checksum = 2KB/512B=4
   4、然后datanode接收客户端发送来的数据，每接收512B的数据，就让checksum的值+1
   5、最后比较客户端和datanade的checksum值
   
   DataNode读取block块的时候
   1、block创建的时候会有一个初始的checksum值
   2、DataNode每隔一段时间就会计算block新的checksum值，看block块是否已经丢失
   3、如果checksum和之前一样，则没丢失，和之前比出现了不一样，那就说明数据丢失（或者异常）
   4、当发生异常的时候，DateNode会报告给NameNode，NameNode会发送一条命令，清除这个异常块，然后找到这个块对应的副本，将完整的副本复制给其他的DataNode节点
   ```

4. 元数据磁盘错误

5. 快照

   快照支持某一特定时刻的数据的复制备份。利用快照，可以让HDFS在数据损坏时恢复到过去一个正确的时间点。





## HDFS数据组织