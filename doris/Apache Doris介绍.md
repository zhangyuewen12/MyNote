# 一、Doris简介

## 1.1 Doris概述

Apache Doris 由百度大数据部研发(之前叫百度 Palo，2018 年贡献到 Apache 社区后， 更名为 Doris )，在百度内部，有超过 200 个产品线在使用，部署机器超过 1000 台，单一 业务最大可达到上百 TB。

Apache Doris 是一个现代化的 MPP(Massively Parallel Processing，即大规模并行处理) 分析型数据库产品。仅需亚秒级响应时间即可获得查询结果，有效地支持实时数据分析。 Apache Doris 的分布式架构非常简洁，易于运维，并且可以支持 10PB 以上的超大数据集。

Apache Doris 可以满足多种数据分析需求，例如固定历史报表，实时数据分析，交互式 数据分析和探索式数据分析等。

## 1.2 Doris架构

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220916101037746.png" alt="image-20220916101037746" style="zoom:50%;" />



Doris 的架构很简洁，只设 FE(Frontend)、BE(Backend)两种角色、两个进程，不依赖于 外部组件，方便部署和运维，FE、BE 都可线性扩展。
 ⚫ **FE**(**Frontend**):存储、维护集群元数据;负责接收、解析查询请求，规划查询计划， 调度查询执行，返回查询结果。

主要有三个角色:

(1)Leader 和 Follower:主要是用来达到元数据的高可用，保证单节点宕机的情况下， 元数据能够实时地在线恢复，而不影响整个服务。

(2)Observer:用来扩展查询节点，同时起到元数据备份的作用。如果在发现集群压力 非常大的情况下，需要去扩展整个查询的能力，那么可以加 observer 的节点。observer 不 参与任何的写入，只参与读取。

⚫ **BE**(**Backend**):负责物理数据的存储和计算;依据 FE 生成的物理计划，分布式地执行查询。

数据的可靠性由 BE 保证，BE 会对整个数据存储多副本或者是三副本。副本数可根据 需求动态调整。

⚫ **MySQLClient
** Doris 借助 MySQL 协议，用户使用任意 MySQL 的 ODBC/JDBC 以及 MySQL 的客户

端，都可以直接访问 Doris。 

⚫ **Broker**

Broker 为一个独立的无状态进程。封装了文件系统接口，提供 Doris 读取远端存储系统 中文件的能力，包括 HDFS，S3，BOS 等。

# 二、编译和安装



# 三、数据表的创建

## 3.1 创建用户和数据库

1)创建 test 用户

> mysql -h hadoop1 -P 9030 -uroot -p 
>
> create user 'test' identified by 'test';

2)创建数据库

> create database test_db; 

3)用户授权 

> grant all on test_db to test;



## 3.2基本概念

在 Doris 中，数据都以关系表(Table)的形式进行逻辑上的描述。

### **3.2.1 Row & Column**

一张表包括行(Row)和列(Column)。Row 即用户的一行数据。Column 用于描述一 行数据中不同的字段。

-   在默认的数据模型中，Column 只分为排序列和非排序列。存储引擎会按照排序列 对数据进行排序存储，并建立稀疏索引，以便在排序数据上进行快速查找。

- 而在聚合模型中，Column 可以分为两大类:Key 和 Value。从业务角度看，Key 和 Value 可以分别对应维度列和指标列。从聚合模型的角度来说，Key 列相同的行， 会聚合成一行。其中 Value 列的聚合方式由用户在建表时指定。

### **3.2.2 Partition & Tablet**

在 Doris 的存储引擎中，用户数据首先被划分成若干个分区(Partition)，划分的规则通 常是按照用户指定的分区列进行范围划分，比如按时间划分。而在每个分区内，数据被进一 步的按照 Hash 的方式分桶，分桶的规则是要找用户指定的分桶列的值进行 Hash 后分桶。 每个分桶就是一个数据分片(Tablet)，也是数据划分的最小逻辑单元。

⚫  Tablet之间的数据是没有交集的，独立存储的。Tablet也是数据移动、复制等操作 的最小物理存储单元。

⚫  Partition可以视为是逻辑上最小的管理单元。数据的导入与删除，都可以或仅能针 对一个 Partition 进行。

### 3.3建表实例

建表语法:

```
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] [database.]table_name (column_definition1[, column_definition2, ...]
[, index_definition1[, index_definition12,]])
[ENGINE = [olap|mysql|broker|hive]] [key_desc]
[COMMENT "table comment"]; [partition_desc] [distribution_desc]
[rollup_index]
[PROPERTIES ("key"="value", ...)] [BROKER PROPERTIES ("key"="value", ...)];
```

