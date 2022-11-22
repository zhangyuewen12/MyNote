# 腾讯广告业务基于Apache Flink + Hudi的批流一体实践

https://mp.weixin.qq.com/s/xuaW7fYy3QW3eNzoHQejqg



# **1.业务背景介绍**

广告主和代理商通过广告投放平台来进行广告投放，由多个媒介进行广告展示 ，从而触达到潜在用户。整个过程中会产生各种各样的数据，比如展现数据、点击数据。其中非常重要的数据是计费数据，以计费日志为依据向上可统计如行业维度、客户维度的消耗数据，分析不同维度的计费数据有助于业务及时进行商业决策，但目前部门内消耗统计以离线为主，这种T+1延迟的结果已经无法满足商业分析同学的日常分析需求，所以我们的目标为：建设口径统一的实时消耗数据，结合BI工具的自动化配置和展现能力，满足业务实时多维消耗分析，提高数据运营的效率和数据准确性。

# **2.架构选型**

## **2.1 Lambda架构**

由于部门内在过去一段时间主要以离线数据分析为主，所以经过持续对数据仓库治理，现有离线数据平台能力为：

- • PB级数据计算能力：基于主流大数据技术栈构建数据基础设施，数据规模**10PB+，日增数据量20T+，服务节点300+，日均提供2.5W+** 批次计算任务。
- • 数据开发实现一站式: 自研数据研发平台集成数据同步/数据计算/数据存储/任务调度/数据发布等数据研发全链路，一站式完成数据开发。
- • 数据治理初步实施：完成对数据的定义、开发、部署和调度进行全链路管控，强制保障数据仓库规范和元数据完整性，数据规范和数据治理初步实施。

