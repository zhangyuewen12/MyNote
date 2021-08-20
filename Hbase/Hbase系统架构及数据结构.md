# 一、基本概念
一个典型的 Hbase Table 表如下：
![](https://camo.githubusercontent.com/38a2568a5134500683222d4b3c4a8b25da28c9387ee5b1e886eb89bdff77392d/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f68626173652d7765627461626c652e706e67)

## 1.1 Row Key (行键)
Row Key 是用来检索记录的主键。想要访问 HBase Table 中的数据，只有以下三种方式：

通过指定的 Row Key 进行访问；

通过 Row Key 的 range 进行访问，即访问指定范围内的行；

进行全表扫描。

Row Key 可以是任意字符串，存储时数据按照 Row Key 的字典序进行排序。这里需要注意以下两点：

因为字典序对 Int 排序的结果是 1,10,100,11,12,13,14,15,16,17,18,19,2,20,21,…,9,91,92,93,94,95,96,97,98,99。如果你使用整型的字符串作为行键，那么为了保持整型的自然序，行键必须用 0 作左填充。

行的一次读写操作时原子性的 (不论一次读写多少列)。

## 1.2 Column Family（列族）
HBase 表中的每个列，都归属于某个列族。列族是表的 Schema 的一部分，所以列族需要在创建表时进行定义。列族的所有列都以列族名作为前缀，例如 courses:history，courses:math 都属于 courses 这个列族。

## 1.3 Column Qualifier (列限定符)
列限定符，你可以理解为是具体的列名，例如 courses:history，courses:math 都属于 courses 这个列族，它们的列限定符分别是 history 和 math。需要注意的是列限定符不是表 Schema 的一部分，你可以在插入数据的过程中动态创建列。

## 1.4 Column(列)
HBase 中的列由列族和列限定符组成，它们由 :(冒号) 进行分隔，即一个完整的列名应该表述为 列族名 ：列限定符。

## 1.5 Cell
Cell 是行，列族和列限定符的组合，并包含值和时间戳。你可以等价理解为关系型数据库中由指定行和指定列确定的一个单元格，但不同的是 HBase 中的一个单元格是由多个版本的数据组成的，每个版本的数据用时间戳进行区分。

## 1.6 Timestamp(时间戳)
HBase 中通过 row key 和 column 确定的为一个存储单元称为 Cell。每个 Cell 都保存着同一份数据的多个版本。版本通过时间戳来索引，时间戳的类型是 64 位整型，时间戳可以由 HBase 在数据写入时自动赋值，也可以由客户显式指定。每个 Cell 中，不同版本的数据按照时间戳倒序排列，即最新的数据排在最前面。

# 二、存储结构

## 2.1 Regions
HBase Table 中的所有行按照 Row Key 的字典序排列。HBase Tables 通过行键的范围 (row key range) 被水平切分成多个 Region, 一个 Region 包含了在 start key 和 end key 之间的所有行。
![](https://camo.githubusercontent.com/d8e79c670f2e6e2ce10b9f16272bae9e108ae3063c818fc767351efa6ff22bca/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f48426173654172636869746563747572652d426c6f672d466967322e706e67)
每个表一开始只有一个 Region，随着数据不断增加，Region 会不断增大，当增大到一个阀值的时候，Region 就会等分为两个新的 Region。当 Table 中的行不断增多，就会有越来越多的 Region。
![](https://camo.githubusercontent.com/98959e8cbea4184cf7aabce8a49a4d3a40a07080448ab944ce5065e3b4c5ca6d/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f68626173652d726567696f6e2d73706c6974652e706e67)
**Region 是 HBase 中分布式存储和负载均衡的最小单元**。这意味着不同的 Region 可以分布在不同的 Region Server 上。但一个 Region 是不会拆分到多个 Server 上的。
![](https://camo.githubusercontent.com/cf115e8419e317a5143ec2c8f0046af0fe3221afef45393dbfd9fe3d22d74f4a/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f68626173652d726567696f6e2d6469732e706e67)

## 2.2 Region Server
Region Server 运行在 HDFS 的 DataNode 上。它具有以下组件：

* WAL(Write Ahead Log，预写日志)：用于存储尚未进持久化存储的数据记录，以便在发生故障时进行恢复。
* BlockCache：读缓存。它将频繁读取的数据存储在内存中，如果存储不足，它将按照 最近最少使用原则 清除多余的数据。
* MemStore：写缓存。它存储尚未写入磁盘的新数据，并会在数据写入磁盘之前对其进行排序。每个 Region 上的每个列族都有一个 MemStore。
* HFile ：将行数据按照 Key\Values 的形式存储在文件系统上。

![](https://camo.githubusercontent.com/dcbb62063e882187a7a98fb0a2b90f7497b904dfdb821f220dae0e6bf4e21bdb/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f68626173652d526567696f6e2d5365727665722e706e67)
Region Server 存取一个子表时，会创建一个 Region 对象，然后对表的每个列族创建一个 Store 实例，每个 Store 会有 0 个或多个 StoreFile 与之对应，每个 StoreFile 则对应一个 HFile，HFile 就是实际存储在 HDFS 上的文件。
![](https://camo.githubusercontent.com/7a99de45089bde39ccf787d476535e52ac152c383a70251a6cc88b914ed7a497/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f68626173652d6861646f6f702e706e67)

# 三、Hbase系统架构
## 3.1 系统架构
HBase 系统遵循 Master/Salve 架构，由三种不同类型的组件组成：

Zookeeper

1. 保证任何时候，集群中只有一个 Master；

1. 存贮所有 Region 的寻址入口；

1. 实时监控 Region Server 的状态，将 Region Server 的上线和下线信息实时通知给 Master；

1. 存储 HBase 的 Schema，包括有哪些 Table，每个 Table 有哪些 Column Family 等信息。

Master

1. 为 Region Server 分配 Region ；

1. 负责 Region Server 的负载均衡 ；

1. 发现失效的 Region Server 并重新分配其上的 Region；

1. GFS 上的垃圾文件回收；

1. 处理 Schema 的更新请求。

Region Server

1. Region Server 负责维护 Master 分配给它的 Region ，并处理发送到 Region 上的 IO 请求；

1. Region Server 负责切分在运行过程中变得过大的 Region。

![](https://camo.githubusercontent.com/0caea88f17562a6ba6fc07ea40b898fec9b9457e90f265019e9c2bcca6989ff7/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f48426173654172636869746563747572652d426c6f672d466967312e706e67)

## 3.2 组件间的协作
HBase 使用 ZooKeeper 作为分布式协调服务来维护集群中的服务器状态。 Zookeeper 负责维护可用服务列表，并提供服务故障通知等服务：

* 每个 Region Server 都会在 ZooKeeper 上创建一个临时节点，Master 通过 Zookeeper 的 Watcher 机制对节点进行监控，从而可以发现新加入的 Region Server 或故障退出的 Region Server；

* 所有 Masters 会竞争性地在 Zookeeper 上创建同一个临时节点，由于 Zookeeper 只能有一个同名节点，所以必然只有一个 Master 能够创建成功，此时该 Master 就是主 Master，主 Master 会定期向 Zookeeper 发送心跳。备用 Masters 则通过 Watcher 机制对主 HMaster 所在节点进行监听；

* 如果主 Master 未能定时发送心跳，则其持有的 Zookeeper 会话会过期，相应的临时节点也会被删除，这会触发定义在该节点上的 Watcher 事件，使得备用的 Master Servers 得到通知。所有备用的 Master Servers 在接到通知后，会再次去竞争性地创建临时节点，完成主 Master 的选举。

![](https://camo.githubusercontent.com/34e8be433817f41ae3965c5841f115b4cf0c4a6b840da1cc5d55066da094cdfe/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f48426173654172636869746563747572652d426c6f672d466967352e706e67)

# 四、数据的读写流程简述
## 4.1 写入数据的流程
1. Client 向 Region Server 提交写请求；

1. Region Server 找到目标 Region；

1. Region 检查数据是否与 Schema 一致；

1. 如果客户端没有指定版本，则获取当前系统时间作为数据版本；

1. 将更新写入 WAL Log；

1. 将更新写入 Memstore；

1. 判断 Memstore 存储是否已满，如果存储已满则需要 flush 为 Store Hfile 文件。
## 4.2 读取数据的流程
以下是客户端首次读写 HBase 上数据的流程：

1. 客户端从 Zookeeper 获取 META 表所在的 Region Server；

1. 客户端访问 META 表所在的 Region Server，从 META 表中查询到访问行键所在的 Region Server，之后客户端将缓存这些信息以及 META 表的位置；

1. 客户端从行键所在的 Region Server 上获取数据。

1. 如果再次读取，客户端将从缓存中获取行键所在的 Region Server。这样客户端就不需要再次查询 META 表，除非 Region 移动导致缓存失效，这样的话，则将会重新查询并更新缓存。

注：META 表是 HBase 中一张特殊的表，它保存了所有 Region 的位置信息，META 表自己的位置信息则存储在 ZooKeeper 上
![](https://camo.githubusercontent.com/4f6e720ac62f21162f45d95d4e7ac2bdca27ce6c47fd85e9ef02083bcb7670c1/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f48426173654172636869746563747572652d426c6f672d466967372e706e67)