Doris 的建表是一个同步命令，命令返回成功，即表示建表成功。

Doris 支持支持单分区和复合分区两种建表方式。



复合分区:既有分区也有分桶

第一级称为 Partition，即分区。用户可以指定某一维度列作为分区列(当前只支持整型 和时间类型的列)，并指定每个分区的取值范围。

第二级称为 Distribution，即分桶。用户可以指定一个或多个维度列以及桶数对数据进 行 HASH 分布.

2)单分区:只做 HASH 

分布，即只分桶

### **3.3.2** 字段类型

### **3.3.2** 建表示例

我们以一个建表操作来说明 Doris 的数据划分。

#### **3.3.2.1 Range Partition**

```sql
CREATE TABLE IF NOT EXISTS example_db.expamle_range_tbl (
`user_id` LARGEINT NOT NULL COMMENT "用户id",
`date` DATE NOT NULL COMMENT "数据灌入日期时间",
`timestamp` DATETIME NOT NULL COMMENT "数据灌入的时间戳",
`city` VARCHAR(20) COMMENT "用户所在城市",
`age` SMALLINT COMMENT "用户年龄",
`sex` TINYINT COMMENT "用户性别",
`last_visit_date` DATETIME REPLACE DEFAULT "1970-01-01
00:00:00" COMMENT "用户最后一次访问时间",
`cost` BIGINT SUM DEFAULT "0" COMMENT "用户总消费", `max_dwell_time` INT MAX DEFAULT "0" COMMENT "用户最大停留时间", `min_dwell_time` INT MIN DEFAULT "99999" COMMENT "用户最小停留时间"
)
ENGINE=olap
AGGREGATE KEY(`user_id`, `date`, `timestamp`, `city`, `age`, `sex`) PARTITION BY RANGE(`date`)
(
PARTITION `p201701` VALUES LESS THAN ("2017-02-01"),
PARTITION `p201702` VALUES LESS THAN ("2017-03-01"),
PARTITION `p201703` VALUES LESS THAN ("2017-04-01")
)
DISTRIBUTED BY HASH(`user_id`) BUCKETS 16 PROPERTIES
(
"replication_num" = "3",
"storage_medium" = "SSD", "storage_cooldown_time" = "2018-01-01 12:00:00"
);
```

#### **3.3.2.2 List Partition**

```sql
CREATE TABLE IF NOT EXISTS example_db.expamle_list_tbl (
`user_id` LARGEINT NOT NULL COMMENT "用户id",
`date` DATE NOT NULL COMMENT "数据灌入日期时间",
`timestamp` DATETIME NOT NULL COMMENT "数据灌入的时间戳",
`city` VARCHAR(20) COMMENT "用户所在城市",
`age` SMALLINT COMMENT "用户年龄",
`sex` TINYINT COMMENT "用户性别",
`last_visit_date` DATETIME REPLACE DEFAULT "1970-01-01
00:00:00" COMMENT "用户最后一次访问时间",
`cost` BIGINT SUM DEFAULT "0" COMMENT "用户总消费", `max_dwell_time` INT MAX DEFAULT "0" COMMENT "用户最大停留时间", `min_dwell_time` INT MIN DEFAULT "99999" COMMENT "用户最小停留时
间"
)
ENGINE=olap
AGGREGATE KEY(`user_id`, `date`, `timestamp`, `city`, `age`, `sex`) PARTITION BY LIST(`city`)
(
PARTITION `p_cn` VALUES IN ("Beijing", "Shanghai", "Hong Kong"),
PARTITION `p_usa` VALUES IN ("New York", "San Francisco"),
PARTITION `p_jp` VALUES IN ("Tokyo") )
DISTRIBUTED BY HASH(`user_id`) BUCKETS 16 PROPERTIES
(
"replication_num" = "3",
"storage_medium" = "SSD", "storage_cooldown_time" = "2018-01-01 12:00:00"
);
```

## **3.4** 数据划分

 以 3.3.2 的建表示例来理解。

### **3.4.1** 列定义

以 AGGREGATE KEY 数据模型为例进行说明。更多数据模型参阅 Doris 数据模型。 列的基本类型，可以通过在 mysql-client 中执行 HELP CREATE TABLE; 查看。 AGGREGATE KEY 数据模型中，所有没有指定聚合方式(SUM、REPLACE、MAX、MIN)的列视为 Key 列。而其余则为 Value 列。
 定义列时，可参照如下建议:
 ➢ Key 列必须在所有 Value 列之前。
 ➢ 尽量选择整型类型。因为整型类型的计算和查找比较效率远高于字符串。

 ➢ 对于不同长度的整型类型的选择原则，遵循够用即可。