基于此，初步规范的方案为：**不改变原有离线数据架构，在现有离线数据架构链路上再增加**[**实时计算**](https://cloud.tencent.com/product/oceanus?from=10680)**链路，该方案所形成的架构为Lambda架构，是目前业界比较主流的整体数仓架构方案**。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220916153418207.png" alt="image-20220916153418207" style="zoom:50%;" />

Lambda架构分为三层：离线处理层，实时处理层，对外服务层，对应图中的左下、左上和中间部分：

- • **离线处理层：** 主要存储数据集，在数据集上进行离线批计算，构建查询所对应的数据。离线处理层可以很好的处理离线数据，并将数据输出至服务层中。当前离线消耗计算的过程为：当天所产生的实时计费数据会输出至HDFS文件中，在第二天作为离线处理的ODS数据源，参与后续数据清洗和维度数据ETL计算，并同步最细维度数据至[数据服务](https://cloud.tencent.com/solution/data-collect-and-label-service?from=10680)层；
- • **实时处理层：** 实时处理层处理的是当天最近的增量数据流，具有较好的时效性。对应到实时消耗统计项目中，其过程为：kafka生产的实时计费日志作为数据源，由Flink计算引擎进行数据清洗，并将中间结果回写到kafka,最终将计算结果同步至数据服务层中；
- • **对外服务层：** 见图中数据服务层，其用于合并离线处理层和实时处理层中的结果数据集到最终数据集，并提供对BI等对外服务接口。如基于对外服务层的数据，业务分析同学可以通过BI工具自助配置，快速分析多维数据，从而提高分析效率。

Lambda架构存在如计算稳定、数据易于订正等优点，但也存在很明显的缺点：

- • 架构维护成本很高：存储框架不统一，计算框架不统一；
- • 业务维护成本高：数据存在两份、schema不统一、 数据处理逻辑不统一(两份)
- • Kafka不支持数据更新，只支持append，对延迟的数据无法更新之前的结果
- • Kafka无法支持DWD等层表高效的OLAP查询

基于以上缺点，进一步探索其他可行性方案。



## **2.2 批流一体架构**

对Lambda架构缺陷进一步分析：

- • **存储框架不统一：** 离线和实时计算采用的存储不统一，基于kafka的实时存储，无法满足即席的Olap查询，且存储能力有限，不支持海量存储。通过调研，可引入数据湖技术，实现离线数据和实时数据在存储层面的统一，数据湖的存储统一体现在用户不用关心底层真实存储，数据湖在真实存储上抽象TableFormat层，可将结构化或非结构化数据映射成数据表，实现存储上的统一；
- • **计算框架不统一：** 离线计算框架所采用的spark/hive计算和实时计算框架flink导致计算框架多样，维护成本高，且需要开发不同计算框架对应的ETL，导致研发成本高；可统一采用flink计算框架，其支持流式和批处理API，解决高成本问题；

基于Flink+数据湖实现计算统一以及存储统一的架构设计方案如下：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220916153733023.png" alt="image-20220916153733023" style="zoom:50%;" />

该方案存在如下优点：

- • 数据计算层：计算框架、存储框架统一，数据可维护性高；
- • 数据服务层：在数据建设上，构建最细维度实时消耗宽表，结合账户属性维度表，便于用户多维数据分析；在数据存储上，引入clickhouse,满足用户查询性能；
- • 数据分析展示层：结合BI工具的配置化能力，全面开放给业务同学，降低研发成本，提高业务分析效率；
- • 数据实时性：基于flink实时计算框架，能保证数据快速计算与输出；
- • 数据规范性：引入数据分层思想，对实时数据分层建设，遵循数据命名规范；



# **3.数据湖技术选型**

**数据湖概念**

- •把一家企业产生的数据都维护在一个平台内，这个平台就称之为“数据湖”，数据湖是一种支持存储多种原始数据格式、多种计算引擎、高效的元数据统一管理和海量统一数据存储。

**数据湖特点**

- • 存储原始数据，这些原始数据来源非常丰富(结构化，非结构化)；
- • 支持多种计算模型；
- • 完善的数据管理能力，要能做到多种数据源接入，实现不同数据之间的连接，支持 schema 管理等；
- • 灵活的底层存储，一般用 hdfs 这种廉价的[分布式文件系统](https://cloud.tencent.com/product/chdfs?from=10680)。

**数据湖架构**

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220916153853961.png" alt="image-20220916153853961" style="zoom:50%;" />

**Apache Hudi vs Iceberg**

由于Delta Lake更多的功能在其商业版本(如SQL模式下ALTER 变更等操作)，所以这里重点比对Hudi(Hadoop Upserts Delete Incrementals)和Iceberg：

| 场景             | 详情                               | Iceberg                               | Hudi                                         |
| :--------------- | :--------------------------------- | :------------------------------------ | :------------------------------------------- |
| (日志流水)Append | 离线数仓                           | 支持                                  | 支持                                         |
| (实时数仓)Upsert | - 近实时数仓 - 大任务批量化/增量化 | 只支持Binlog CDC 增量读时合并会丢数据 | 支持Binlog CDC 支持增量流读                  |
| 分钟入库         | Flink job                          | 支持                                  | 支持                                         |
| 分钟级查询       | Presto OLAP查询                    | 支持                                  | 支持                                         |
| 端到端分钟级     | 从数据采集到dal层（数据访问层）    | 支持，近10分钟                        | 支持，近分钟级                               |
| compaction影响   | 小文件合并                         | 数平提供了DLA服务需要额外的资源       | 默认inline compaction 也支持单独部署         |
| clean            | 数据清理                           | 数平提供了DLA服务需要额外的资源       | 默认inline clean(三种清理策略)也支持单独部署 |
| copy on write    | 分析场景                           | 支持                                  | 支持                                         |
| merge on read    | 低延迟                             | 支持                                  | 支持                                         |
| 离线数仓兼容     | 现有离线数仓平滑迁移               | 需要改造thive做适配                   | 需要改造thive做适配                          |
| Presto查询(OLAP) | 分析                               | 无索引                                | 有索引，性能更优                             |
| Spark分析计算    | 离线数仓                           | 无索引                                | 有索引，性能更优                             |

分析当前业务需求希望实时技术具备的能力

1. 高效的upsert；
2. 流式增量读写;
3. 高性能Olap查询；
4. ETL过程中数据回撤；

综合以上对比，结合当前业务所希望具备的数据能力，Hudi支持upsert、streaming read(增量流读)等功能和特性更适合实现批流一体的能力。



# **4.Hudi原理&实践要点**

## **4.1 文件组织结构**

### **4.1.1 整体结构**

Hudi将一个表映射为如下文件结构

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220916154042024.png" alt="image-20220916154042024" style="zoom:50%;" />

Hudi存储分为两个部分：

- •**元数据：** .hoodie目录对应着表的元数据信息，包括表的版本管理(Timeline)、归档目录(存放过时的instant也就是版本)，一个instant记录了一次提交（commit）的行为、时间戳和状态，Hudi以时间轴的形式维护了在数据集上执行的所有操作的元数据；
- • **数据：** 和hive一样，以分区方式存放数据；分区里面存放着Base File(*.parquet)和Log File(*.log.*)；



### **4.1.2 元数据区**

#### **4.1.2.1 Timeline**

Hudi维护着一条对Hudi数据集所有操作的不同 Instant组成的 Timeline（时间轴），通过时间轴，用户可以轻易的进行增量查询或基于某个历史时间点的查询。如下为实践过程中产生的Timeline：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220916154157992.png" alt="image-20220916154157992" style="zoom:50%;" />

一个Instant的组成如下

- • **state：**状态，目前包括REQUESTED（已调度但未初始化）、INFLIGHT（当前正在执行）、COMPLETED（操作执行完成），状态会转变，如当提交完成时会从 inflight状态转变为 completed状态。
- • **action：**操作，对数据集执行的操作类型，如 commit、 deltacommit等。
  - **提交（commit）:** 一次提交表示将一批记录原子写入数据集中的过程。
  - **增量提交(delta_commit) ：** 增量提交是指将一批记录原子写入到MOR表中，其中数据都将只写入到日志中。
  - **清理（clean）:** 清理数据集中不再被查询中使用的文件的较旧版本。
  - **压缩（compaction）:** 将MOR表中多个log文件进行合并，用以减小数据存储，本质是将行式文件转化为列式文件的动作。
- • **timestamp：**start 一个Instance发生的时间戳，Hudi会保证单调递增。

#### **4.1.2.2 Commit元数据文件分析**

如下为具体一个basePath/.hoodie/xxx.deltacommit 文件内容：

```
{
  "partitionToWriteStats" : {
    "20220902" : [ {
      "fileId" : "3628ebc8-78df-4e89-a3b3-6bc9a01366fe",
      "path" : "20220902/.3628ebc8-78df-4e89-a3b3-6bc9a01366fe_20220902104559106.log.1_0-1-0",
      "prevCommit" : "20220902104559106",
      "numWrites" : 1,
      "numDeletes" : 0,
      "numUpdateWrites" : 0,
      "numInserts" : 1,
      "totalWriteBytes" : 1081,
      "totalWriteErrors" : 0,
      "tempPath" : null,
      "partitionPath" : "20220902",
      "totalLogRecords" : 0,
      "totalLogFilesCompacted" : 0,
      "totalLogSizeCompacted" : 0,
      "totalUpdatedRecordsCompacted" : 0,
      "totalLogBlocks" : 0,
      "totalCorruptLogBlock" : 0,
      "totalRollbackBlocks" : 0,
      "fileSizeInBytes" : 1081,
      "minEventTime" : null,
      "maxEventTime" : null,
      "logVersion" : 1,
      "logOffset" : 0,
      "baseFile" : "",
      "logFiles" : [ ".3628ebc8-78df-4e89-a3b3-6bc9a01366fe_20220902104559106.log.1_0-1-0" ],
      "recordsStats" : {
        "val" : null,
        "present" : false
      },
      "columnStats" : {
        "val" : null,
        "present" : false
      }
    } ]
  },
  "compacted" : false,
  "extraMetadata" : {
    "schema" : "{\"type\":\"record\",\"name\":\"record\",\"fields\":[{\"name\":\"id\",\"type\":\"long\"},{\"name\":\"name\",\"type\":[\"null\",\"string\"],\"default\":null},{\"name\":\"birthday\",\"type\":[\"null\",{\"type\":\"long\",\"logicalType\":\"timestamp-millis\"}],\"default\":null},{\"name\":\"ts\",\"type\":[\"null\",{\"type\":\"long\",\"logicalType\":\"timestamp-millis\"}],\"default\":null},{\"name\":\"partition\",\"type\":[\"null\",\"string\"],\"default\":null}]}"
  },
  "operationType" : null,
  "totalRecordsDeleted" : 0,
  "totalLogFilesSize" : 0,
  "totalScanTime" : 0,
  "totalCreateTime" : 0,
  "totalUpsertTime" : 1070,
  "minAndMaxEventTime" : {
    "Optional.empty" : {
      "val" : null,
      "present" : false
    }
  },
  "writePartitionPaths" : [ "20220902" ],
  "writeStats" : [ {
    "fileId" : "3628ebc8-78df-4e89-a3b3-6bc9a01366fe",
    "path" : "20220902/.3628ebc8-78df-4e89-a3b3-6bc9a01366fe_20220902104559106.log.1_0-1-0",
    "prevCommit" : "20220902104559106",
    "numWrites" : 1,
    "numDeletes" : 0,
    "numUpdateWrites" : 0,
    "numInserts" : 1,
    "totalWriteBytes" : 1081,
    "totalWriteErrors" : 0,
    "tempPath" : null,
    "partitionPath" : "20220902",
    "totalLogRecords" : 0,
    "totalLogFilesCompacted" : 0,
    "totalLogSizeCompacted" : 0,
    "totalUpdatedRecordsCompacted" : 0,
    "totalLogBlocks" : 0,
    "totalCorruptLogBlock" : 0,
    "totalRollbackBlocks" : 0,
    "fileSizeInBytes" : 1081,
    "minEventTime" : null,
    "maxEventTime" : null,
    "logVersion" : 1,
    "logOffset" : 0,
    "baseFile" : "",
    "logFiles" : [ ".3628ebc8-78df-4e89-a3b3-6bc9a01366fe_20220902104559106.log.1_0-1-0" ],
    "recordsStats" : {
      "val" : null,
      "present" : false
    },
    "columnStats" : {
      "val" : null,
      "present" : false
    }
  } ],
  "fileIdAndRelativePaths" : {
    "3628ebc8-78df-4e89-a3b3-6bc9a01366fe" : "20220902/.3628ebc8-78df-4e89-a3b3-6bc9a01366fe_20220902104559106.log.1_0-1-0"
  },
  "totalLogRecordsCompacted" : 0,
  "totalLogFilesCompacted" : 0,
  "totalCompactedRecordsUpdated" : 0
}
```

文件包含如下重要元素

- • **fileId：** 在每个分区内，文件被组织为File Group，由文件Id唯一标识。每个File Group包含多个File Slice，其中每个Slice包含在某个Commit或Compcation Instant时间生成的Base File(*.parquet）以及Log Files（*.log*），该文件包含自生成基本文件以来对基本文件的插入和更新；
- • **path：** 对应着本次写入的文件路径，因为是MOR的表，所以写入的是日志文件；
- • **prevCommit：** 上一次的成功Commit；
- • **baseFile：** 基本文件，经过上一次Compaction后的文件，对于MOR表来说，每次读取的时候，通过将baseFile和logFiles合并，就会读取到实时的数据；
- • **logFiles：** 日志文件，MOR表写数据时，数据首先写进日志文件，之后会通过一次Compaction进行合并；
- • **compacted：** 本次操作是否为合并；
- • **extraMetadata：** 元数据信息，如表Schema信息；



### **4.1.3 数据区**

#### **4.1.3.1 基本概念**

**数据文件/基础文件**

Hudi将数据以列存格式（Parquet）存放，称为数据文件/基础文件。

**增量日志文件**

在 MOR 表格式中，更新被写入到增量日志文件中，该文件以 avro 格式存储。这些增量日志文件始终与基本文件相关联。假设有一个名为 data_file_1 的数据文件，对 data_file_1 中记录的任何更新都将写入到新的增量日志文件。在服务读取查询时，Hudi 将实时合并基础文件及其相应的增量日志文件中的记录。

**文件组(FileGroup)**

通常根据存储的数据量，可能会有很多数据文件。每个数据文件及其对应的增量日志文件形成一个文件组。在 COW表中，只有基本文件。

**文件版本**

比如COW表每当数据文件发生更新时，将创建数据文件的较新版本，其中包含来自较旧数据文件和较新传入记录的合并记录。

**文件切片(FileSlice)**

对于每个文件组，可能有不同的文件版本。因此文件切片由特定版本的数据文件及其增量日志文件组成。对于 COW表，最新的文件切片是指所有文件组的最新数据/基础文件。对于 MOR表，最新文件切片是指所有文件组的最新数据/基础文件及其关联的增量日志文件。

#### **4.1.3.2 数据组织**

在每个分区内，文件被组织为文件组，由文件ID充当唯一标识。每个文件组包含多个文件切片，其中每个切片包含在某个即时时间的提交/压缩生成的基本列文件(.parquet)以及一组日志文件(.log),该文件包含自生成基本文件以来对基本文件的插入/更新；数据构成关系：table -> partition -> FileGroup -> FileSlice -> parquet + log



## **4.2 表类型**

Hudi支持两种表类型：Copy On Write(COW) & Merge On Read(MOR)。

**COW表：**在数据写入的时候，通过复制旧文件数据并且与新写入的数据进行合并，对 Hudi 的每一个新批次写入都将创建相应数据文件的新版本。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220916160354153.png" alt="image-20220916160354153" style="zoom:50%;" />

data_file1 和 data_file2 都将创建更新的版本，data file 1 V2 是数据文件 data file 1 V1 的内容与数据文件data file 1 中传入批次匹配记录的记录合并。由于在写入期间进行合并，COW 会产生一些写入延迟。但是COW 的优势在于它的简单性，不需要其他表服务（如压缩）



**MOR表：**对于具有要更新记录的现有数据文件，Hudi 创建增量日志文件记录更新数据。此在写入期间不会合并或创建较新的数据文件版本；在进行数据读取的时候，将本批次读取到的数据进行Merge。Hudi 使用压缩机制来将数据文件和日志文件合并在一起并创建更新版本的数据文件。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220916160647639.png" alt="image-20220916160647639" style="zoom:50%;" />

**COW vs MOR**

| 说明       | COW  | MOR  |
| :--------- | :--- | :--- |
| 更新代价   | 高   | 低   |
| 读取延迟   | 低   | 一般 |
| 写放大问题 | 高   | 低   |



## **4.3 表写入原理**

重点分析Hudi与Flink集成时流式数据写入过程：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221108110312925.png" alt="image-20221108110312925" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221108110329050.png" alt="image-20221108110329050" style="zoom:50%;" />

分为三个模块：数据写入、数据压缩与数据清理。

**1.数据写入分析**

- • 基础数据封装：将数据流中flink的RowData封装成Hoodie实体；
- • BucketAssigner:桶分配器,主要是给数据分配写入的文件地址：若为插入操作，则取大小最小的FileGroup对应的FileId文件内进行插入；在此文件的后续写入中文件 ID 保持不变，并且提交时间会更新以显示最新版本。这也意味着记录的任何特定版本，给定其分区路径，都可以使用文件 ID 和 instantTime进行唯一定位；若为更新操作，则直接在当前location进行数据更新；
- • Hoodie Stream Writer: 数据写入,将数据缓存起来，在超过设置的最大flushSize或是做checkpoint时进行刷新到文件中；
- • Oprator Coordinator:主要与Hoodie Stream Writer进行交互，处理checkpoint等事件，在做checkpoint时，提交instant到timeLine上，并生成下一个instant的时间，算法为取当前最新的commi时间，比对当前时间与commit时间，若当前时间大于commit时间，则返回，否则一直循环等待生成。

**2.数据压缩**

压缩( compaction)用于在 MergeOnRead存储类型时将基于行的log日志文件转化为parquet列式数据文件，用于加快记录的查找。compaction首先会遍历各分区下最新的parquet数据文件和其对应的log日志文件进行合并，并生成新的FileSlice,在TimeLine 上提交新的Instance
<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220916161324724.png" alt="image-20220916161324724" style="zoom:50%;" />

具体策略分为4种，具体见官网说明：

```
compaction.trigger.strategy:
Strategy to trigger compaction, options are 
1.'num_commits': trigger compaction when reach N delta commits; 
2.'time_elapsed': trigger compaction when time elapsed > N seconds since last compaction; 
3.'num_and_time': trigger compaction when both NUM_COMMITS and TIME_ELAPSED are satisfied; 
4.'num_or_time': trigger compaction when NUM_COMMITS or TIME_ELAPSED is satisfied. Default is 'num_commits'
Default Value: num_commits (Optional)
```

在项目实践中需要注意参数'**read.streaming.skip_compaction' 参数的配置**，其表示在流式读取该表是否跳过压缩后的数据，若该表用于后续聚合操作表的输入表，则需要配置值为true,表示聚合操作表不再消费读取压缩数据。若不配置或配置为false,则该表中的数据在未被压缩之前被聚合操作表读取了一次，在压缩后数据又被读取一次，会导致聚合表的sum、count等算子结果出现双倍情况。

**3.数据清理**

随着用户向表中写入更多数据，对于每次更新，Hudi会生成一个新版本的数据文件用于保存更新后的记录(COPY_ON_WRITE) 或将这些增量更新写入日志文件以避免重写更新版本的数据文件 (MERGE_ON_READ)。在这种情况下，根据更新频率，文件版本数可能会无限增长，但如果不需要保留无限的历史记录，则必须有一个流程（服务）来回收旧版本的数据，这就是 Hudi 的清理服务。具体清理策略可参考官网，一般使用的清理策略为：KEEP_LATEST_FILE_VERSIONS：此策略具有保持 N 个文件版本而不受时间限制的效果。会删除N之外的FileSlice。

**4.Job图**

如下为生产环境中flink Job图，可以看到各task和上述分析过程对应，需要注意的是可以调整并行度来提升写入速度。

![image-20220916161716264](/Users/zyw/Library/Application Support/typora-user-images/image-20220916161716264.png)

## **4.4 表读取原理**

### **4.4.1 数据读取种类**

Hudi支持如下三种查询类型：

**Snapshot Queries：**可以查询最新COMMIT的快照数据。针对Merge On Read类型的表，查询时需要在线合并列存中的Base数据和日志中的实时数据；针对Copy On Write表，可以查询最新版本的Parquet数据。Copy On Write和Merge On Read表支持该类型的查询。

**Incremental Queries：**支持增量查询的能力，可以查询给定COMMIT之后的最新数据。Copy On Write和Merge On Read表支持该类型的查询。

**Read Optimized Queries：**只能查询到给定COMMIT之前所限定范围的最新数据。Read Optimized Queries是对Merge On Read表类型快照查询的优化，仅限于 MergeOnRead 表，可以查询到列存文件的数据,其原理是通过牺牲查询数据的时效性，来减少在线合并日志数据产生的查询延迟。

### **4.4.2 读取过程分析**

如下为Hudi数据流式读取Job图

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220916162438707.png" alt="image-20220916162438707" style="zoom:50%;" />

其过程为：

- • 开启split_monitor算子，每隔N秒(可配置)监听TimeLine上变化，并将变更的Instance封装为FileSlice 采用Rebanlance下发给split_reader task;
- • split_reader task根据FileSlice信息进行数据读取；

### **4.4.3 实践过程**

简化的数据流图如下，若大家和该数据流类似，那么在开发过程中会遇到并发导致的数据一致性问题、读任务在无数据时操作类型封装不正确问题 

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220916162549089.png" alt="image-20220916162549089" style="zoom:50%;" />

#### **4.4.3.1 并发导致的数据一致性问题**

**问题&原因分析**

问题出现在第4步：flink会启动split_monitor任务，每隔N秒(可配置)监听TimeLine上变化；同时会开启split_reader task任务，split_reader task会定位到具体文件中进行数据读取，而sink算子同样会集成在同一个split_reader task任务中(flink oprator chain原理，可节省数据传输带来的序列化反序列化和网络传输开销)。split_monitor对split_reader task采取的是Rebanlance分发策略，若同一个key在并发下，提交到不同Instance中，则split_monitor可能将包含同一个key的两次Instance分发到不同split_reader task任务中，当读取到数据向外部存储sink时，由于网络速度等因素，先处理的split_reader task任务对应的结果可能会后sink,导致外部存储结果的错误，即之前更新结果覆盖了最新的更新结果。具体分析流程如下





#### **4.3.3.3 参数设置**

由于Hudi ods表作为dwd表的输入，dwd表作为dws表的输入，dws表作为sink到外部存储的输入，所以在创建表时，需要指定流式读取，增量消费数据：

```javascript
'read.streaming.enabled' = 'true',  
'read.streaming.check-interval' = '2' ,
'read.start-commit' = '20210316134557' ,
'changelog.enabled' = 'true',
'read.streaming.skip_compaction' = 'true'
```



# **5. 收益**

- • 数据延迟从T+1向近实时演进(1min数据延迟)；
- • 高维护性：计算框架、存储框架统一，数据可维护性高；
- • 低业务维护成本：数据一份、schema统一、 数据处理逻辑不用维护两套；
- • 纯SQL化极大的加速了用户的开发效率；
- • 基于Hudi存储的高效OLAP查询支持；