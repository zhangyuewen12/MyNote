









## Flink SQL

Once the flink Hudi tables have been registered to the Flink catalog, it can be queried using the Flink SQL. It supports all query types across both Hudi table types, relying on the custom Hudi input formats again like Hive. Typically notebook users and Flink SQL CLI users leverage flink sql for querying Hudi tables. Please add hudi-flink-bundle as described in the [Flink Quickstart](https://hudi.apache.org/cn/docs/flink-quick-start-guide).

By default, Flink SQL will try to use its own parquet reader instead of Hive SerDe when reading from Hive metastore parquet tables.

```sql
# HADOOP_HOME is your hadoop root directory after unpack the binary package.
export HADOOP_CLASSPATH=`$HADOOP_HOME/bin/hadoop classpath`

./bin/sql-client.sh embedded -j .../hudi-flink-bundle_2.1?-*.*.*.jar shell
```



```sql
-- this defines a COPY_ON_WRITE table named 't1'
CREATE TABLE t1(
  uuid VARCHAR(20), -- you can use 'PRIMARY KEY NOT ENFORCED' syntax to specify the field as record key
  name VARCHAR(10),
  age INT,
  ts TIMESTAMP(3),
  `partition` VARCHAR(20)
)
PARTITIONED BY (`partition`)
WITH (
  'connector' = 'hudi',
  'path' = 'table_base+path'
);

-- query the data
select * from t1 where `partition` = 'par1';
```

Flink's built-in support parquet is used for both COPY_ON_WRITE and MERGE_ON_READ tables, additionally partition prune is applied by Flink engine internally if a partition path is specified in the filter. Filters push down is not supported yet (already on the roadmap).

For MERGE_ON_READ table, in order to query hudi table as a streaming, you need to add option `'read.streaming.enabled' = 'true'`, when querying the table, a Flink streaming pipeline starts and never ends until the user cancel the job manually. You can specify the start commit with option `read.streaming.start-commit` and source monitoring interval with option `read.streaming.check-interval`.

### Streaming Query

By default, the hoodie table is read as batch, that is to read the latest snapshot data set and returns. Turns on the streaming read mode by setting option `read.streaming.enabled` as `true`. Sets up option `read.start-commit` to specify the read start offset, specifies the value as `earliest` if you want to consume all the history data set.

#### Options

| Option Name                      | Required | Default           | Remarks                                                      |
| -------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| `read.streaming.enabled`         | false    | `false`           | Specify `true` to read as streaming                          |
| `read.start-commit`              | false    | the latest commit | Start commit time in format 'yyyyMMddHHmmss', use `earliest` to consume from the start commit |
| `read.streaming.skip_compaction` | false    | `false`           | Whether to skip compaction commits while reading, generally for two purposes: 1) Avoid consuming duplications from the compaction instants 2) When change log mode is enabled, to only consume change logs for right semantics. |
| `clean.retain_commits`           | false    | `10`              | The max number of commits to retain before cleaning, when change log mode is enabled, tweaks this option to adjust the change log live time. For example, the default strategy keeps 50 minutes of change logs if the checkpoint interval is set up as 5 minutes. |