➢ 对于 VARCHAR 和 STRING 类型的长度，遵循 够用即可。

 ➢ 所有列的总字节长度(包括 Key 和 Value)不能超过 100KB。

### **3.4.2** 分区与分桶

Doris 支持两层的数据划分。第一层是 Partition，支持 Range 和 List 的划分方式。第二 层是 Bucket(Tablet)，仅支持 Hash 的划分方式。

也可以仅使用一层分区。使用一层分区时，只支持 Bucket 划分。 

#### **3.4.2.1 Partition**

- ➢  Partition 列可以指定一列或多列。分区类必须为 KEY 列。多列分区的使用方式在 后面介绍。
- ➢  不论分区列是什么类型，在写分区值时，都需要加双引号。
- ➢  分区数量理论上没有上限。
- ➢  当不使用 Partition 建表时，系统会自动生成一个和表名同名的，全值范围的Partition。该 Partition 对用户不可见，并且不可删改。

**1**) **Range** 分区

分区列通常为时间列，以方便的管理新旧数据。不可添加范围重叠的分区。
 Partition 指定范围的方式
 ⚫ VALUES LESS THAN (...) 仅指定上界，系统会将前一个分区的上界作为该分区的下界，生成一个左闭右开的区间。分区的删除不会改变已存在分区的范围。删除分 区可能出现空洞。

⚫ VALUES [...) 指定同时指定上下界，生成一个左闭右开的区间。

通过 VALUES [...) 同时指定上下界比较容易理解。这里举例说明，当使用 VALUES LESS THAN (...) 语句进行分区的增删操作时，分区范围的变化情况:

(1)如上 expamle_range_tbl 示例，当建表完成后，会自动生成如下 3 个分区:

```
 
```



(2)增加一个分区 p201705 VALUES LESS THAN ("2017-06-01")，分区结果如下:

(3)此时删除分区 p201703，则分区结果如下:



**2**)**List** 分区

分区列支持 BOOLEAN, TINYINT, SMALLINT, INT, BIGINT, LARGEINT, DATE, DATETIME, CHAR, VARCHAR 数据类型，分区值为枚举值。只有当数据为目标分区枚举值其中之一时，才可以命中分区。不可添加范围重叠的分区。
 Partition 支持通过 VALUES IN (...) 来指定每个分区包含的枚举值。下面通过示例说明，

进行分区的增删操作时，分区的变化。



#### **3.4.2.2** **Bucket**

(1)如果使用了 Partition，则 DISTRIBUTED ... 语句描述的是数据在各个分区内的划 分规则。如果不使用 Partition，则描述的是对整个表的数据的划分规则。

(2)分桶列可以是多列，但必须为 **Key** 列。分桶列可以和 Partition 列相同或不同。

(3)分桶列的选择，是在 查询吞吐 和 查询并发 之间的一种权衡:

- 如果选择多个分桶列，则数据分布更均匀。

如果一个查询条件不包含所有分桶列的等值条件，那么该查询会触发所有分桶同时 扫描，这样查询的吞吐会增加，单个查询的延迟随之降低。这个方式适合大吞吐低并发 的查询场景。

- 如果仅选择一个或少数分桶列，则对应的点查询可以仅触发一个分桶扫描。

此时，当多个点查询并发时，这些查询有较大的概率分别触发不同的分桶扫描，各 个查询之间的 IO 影响较小(尤其当不同桶分布在不同磁盘上时)，所以这种方式适合 高并发的点查询场景。

(4)分桶的数量理论上没有上限。



#### **3.4.2.3** 使用复合分区的场景

 以下场景推荐使用复合分区

(1)有时间维度或类似带有有序值的维度，可以以这类维度列作为分区列。分区粒度可以根据导入频次、分区数据量等进行评估。

(2)历史数据删除需求:如有删除历史数据的需求(比如仅保留最近 N 天的数据)。 使用复合分区，可以通过删除历史分区来达到目的。也可以通过在指定分区内发送 DELETE 语句进行数据删除。

(3)解决数据倾斜问题:每个分区可以单独指定分桶数量。如按天分区，当每天的数 据量差异很大时，可以通过指定分区的分桶数，合理划分不同分区的数据,分桶列建议选择 区分度大的列。



#### **3.4.2.4** 多列分区

