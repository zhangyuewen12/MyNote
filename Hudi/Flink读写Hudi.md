# 1. 写入方式

## 1.1 CDC Ingestion

有两种方式同步数据到Hudi

1. 使用Flink CDC直接将Mysql的binlog日志同步到Hudi
2. 数据先同步到Kafka/Pulsar等消息系统，然后再使用Flink cdc-format将数据同步到Hudi

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221106132419411.png" alt="image-20221106132419411" style="zoom:50%;" />

**注意**：

1. 如果upstream不能保证数据的order，则需要显式指定`write.precombine.field`
2. MOR类型的表，还不能处理delete，所以会导致数据不一致。可以通过`changelog.enabled`转换到change log模式

## 1.2 Bulk Insert

主要用于数据初始化导入。Bulk Insert不会进行数据去重，需要用户在数据插入前进行数据去重

Bulk Insert在batch execution mode下更高效



# 2. 写入模式

## 2.1 Changelog Mode

使用参数如下：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221106132633798.png" alt="image-20221106132633798" style="zoom:50%;" />

保留消息的all changes(I / -U / U / D)，Hudi MOR类型的表将all changes append到file log中，但是compaction会对all changes进行merge。如果想消费all changes，需要调整compaction参数：compaction.delta_commits和 compaction.delta_seconds

Snapshot读取，永远读取merge后的结果数据；



# 3. write写入速率限制

场景：使用Flink消费历史数据 + 实时增量数据，然后写入到Hudi。会造成写入吞吐量巨大 + 写入分区乱序严重，影响集群和application的稳定性。所以需要限制速率

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221106132924180.png" alt="image-20221106132924180" style="zoom:50%;" />

# 4. 读取方式

## 4.1 Streaming Query

默认是Batch query，查询最新的Snapshot

Streaming Query需要设置read.streaming.enabled = true。再设置read.start-commit，如果想消费所以数据，设置值为earliest

使用参数如下

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221106132950682.png" alt="image-20221106132950682" style="zoom:50%;" />

注意：如果开启`read.streaming.skip_compaction`，但[stream](https://so.csdn.net/so/search?q=stream&spm=1001.2101.3001.7020) reader的速度比`clean.retain_commits`慢，可能会造成数据丢失

## 4.2 Incremental Query

有3种使用场景

Streaming query: 设置read.start-commit
Batch query: 同时设置read.start-commit和read.end-commit，start commit和end commit都包含
TimeTravel: 设置read.end-commit为大于当前的一个instant time，read.start-commit默认为latest
使用参数如下：

![image-20221106133115527](/Users/zyw/Library/Application Support/typora-user-images/image-20221106133115527.png)