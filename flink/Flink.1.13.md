# 重要特性

## 被动扩缩容

Flink 项目的一个初始目标，就是希望流处理应用可以像普通应用一样简单和自然， 被动扩缩容是 Flink 针对这一目标上的最新进展。

当考虑资源管理和部分的时候，Flink 有两种可能的模式。用户可以将 Flink 应用部署 到 k8s、yarn 等资源管理系统之上，并且由 Flink 主动的来管理资源并按需分配和释 放资源。这一模式对于经常改变资源需求的作业和应用非常有用，比如批作业和实时 SQL 查询。在这种模式下，Flink 所启动的 Worker 数量是由应用设置的并发度决定的。 在 Flink 中我们将这一模式叫做主动扩缩容。

对于长时间运行的流处理应用，一种更适合的模型是用户只需要将作业像其它的长 期运行的服务一样启动起来，而不需要考虑是部署在 k8s、yarn 还是其它的资源管 理平台上，并且不需要考虑需要申请的资源的数量。相反，它的规模是由所分配的 worker 数量来决定的。当 worker 数量发生变化时，Flink 自动的改动应用的并发度。 在 Flink 中我们将这一模式叫做被动扩缩容。

Flink 的 Application 部署模式 [4] 开启了使 Flink 作业更接近普通应用（即启动 Flink 作业不需要执行两个独立的步骤来启动集群和提交应用）的努力，而被动扩缩容完成 了这一目标：用户不再需要使用额外的工具（如脚本、K8s 算子）来让 Worker 的数 量与应用并发度设置保持一致。

用户现在可以将自动扩缩容的工具应用到 Flink 应用之上，就像普通的应用程序一样， 只要用户了解扩缩容的代价：有状态的流应用在扩缩容的时候需要将状态重新分发。

如果想要尝试被动扩缩容，用户可以增加 scheduler-mode: reactive 这一配置项，然 后启动一个应用集群（Standalone[5] 或者 K8s[6]）。更多细节见被动扩缩容的文档 [7]。

## 分析应用的性能

对所有应用程序来说，能够简单的分析和理解应用的性能是非常关键的功能。这一 功能对 Flink 更加重要，因为 Flink 应用一般是数据密集的（即需要处理大量的数据） 并且需要在（近）实时的延迟内给出结果。

当 Flink 应用处理的速度跟不上数据输入的速度时，或者当一个应用占用的资源超过 预期，下文介绍的这些工具可以帮你分析原因。

### 瓶颈检测与反压监控

Flink 性能分析首先要解决的问题经常是：哪个算子是瓶颈？ 为了回答这一问题，Flink 引入了描述作业繁忙（即在处理数据）与反压（由于下游 算子不能及时处理结果而无法继续输出）程度的指标。应用中可能的瓶颈是那些繁忙 并且上游被反压的算子。

 Flink 1.13 优化了反压检测的逻辑（使用基于任务 Mailbox 计时，而不在再于堆栈采 样），并且重新实现了作业图的 UI 展示：Flink 现在在 UI 上通过颜色和数值来展示繁 忙和反压的程度。



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210712132917840.png" alt="image-20210712132917840" style="zoom:50%;" />



### Web UI 中的 CPU 火焰图

Flink 关于性能另一个经常需要回答的问题：瓶颈算子中的哪部分计算逻辑消耗巨大？

针对这一问题，一个有效的可视化工具是火焰图。

它可以帮助回答以下问题：

哪个方法调用现在在占用 CPU ？

不同方法占用 CPU 的比例如何？

一个方法被调用的栈是什么样子的？

火焰图是通过重复采样线程的堆栈来构建的。在火焰图中，每个方法调用被表示为一 个矩形，矩形的长度与这个方法出现在采样中的次数成正比。火焰图在 UI 上的一个 例子如下图所示。



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210712133107718.png" alt="image-20210712133107718" style="zoom:50%;" />

##  State 访问延迟指标

另一个可能的性能瓶颈是 state backend，尤其是当作业的 state 超过内存容量而必 须使用 RocksDB state backend[9] 时。

这里并不是想说 RocksDB 性能不够好（我们非常喜欢 RocksDB ！），但是它需要满 足一些条件才能达到最好的性能。