Doris 支持指定多列作为分区列，示例如下: 

**1**)**Range** 分区

**2**)**List** 分区



### **3.4.3 PROPERTIES**

在建表语句的最后 PROPERTIES 中，可以指定以下两个参数

#### **3.4.3.1 replication_num**

每个 Tablet 的副本数量。默认为 3，建议保持默认即可。在建表语句中，所有 Partition 中的 Tablet 副本数量统一指定。而在增加新分区时，可以单独指定新分区中 Tablet 的副本 数量。  副本数量可以在运行时修改。强烈建议保持奇数。

最大副本数量取决于集群中独立 **IP** 的数量(注意不是 **BE** 数量)。Doris 中副本分布的 原则是，不允许同一个 Tablet 的副本分布在同一台物理机上，而识别物理机即通过 IP。所 以，即使在同一台物理机上部署了 3 个或更多 BE 实例，如果这些 BE 的 IP 相同，则依然只 能设置副本数为 1。

对于一些小，并且更新不频繁的维度表，可以考虑设置更多的副本数。这样在 Join 查询 时，可以有更大的概率进行本地数据 Join。



#### **3.4.3.2 storage_medium & storage_cooldown_time**

BE 的数据存储目录可以显式的指定为 SSD 或者 HDD(通过 .SSD 或者 .HDD 后缀 区分)。建表时，可以统一指定所有 Partition 初始存储的介质。注意，后缀作用是显式指 定磁盘介质，而不会检查是否与实际介质类型相符。

默认初始存储介质可通过 fe 的配置文件 fe.conf 中指定 default_storage_medium=xxx， 如果没有指定，则默认为 HDD。如果指定为 SSD，则数据初始存放在 SSD 上。

如果没有指定 storage_cooldown_time，则默认 30 天后，数据会从 SSD 自动迁移到 HDD 上。如果指定了 storage_cooldown_time，则在到达 storage_cooldown_time 时间后，数据才会 迁移。

注意，当指定 storage_medium 时，如果 FE 参数 enable_strict_storage_medium_check 为 False 该参数只是一个“尽力而为”的设置。即使集群内没有设置 SSD 存储介质，也不会报 错，而是自动存储在可用的数据目录中。 同样，如果 SSD 介质不可访问、空间不足，都可 能导致数据初始直接存储在其他可用介质上。而数据到期迁移到 HDD 时，如果 HDD 介质 不可访问、空间不足，也可能迁移失败(但是会不断尝试)。 如果 FE 参数 enable_strict_storage_medium_check 为 True 则当集群内没有设置 SSD 存储介质时，会报错 Failed to find enough host in all backends with storage medium is SSD。



### **3.4.4 ENGINE**

本示例中，ENGINE 的类型是 olap，即默认的 ENGINE 类型。在 Doris 中，只有这个 ENGINE 类型是由 Doris 负责数据管理和存储的。其他 ENGINE 类型，如 mysql、broker、 es 等等，本质上只是对外部其他数据库或系统中的表的映射，以保证 Doris 可以读取这些数 据。而 Doris 本身并不创建、管理和存储任何非 olap ENGINE 类型的表和数据。

### **3.4.5** 其他

 `IF NOT EXISTS` 表示如果没有创建过该表，则创建。注意这里只判断表名是否存在，而不会判断新建表结构是否与已存在的表结构相同.

## **3.5** 数据模型

 Doris 的数据模型主要分为 3 类:Aggregate、Uniq、Duplicate

### **3.5.1 Aggregate** 模型

表中的列按照是否设置了 AggregationType，分为 Key(维度列)和 Value(指标列)。 没有设置 AggregationType 的称为 Key，设置了 AggregationType 的称为 Value。

当我们导入数据时，对于 Key 列相同的行会聚合成一行，而 Value 列会按照设置的 AggregationType 进行聚合。AggregationType 目前有以下四种聚合方式:

- ➢  SUM:求和，多行的Value进行累加。

- ➢  REPLACE:替代，下一批数据中的 Value 会替换之前导入过的行中的 Value。

  REPLACE_IF_NOT_NULL :当遇到 null 值则不更新。

- ➢  MAX:保留最大值。

- ➢  MIN:保留最小值。
   数据的聚合，在 Doris 中有如下三个阶段发生:

(1)每一批次数据导入的 ETL 阶段。该阶段会在每一批次导入的数据内部进行聚合。

(2)底层 BE 进行数据 Compaction 的阶段。该阶段，BE 会对已导入的不同批次的 数据进行进一步的聚合。

