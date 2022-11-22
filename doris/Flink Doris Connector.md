# Apache Doris 1.1 特性揭秘：Flink 实时写入如何兼顾高吞吐和低延时

## 一、挑战

通常实时数仓要保证端到端高并发以及低延迟，往往面临诸多挑战，比如：

- 如何保证端到端的**秒级别数据同步**？
- 如何快速保证**数据可见性**？
- 在高并发大压力下，如何解决**大量小文件写入**的问题？
- 如何确保端到端的 **Exactly Once** 语义？



## 二、流式写入

Flink Doris Connector 最初的做法是在接收到数据后，缓存到内存 Batch 中，通过攒批的方式进行写入，同时使用 batch.size、batch.interval 等参数来控制 Stream Load 写入的时机。

这种方式通常在参数合理的情况下可以稳定运行，一旦参数不合理导致频繁的 Stream Load，便会引发 Compaction 不及时，从而导致 version 过多的错误(-235)；其次，当数据过多时，为了减少 Stream Load 的写入时机，batch.size 过大的设置还可能会引发 Flink 任务的 OOM。

为了解决这个问题，**我们引入了流式写入****：**

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220812095153822.png" alt="image-20220812095153822" style="zoom:50%;" />

> 1. Flink 任务启动后，会异步发起一个 Stream Load 的 Http 请求。
>
> 2. 接收到实时数据后，通过 Http 的分块传输编码(Chunked transfer encoding)机制持续向 Doris 传输数据。
>
> 3. 在 Checkpoint 时结束 Http 请求，完成本次 Stream Load 写入，同时异步发起下一次 Stream Load 的请求。
>
> 4. 继续接收实时数据，后续流程同上。



**由于采用 Chunked 机制传输数据，就避免了攒批对内存的压力，同时将写入的时机和 Checkpoint 绑定起来，使得 Stream Load 的时机可控，并且为下面的 Exactly-Once 语义提供了基础。**



## 三、Exactly-ONECE

Exactly-Once 语义是指即使在机器或应用出现故障的情况下，也不会重复处理数据或者丢失数据。Flink 很早就支持 End-to-End 的 Exactly-Once 场景，主要是通过两阶段提交协议来实现 Sink 算子的 Exactly-Once 语义。

在 Flink 两阶段提交的基础上，同时借助 Doris 1.0 的 Stream Load 两阶段提交，**Flink Doris Connector 实现了 Exactly Once 语义，具体原理如下：**

1. Flink 任务在启动的时候，会发起一个 Stream Load 的 Prepare 请求，此时会先开启一个事务，同时会通过 Http 的 Chunked 机制将数据持续发送到 Doris。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220812095409609.png" alt="image-20220812095409609" style="zoom:50%;" />



2. 在 Checkpoint 时，结束数据写入，同时完成 Http 请求，并且将事务状态设置为预提交(PreCommitted)，此时数据已经写入 BE，对用户不可见。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220812095452416.png" alt="image-20220812095452416" style="zoom:50%;" />

3. Checkpoint 完成后，发起 Commit 请求，并且将事务状态设置为提交(Committed)，完成后数据对用户可见。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220812095535372.png" alt="image-20220812095535372" style="zoom:50%;" />

4. Flink 应用意外挂掉后，从 Checkpoint 重启时，若上次事务为预提交(PreCommitted)状态，则会发起回滚请求，并且将事务状态设置为 Aborted。

**基于此，可以借助 Flink Doris Connector 实现数据实时入库时数据不丢不重。**