例如，用户可能很容易遇到非故意的在云上由于 使用了错误的磁盘资源类型而不能满足 RockDB 的 IO 性能需求 [10] 的问题。 

基于 CPU 火焰图，新的 State Backend 的延迟指标可以帮助用户更好的判断性能不 符合预期是否是由 State Backend 导致的。例如，如果用户发现 RocksDB 的单次访 问需要几毫秒的时间，那么就需要查看内存和 I/O 的配置。这些指标可以通过设置 state.backend.rocksdb.latency-track-enabled 这一选项来启用。

这些指标是通过采 样的方式来监控性能的，所以它们对 RocksDB State Backend 的性能影响是微不足道 的。

### 通过 Savepoint 来切换 State Backend

用户现在可以在从一个 Savepoint 重启时切换一个 Flink 应用的 State Backend。

这 使得 Flink 应用不再被限制只能使用应用首次运行时选择的 State Backend。 

基于这一功能，用户现在可以首先使用一个 HashMap State Backend（纯内存的 State Backend），如果后续状态变得过大的话，就切换到 RocksDB State Backend 中。

 **在实现层，Flink 现在统一了所有 State Backend 的 Savepoint 格式来实现这一功能**。



### K8s 部署时使用用户指定的 Pod 模式 

原生 kubernetes 部署 [11]（Flink 主动要求 K8s 来启动 Pod）中，现在可以使用自定 义的 Pod 模板。 

使用这些模板，用户可以使用一种更符合 K8s 的方式来设置 JM 和 TM 的 Pod，这种 方式比 Flink K8s 集成内置的配置项更加灵活。



## SQL / Table API 进展

在流式 SQL 查询中，一个最经常使用的是定义时间窗口。Apache Flink 1.13 中引入 了一种新的定义窗口的方式：通过 Table-valued 函数。这一方式不仅有更强的表达 能力（允许用户定义新的窗口类型），并且与 SQL 标准更加一致。

Apache Flink 1.13 在新的语法中支持 TUMBLE 和 HOP 窗口，在后续版本中也会支持 SESSION 窗口。我们通过以下两个例子来展示这一方法的表达能力：

例 1：一个新引入的 CUMULATE 窗口函数，它可以支持按特定步长扩展的窗口， 直到达到最大窗口大小：

```sql
SELECT window_time, window_start, window_end, SUM(price) AS total_price
 FROM TABLE(CUMULATE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '2' MINUTES, INTERVAL '10'
MINUTES))
GROUP BY window_start, window_end, window_time;
```

例 2：用户在 table-valued 窗口函数中可以访问窗口的起始和终止时间，从而使 用户可以实现新的功能。例如，除了常规的基于窗口的聚合和 Join 之外，用户现 在也可以实现基于窗口的 Top-K 聚合：

```sql
SELECT window_time, ...
 FROM (
 SELECT *, ROW_NUMBER() OVER (PARTITION BY window_start, window_end ORDER BY total_price
DESC)
 as rank
 FROM t
 ) WHERE rank <= 100;
```

### 提高 DataStream API 与 Table API / SQL 的互操作能力

这一版本极大的简化了 DataStream API 与 Table API 混合的程序。

 Table API 是一种非常方便的应用开发接口，因为这仅支持表达式的程序编写并提供 了大量的内置函数。但是有时候用户也需要切换回 DataStream，例如当用户存在表 达能力、灵活性或者 State 访问的需求时。 

Flink 新引入的 StreamTableEnvironment.toDataStream()/.fromDataStream() 可以 将一个 DataStream API 声明的 Source 或者 Sink 当作 Table 的 Source 或者 Sink 来使 用。主要的优化包括：

- DataStream 与 Table API 类型系统的自动转换。

- Event Time 配置的无缝集成，Watermark 行为的高度一致性。

- Row 类 型（ 即 Table API 中 数 据 的 表 示） 有 了 极 大 的 增 强， 包 括 toString() / hashCode() 和 equals() 方法的优化，按名称访问字段值的支持与稀疏表示的支持。