(3)数据查询阶段。在数据查询时，对于查询涉及到的数据，会进行对应的聚合。

> 数据在不同时间，可能聚合的程度不一致。比如一批数据刚导入时，可能还未与之前已 存在的数据进行聚合。但是对于用户而言，用户只能查询到聚合后的数据。即不同的聚合程 度对于用户查询而言是透明的。用户需始终认为数据以最终的完成的聚合程度存在，而不应 假设某些聚合还未发生。(可参阅聚合模型的局限性一节获得更多详情。)



#### **3.5.1.1** 示例一:导入数据聚合

1)建表

```sql
CREATE TABLE IF NOT EXISTS test_db.example_site_visit (
`user_id` LARGEINT NOT NULL COMMENT "用户id",
`date` DATE NOT NULL COMMENT "数据灌入日期时间", 
`city` VARCHAR(20) COMMENT "用户所在城市",
`age` SMALLINT COMMENT "用户年龄",
`sex` TINYINT COMMENT "用户性别",
`last_visit_date` DATETIME REPLACE DEFAULT "1970-01-01 00:00:00" COMMENT "用户最后一次访问时间",
`last_visit_date_not_null` DATETIME REPLACE_IF_NOT_NULL DEFAULT "1970-01-01 00:00:00" COMMENT "用户最后一次访问时间",
`cost` BIGINT SUM DEFAULT "0" COMMENT "用户总消费", `max_dwell_time` INT MAX DEFAULT "0" COMMENT "用户最大停留时间", `min_dwell_time` INT MIN DEFAULT "99999" COMMENT "用户最小停留时
间"
)
AGGREGATE KEY(`user_id`, `date`, `city`, `age`, `sex`) 
DISTRIBUTED BY HASH(`user_id`) BUCKETS 10;
```

2)插入数据

```sql
insert into test_db.example_site_visit values\ (10000,'2017-10-01','北京',20,0,'2017-10-01 06:00:00','2017-10-01 06:00:00',20,10,10),\
(10000,'2017-10-01','北京',20,0,'2017-10-01 07:00:00','2017-10-01 07:00:00',15,2,2),\
(10001,'2017-10-01','北京',30,1,'2017-10-01 17:05:45','2017-10-01 07:00:00',2,22,22),\
(10002,'2017-10-02',' 上 海 ',20,1,'2017-10-02 12:59:12',null,200,5,5),\ (10003,'2017-10-02','广州',32,0,'2017-10-02 11:20:00','2017-10-02 11:20:00',30,11,11),\
(10004,'2017-10-01','深圳',35,0,'2017-10-01 10:00:15','2017-10-01 10:00:15',100,3,3),\
(10004,'2017-10-03','深圳',35,0,'2017-10-03 10:20:22','2017-10-03	10:20:22',11,6,6);
```

> 注意:Insert into 单条数据这种操作在 Doris 里只能演示不能在生产使用，会引发写阻 塞。

3)查看表

 ```sql
 select * from test_db.example_site_visit;
 ```

可以看到，用户 10000 只剩下了一行聚合后的数据。而其余用户的数据和原始数据保 持一致。经过聚合，Doris 中最终只会存储聚合后的数据。换句话说，即明细数据会丢失， 用户不能够再查询到聚合前的明细数据了。

#### **3.5.1.2** 示例二:保留明细数据

1)建表

```sql
CREATE TABLE IF NOT EXISTS test_db.example_site_visit2 (
`user_id` LARGEINT NOT NULL COMMENT "用户id", 
`date` DATE NOT NULL COMMENT "数据灌入日期时间", 
`timestamp` DATETIME COMMENT "数据灌入时间，精确到秒", 
`city` VARCHAR(20) COMMENT "用户所在城市",
`age` SMALLINT COMMENT "用户年龄",
`sex` TINYINT COMMENT "用户性别",
`last_visit_date` DATETIME REPLACE DEFAULT "1970-01-0100:00:00" COMMENT "用户最后一次访问时间",
`cost` BIGINT SUM DEFAULT "0" COMMENT "用户总消费",
`max_dwell_time` INT MAX DEFAULT "0" COMMENT "用户最大停留时间",
`min_dwell_time` INT MIN DEFAULT "99999" COMMENT "用户最小停留时
间"
)
AGGREGATE KEY(`user_id`, `date`, `timestamp`, `city`, `age`, `sex`) 
DISTRIBUTED BY HASH(`user_id`) BUCKETS 10;
```

2)插入数据

