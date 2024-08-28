# Key Generation

Every record in Hudi is uniquely identified by a primary key, which is a pair of record key and partition path where the record belongs to. Using primary keys, Hudi can impose a) partition level uniqueness integrity constraint b) enable fast updates and deletes on records. One should choose the partitioning scheme wisely as it could be a determining factor for your ingestion and query latency.

In general, Hudi supports both partitioned and global indexes. For a dataset with partitioned index(which is most commonly used), each record is uniquely identified by a pair of record key and partition path. But for a dataset with global index, each record is uniquely identified by just the record key. There won't be any duplicate record keys across partitions.

> 每条Hudi中的记录都由主键唯一标识。
>
> 使用主键，Hudi提供两种功能：1. 分区级别的唯一性、完整性约束。 2. 加速记录更新和删除。
>
> 分区健 影响数据插入和查询性能。
>
> Hudi支出分区和全局索引。对于分区表，每一条记录都是由 主键和分区路径 唯一约束。

## Key Generators

Hudi provides several key generators out of the box that users can use based on their need, while having a pluggable implementation for users to implement and use their own KeyGenerator. This page goes over all different types of key generators that are readily available to use.

[Here](https://github.com/apache/hudi/blob/6f9b02decb5bb2b83709b1b6ec04a97e4d102c11/hudi-common/src/main/java/org/apache/hudi/keygen/KeyGenerator.java) is the interface for KeyGenerator in Hudi for your reference.

Before diving into different types of key generators, let’s go over some of the common configs required to be set for key generators.

>  主键生成器，Hudi提供了几个开箱即用的主键生成器，用户可以根据自己需要选择使用。同时，用户可以实现和是使用自己的主键生成器。



主键生成器的通用配置



| Config                                            | Meaning/purpose                                              |
| ------------------------------------------------- | ------------------------------------------------------------ |
| `hoodie.datasource.write.recordkey.field`         | Refers to record key field. This is a mandatory field.       |
| `hoodie.datasource.write.partitionpath.field`     | Refers to partition path field. This is a mandatory field.   |
| `hoodie.datasource.write.keygenerator.class`      | Refers to Key generator class(including full path). Could refer to any of the available ones or user defined one. This is a mandatory field. |
| `hoodie.datasource.write.partitionpath.urlencode` | When set to true, partition path will be url encoded. Default value is false. |
| `hoodie.datasource.write.hive_style_partitioning` | When set to true, uses hive style partitioning. Partition field name will be prefixed to the value. Format: “<partition_path_field_name>=<partition_path_value>”. Default value is false. |