```java
Table table = tableEnv.fromDataStream(
 dataStream,
 Schema.newBuilder()
 .columnByMetadata("rowtime", "TIMESTAMP(3)")
 .watermark("rowtime", "SOURCE_WATERMARK()")
 .build());
DataStream<Row> dataStream = tableEnv.toDataStream(table)
 .keyBy(r -> r.getField("user"))
 .window(...);
```

### SQL Client: 初始化脚本和语句集合 （Statement Sets） 

SQL Client 是一种直接运行和部署 SQL 流或批作业的简便方式，用户不需要编写代码 就可以从命令行调用 SQL，或者作为 CI / CD 流程的一部分。

 这个版本极大的提高了 SQL Client 的功能。现在基于所有通过 Java 编程（即通过编 程的方式调用 TableEnvironment 来发起查询）可以支持的语法，现在 SQL Client 和 SQL 脚本都可以支持。

这意味着 SQL 用户不再需要添加胶水代码来部署他们的 SQL 作业.

####  配置简化和代码共享

Flink 后续将不再支持通过 Yaml 的方式来配置 SQL Client（注：目前还在支持，但是 已经被标记为废弃）。作为替代，SQL Client 现在支持使用一个初始化脚本在主 SQL 脚本执行前来配置环境。

这些初始化脚本通常可以在不同团队 / 部署之间共享。它可以用来加载常用的 catalog，应用通用的配置或者定义标准的视图。

```
./sql-client.sh -i init1.sql init2.sql -f sqljob.sql
```

#### 更多的配置项 

通过增加配置项，优化 SET / RESET 命令，用户可以更方便的在 SQL Client 和 SQL 脚 本内部来控制执行的流程

#### 通过语句集合来支持多查询

多查询允许用户在一个 Flink 作业中执行多个 SQL 查询（或者语句）。这对于长期运 行的流式 SQL 查询非常有用。

语句集可以用来将一组查询合并为一组同时执行。

以下是一个可以通过 SQL Client 来执行的 SQL 脚本的例子。它初始化和配置了执行 多查询的环境。这一脚本包括了所有的查询和所有的环境初始化和配置的工作，从而 使它可以作为一个自包含的部署组件。

```
-- set up a catalog
CREATE CATALOG hive_catalog WITH ('type' = 'hive');
USE CATALOG hive_catalog;

-- or use temporary objects
CREATE TEMPORARY TABLE clicks (
 user_id BIGINT,
 page_id BIGINT,
 viewtime TIMESTAMP) WITH (
 'connector' = 'kafka',
 'topic' = 'clicks',
 'properties.bootstrap.servers' = '...',
 'format' = 'avro'
);

-- set the execution mode for jobs
SET execution.runtime-mode=streaming;

-- set the sync/async mode for INSERT INTOs
SET table.dml-sync=false;

-- set the job's parallelism
SET parallism.default=10;

-- set the job name
SET pipeline.name = my_flink_job;

-- restore state from the specific savepoint path
SET execution.savepoint.path=/tmp/flink-savepoints/savepoint-bb0dab;

BEGIN STATEMENT SET;

INSERT INTO pageview_pv_sink
SELECT page_id, count(1) FROM clicks GROUP BY page_id;


INSERT INTO pageview_uv_sink
SELECT page_id, count(distinct user_id) FROM clicks GROUP BY page_id;

END;
```



#### Hive 查询语法兼容性 

用户现在在 Flink 上也可以使用 Hive SQL 语法。

除了 Hive DDL 方言之外，Flink 现在 也支持常用的 Hive DML 和 DQL 方言。 

为了使用 Hive SQL 方言，需要设置 table.sql-dialect 为 hive 并且加载 HiveModule。 后者非常重要，因为必须要加载 Hive 的内置函数后才能正确实现对 Hive 语法和语义 的兼容性。

例子如下：

```sql
CREATE CATALOG myhive WITH ('type' = 'hive'); -- setup HiveCatalog
USE CATALOG myhive;
LOAD MODULE hive; -- setup HiveModule
USE MODULES hive,core;
SET table.sql-dialect = hive; -- enable Hive dialect
SELECT key, value FROM src CLUSTER BY key; -- run some Hive queries
```