```sql
insert into test_db.example_site_visit2 values(10000,'2017-10- 01','2017-10-01 08:00:05',' 北 京 ',20,0,'2017-10-01
06:00:00',20,10,10),\
(10000,'2017-10-01','2017-10-01 09:00:05','北京',20,0,'2017-10-01 07:00:00',15,2,2),\
(10001,'2017-10-01','2017-10-01 18:12:10','北京',30,1,'2017-10-01 17:05:45',2,22,22),\
(10002,'2017-10-02','2017-10-02 13:10:00','上海',20,1,'2017-10-02 12:59:12',200,5,5),\
(10003,'2017-10-02','2017-10-02 13:15:00','广州',32,0,'2017-10-02 11:20:00',30,11,11),\
(10004,'2017-10-01','2017-10-01 12:12:48','深圳',35,0,'2017-10-01 10:00:15',100,3,3),\
(10004,'2017-10-03','2017-10-03 12:38:20','深圳',35,0,'2017-10-03 10:20:22',11,6,6);
```

3)查看表

```sql
 select * from test_db.example_site_visit2;
```

存储的数据，和导入数据完全一样，没有发生任何聚合。这是因为，这批数据中，因为 加入了 timestamp 列，所有行的 Key 都不完全相同。也就是说，只要保证导入的数据中， 每一行的 Key 都不完全相同，那么即使在聚合模型下，Doris 也可以保存完整的明细数据。



#### *3.5.1.3** 示例三:导入数据与已有数据聚合

1)往实例一中继续插入数据

```sql
insert into test_db.example_site_visit values(10004,'2017-10-03',' 深圳',35,0,'2017-10-03 11:22:00',null,44,19,19),\ (10005,'2017-10-03','长沙',29,1,'2017-10-03 18:11:02','2017-10-03
18:11:02',3,1,1);
```

2)查看表

```sql
select * from test_db.example_site_visit;
```

可以看到，用户 10004 的已有数据和新导入的数据发生了聚合。同时新增了 10005 用 户的数据.



### **3.5.2 Uniq** 模型

在某些多维分析场景下，用户更关注的是如何保证 Key 的唯一性，即如何获得 Primary Key 唯一性约束。因此，我们引入了 Uniq 的数据模型。该模型本质上是聚合模型的一个特 例，也是一种简化的表结构表示方式。

1)建表

```sql
CREATE TABLE IF NOT EXISTS test_db.user (
`user_id` LARGEINT NOT NULL COMMENT "用户id",
`username` VARCHAR(50) NOT NULL COMMENT "用户昵称", 
`city` VARCHAR(20) COMMENT "用户所在城市",
`age` SMALLINT COMMENT "用户年龄",
`sex` TINYINT COMMENT "用户性别",
`phone` LARGEINT COMMENT "用户电话",
`address` VARCHAR(500) COMMENT "用户地址",
`register_time` DATETIME COMMENT "用户注册时间" 
)
UNIQUE KEY(`user_id`, `username`) 
DISTRIBUTED BY HASH(`user_id`) BUCKETS 10;
```

2)插入数据

```sql
insert into test_db.user values\
(10000,'wuyanzu',' 北 京 ',18,0,12345678910,' 北 京 朝 阳 区 ','2017-10-01 07:00:00'),\
(10000,'wuyanzu',' 北 京 ',19,0,12345678910,' 北 京 朝 阳 区 ','2017-10-01 07:00:00'),\
(10000,'zhangsan',' 北 京 ',20,0,12345678910,' 北 京 海 淀 区 ','2017-11-15 06:10:20');
```

3)查询表

```sql
select * from test_db.user;
```

Uniq 模型完全可以用聚合模型中的 REPLACE 方式替代。其内部的实现方式和数据存 储方式也完全一样。

### **3.5.3 Duplicate** 模型

在某些多维分析场景下，数据既没有主键，也没有聚合需求。Duplicate 数据模型可以 满足这类需求。数据完全按照导入文件中的数据进行存储，不会有任何聚合。即使两行数据 完全相同，也都会保留。 **而在建表语句中指定的 DUPLICATE KEY，只是用来指明底层数 据按照那些列进行排序。**

1)建表

```sql
CREATE TABLE IF NOT EXISTS test_db.example_log (
`timestamp` DATETIME NOT NULL COMMENT "日志时间",
`type` INT NOT NULL COMMENT "日志类型",
`error_code` INT COMMENT "错误码",
`error_msg` VARCHAR(1024) COMMENT "错误详细信息",
`op_id` BIGINT COMMENT "负责人 id",
`op_time` DATETIME COMMENT "处理时间"
)
DUPLICATE KEY(`timestamp`, `type`)
DISTRIBUTED BY HASH(`timestamp`) BUCKETS 10;
```

