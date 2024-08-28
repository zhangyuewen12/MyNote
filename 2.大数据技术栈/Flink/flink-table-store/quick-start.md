**Step 5: Create a Catalog and a Table**

```sql
Create a Catalog and a Table

-- if you're trying out Table Store in a distributed environment,
-- warehouse path should be set to a shared file system, such as HDFS or OSS
-- CREATE CATALOG my_catalog WITH (
--    'type'='table-store',
--    'warehouse'='file:/tmp/table_store'
-- );

CREATE CATALOG my_catalog WITH (
    'type'='table-store',
    'warehouse'='hdfs:///user/flink/warehouse/table_store'
);

USE CATALOG my_catalog;

-- create a word count table
CREATE TABLE word_count (
    word STRING PRIMARY KEY NOT ENFORCED,
    cnt BIGINT
);

```



**Step 6: Write Data**

```sql

-- create a word data generator table
CREATE TEMPORARY TABLE word_table (
    word STRING
) WITH (
    'connector' = 'datagen',
    'fields.word.length' = '1'
);

-- table store requires checkpoint interval in streaming mode
SET 'execution.checkpointing.interval' = '10 s';

-- write streaming data to dynamic table
INSERT INTO word_count SELECT word, COUNT(*) FROM word_table GROUP BY word;
```