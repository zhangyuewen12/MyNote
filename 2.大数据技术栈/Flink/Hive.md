## Flink Hive支持

[Apache Hive](https://hive.apache.org/) has established itself as a focal point of the data warehousing ecosystem. It serves as not only a SQL engine for big data analytics and ETL, but also a data management platform, where data is discovered, defined, and evolved.

Flink offers a two-fold integration with Hive.

The first is to leverage Hive’s Metastore as a persistent catalog with Flink’s `HiveCatalog` for storing Flink specific metadata across sessions. For example, users can store their Kafka or ElasticSearch tables in Hive Metastore by using `HiveCatalog`, and reuse them later on in SQL queries.

```
第一种使用HIve的方式是,通过Flink的HiveCataLog跨会话存储Flink特定的元数据，把Hive元数据作为持久化catalog。例如，用户可以存储在Hive metastore中存储kafka、es table。在之后的SQL查询中重复使用。
```



The second is to offer Flink as an alternative engine for reading and writing Hive tables.

```
第二种方式是给Flink提供一种读取或者写入hive的引擎。
```



The `HiveCatalog` is designed to be “out of the box” compatible with existing Hive installations. You do not need to modify your existing Hive Metastore or change the data placement or partitioning of your tables.

## Hive Catalog

Hive Metastore has evolved into the de facto metadata hub over the years in Hadoop ecosystem. Many companies have a single Hive Metastore service instance in their production to manage all of their metadata, either Hive metadata or non-Hive metadata, as the source of truth.

```
在Hadoop生态系统中，hive metastore 多年来已经演变成事实上的元数据中心。许多公司在生产中都有一个Hive Metastore服务实例来管理所有元数据，无论是Hive元数据还是非Hive元数据，作为真相的来源。
```



For users who have both Hive and Flink deployments, `HiveCatalog` enables them to use Hive Metastore to manage Flink’s metadata.

For users who have just Flink deployment, `HiveCatalog` is the only persistent catalog provided out-of-box by Flink. Without a persistent catalog, users using [Flink SQL CREATE DDL](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/create/) have to repeatedly create meta-objects like a Kafka table in each session, which wastes a lot of time. `HiveCatalog` fills this gap by empowering users to create tables and other meta-objects only once, and reference and manage them with convenience later on across sessions.