2)插入数据

```
insert into test_db.example_log values\
('2017-10-01 08:00:05',1,404,'not found page', 101, '2017-10-01
08:00:05'),\
('2017-10-01
08:00:05'),\
('2017-10-01
08:00:06'),\
('2017-10-01
08:00:07');
08:00:05',1,404,'not found page', 101, '2017-10-01 08:00:05',1,404,'not found page', 101, '2017-10-01 08:00:05',2,404,'not found page', 101, '2017-10-01 08:00:06',2,404,'not found page', 101, '2017-10-01
```



### **3.5.4** 数据模型的选择建议

因为数据模型在建表时就已经确定，且无法修改。所以，选择一个合适的数据模型非常 重要。

**(1)Aggregate 模型可以通过预聚合，极大地降低聚合查询时所需扫描的数据量和查询 的计算量，非常适合有固定模式的报表类查询场景。但是该模型对 count(*) 查询很不友好。 同时因为固定了 Value 列上的聚合方式，在进行其他类型的聚合查询时，需要考虑语意正确 性。**

**(2)Uniq 模型针对需要唯一主键约束的场景，可以保证主键唯一性约束。但是无法利 用 ROLLUP 等预聚合带来的查询优势(因为本质是REPLACE，没有SUM这种聚合方式)。**

**(3)Duplicate 适合任意维度的 Ad-hoc 查询。虽然同样无法利用预聚合的特性，但是不 受聚合模型的约束，可以发挥列存模型的优势(只读取相关列，而不需要读取所有 Key 列)**

### **3.5.5** 聚合模型的局限性

这里我们针对 Aggregate 模型(包括 Uniq 模型)，来介绍下聚合模型的局限性。

在聚合模型中，模型对外展现的，是最终聚合后的数据。也就是说，任何还未聚合的数 据(比如说两个不同导入批次的数据)，必须通过某种方式，以保证对外展示的一致性。我 们举例说明。

假设表结构如下:

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220919165222202.png" alt="image-20220919165222202" style="zoom:50%;" />

假设存储引擎中有如下两个已经导入完成的批次的数据:



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220919165243528.png" alt="image-20220919165243528" style="zoom:50%;" />

可以看到，用户 10001 分属在两个导入批次中的数据还没有聚合。但是为了保证用户 只能查询到如下最终聚合后的数据:



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220919165307220.png" alt="image-20220919165307220" style="zoom:50%;" />



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220919165330164.png" alt="image-20220919165330164" style="zoom:50%;" />



在查询引擎中加入了聚合算子，来保证数据对外的一致性。

**另外，在聚合列(Value)上，执行与聚合类型不一致的聚合类查询时，要注意语意。比 如我们在如上示例中执行如下查询**:

```sql
SELECT MIN(cost) FROM table;
```

得到的结果是 5，而不是 1。

同时，这种一致性保证，在某些查询中，会极大的降低查询效率。

我们以最基本的 count(*) 查询为例: *

```sql
SELECT COUNT(*) FROM table;
```

在其他数据库中，这类查询都会很快的返回结果。因为在实现上，我们可以通过如“导 入时对行进行计数，保存 count 的统计信息”，或者在查询时“仅扫描某一列数据，获得 count 值”的方式，只需很小的开销，即可获得查询结果。但是在 Doris 的聚合模型中，这种查询 的开销非常大。

上面的例子，select count(*) from table; 的正确结果应该为 4。但如果我们只扫描 user_id 这一列，如果加上查询时聚合，最终得到的结果是 3(10001, 10002, 10003)。而如果不加 查询时聚合，则得到的结果是 5(两批次一共 5 行数据)。可见这两个结果都是不对的。

为了得到正确的结果，我们必须同时读取 user_id 和 date 这两列的数据，再加上查询时 聚合，才能返回 4 这个正确的结果。也就是说，在 count(*) 查询中，Doris 必须扫描所有的 AGGREGATE KEY 列(这里就是 user_id 和 date)，并且聚合后，才能得到语意正确的结果。 当聚合列非常多时，count(*)查询需要扫描大量的数据。

因此，当业务上有频繁的 count(*) 查询时，我们建议用户通过增加一个值恒为 1 的， 聚合类型为 SUM 的列来模拟 count(*)。如刚才的例子中的表结构，我们修改如下:

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220919170158859.png" alt="image-20220919170158859" style="zoom:50%;" />

增加一个 count 列，并且导入数据中，该列值恒为 1。则 select count(*) from table 的结果等价于 select sum(count) from table;。而后者的查询效率将远高于前者。不过这种方式也有 使用限制，就是用户需要自行保证，不会重复导入 AGGREGATE KEY 列都相同的行。否则， select sum(count) from table; 只能表述原始导入的行数，而不是 select count(*) from table; 的 语义

另一种方式，就是 将如上的 count 列的聚合类型改为 REPLACE，且依然值恒为 1。那 么 select sum(count) from table; 和 select count(*) from table; 的结果将是一致的。并且这种 方式，没有导入重复行的限制。

## **3.6** 动态分区

动态分区是在 Doris 0.12 版本中引入的新功能。旨在对表级别的分区实现生命周期管理 (TTL)，减少用户的使用负担。

目前实现了动态添加分区及动态删除分区的功能。动态分区只支持 Range 分区。

###  **3.6.1** 原理

在某些使用场景下，用户会将表按照天进行分区划分，每天定时执行例行任务，这时需 要使用方手动管理分区，否则可能由于使用方没有创建分区导致数据导入失败，这给使用方 带来了额外的维护成本。

通过动态分区功能，用户可以在建表时设定动态分区的规则。FE 会启动一个后台线程， 根据用户指定的规则创建或删除分区。用户也可以在运行时对现有规则进行变更。

### **3.6.2** 使用方式

动态分区的规则可以在建表时指定，或者在运行时进行修改。当前仅支持对单分区列的 分区表设定动态分区规则。

建表时指定:

```sql
CREATE TABLE tbl1
(...)
PROPERTIES
(
"dynamic_partition.prop1" = "value1", "dynamic_partition.prop2" = "value2", ...
)
```

运行时修改

```sql
ALTER TABLE tbl1 SET
(
"dynamic_partition.prop1" = "value1",
"dynamic_partition.prop2" = "value2",
)
```



### **3.6.3** 动态分区规则参数



#### **3.6.3.1** 主要参数

动态分区的规则参数都以 dynamic_partition. 为前缀:

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220919171507699.png" alt="image-20220919171507699" style="zoom:50%;" />



#### **3.6.3.2** 创建历史分区的参数

⚫ dynamic_partition.create_history_partition
 默认为 false。当置为 true 时，Doris 会自动创建所有分区，当期望创建的分区个数大于 max_dynamic_partition_num 值时，操作将被禁止。当不指定 start 属性时，该参数不生效。

⚫ dynamic_partition.history_partition_num

当 create_history_partition 为 true 时，该参数用于指定创建历史分区数量。默认值为 - 1， 即未设置

⚫ dynamic_partition.hot_partition_num

指定最新的多少个分区为热分区。对于热分区，系统会自动设置其 storage_medium 参 数为 SSD，并且设置 storage_cooldown_time。

hot_partition_num 是往前 n 天和未来所有分区

我们举例说明。假设今天是 2021-05-20，按天分区，动态分区的属性设置为: hot_partition_num=2, end=3, start=-3。则系统会自动创建以下分区，并且设置 storage_medium 和 storage_cooldown_time 参数:

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220919171935441.png" alt="image-20220919171935441" style="zoom:50%;" />

⚫ dynamic_partition.reserved_history_periods

需要保留的历史分区的时间范围。当 dynamic_partition.time_unit 设置为 "DAY/WEEK/MONTH" 时，需要以 [yyyy-MM-dd,yyyy-MM-dd],[...,...] 格式进行设置。当 dynamic_partition.time_unit 设置为 "HOUR" 时，需要以 [yyyy-MM-dd HH:mm:ss,yyyy- MM-dd HH:mm:ss],[...,...] 的格式来进行设置。如果不设置，默认为 "NULL"。



#### **3.6.3.3** 创建历史分区规则

假设需要创建的历史分区数量为 expect_create_partition_num，根据不同的设置具体数 量如下:

(1)create_history_partition = true
 1 dynamic_partition.history_partition_num 未设置，即 -1.则 expect_create_partition_num = end - start;
 2 dynamic_partition.history_partition_num 已设置,则 expect_create_partition_num = end - max(start, -histoty_partition_num); (2)create_history_partition = false
 不会创建历史分区，expect_create_partition_num = end - 0;
 (3)当 expect_create_partition_num > max_dynamic_partition_num(默认 500)时，禁止创建过多分区。



# 四、数据导入和导出

# 五、查询

# 六、集成其他系统

# 七、监控和报警

# 八、使用优化

# 九、数据备份和恢复

