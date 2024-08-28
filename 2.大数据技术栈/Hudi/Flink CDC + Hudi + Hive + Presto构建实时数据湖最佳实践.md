# 1. 测试过程环境版本说明 

> Flink1.13.6
>
> Scala2.12
>
> Hadoop3.0.0
>
> Hive2.1.1
>
> Hudi0.12
>
> PrestoDB0.256
>
> Mysql5.7



## 2.环境启动

> 1. 启动HDFS服务 start-dfs.sh
> 2. 启动Yarn服务 start-yarn.sh
> 3. 启动Flink集群 $FLINK_HOME/bin/yarn-session.sh -s 2 -jm 2048 -tm 2048 -nm ys-hudi01 -d
> 4. 启动sql-client 



# 3.执行步骤

```sql
-- 0.测试环境 可设置秒级别（不能太小），生产环境可设置分钟级别。
set execution.checkpointing.interval=30sec;

-- 1. mysql 建表
create table users_cdc(
   id bigint auto_increment primary key,
   name varchar(20) null,
   birthday timestamp default CURRENT_TIMESTAMP not null,
   ts timestamp default CURRENT_TIMESTAMP not null
);

-- 2. Flink sql cdc DDL 
CREATE TABLE mysql_users (
    id BIGINT PRIMARY KEY NOT ENFORCED ,
    name STRING,
    birthday DATE,
    ts TIMESTAMP(3)
) WITH (
    'connector'= 'mysql-cdc',
    'hostname'= 'localhost',
    'port'= '3306',
    'username'= 'root',
    'password'='12345678',
    'server-time-zone'= 'Asia/Shanghai',
    'debezium.snapshot.mode'='initial',
    'database-name'= 'test',
    'table-name'= 'users_cdc'
);

-- 3.创建一个临时视图，增加分区列 方便后续同步hive分区表
-- 说明：partition 关键字需要 `` 引起来
create view mycdc_v AS SELECT *, DATE_FORMAT(birthday, 'yyyyMMdd') as `partition` FROM mysql_users;


-- 4.Flinksql 创建 cdc sink hudi文件，并自动同步hive分区表DDL 语句

CREATE TABLE mysqlcdc_sync_hive01(
id bigint ,
name string,
birthday TIMESTAMP(3),
ts TIMESTAMP(3),
`partition` VARCHAR(20),
primary key(id) not enforced --必须指定uuid 主键
)
PARTITIONED BY (`partition`)
with(
'connector'='hudi',
'path'= 'hdfs://localhost:9000/user/hudi/mysqlcdc_sync_hive01'
, 'hoodie.datasource.write.recordkey.field'= 'id'-- 主键
, 'write.precombine.field'= 'ts'-- 自动precombine的字段
, 'write.tasks'= '1'
, 'compaction.tasks'= '1'
, 'write.rate.limit'= '2000'-- 限速
, 'table.type'= 'MERGE_ON_READ'-- 默认COPY_ON_WRITE,可选MERGE_ON_READ 
, 'compaction.async.enabled'= 'true'-- 是否开启异步压缩
, 'compaction.trigger.strategy'= 'num_commits'-- 按次数压缩
, 'compaction.delta_commits'= '1'-- 默认为5
, 'changelog.enabled'= 'true'-- 开启changelog变更
, 'read.streaming.enabled'= 'true'-- 开启流读
, 'read.streaming.check-interval'= '3'-- 检查间隔，默认60s
);


-- 5. Flink sql mysql cdc数据写入hudi文件数据
insert into mysqlcdc_sync_hive01 select id,name,birthday,ts,`partition` from mycdc_v;


select * from mysqlcdc_sync_hive01;

```



# 4.执行结果

### Mysql数据源未写入测试数据

> 说明：目前还没写入测试数据，hudi目录只生成一些状态标记文件，还未生成分区目录以及.log 和.parquet数据文件，具体含义可见hudi官方文档。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220902105000205.png" alt="image-20220902105000205" style="zoom:50%;" />



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220902105102708.png" alt="image-20220902105102708" style="zoom:50%;" />



### Mysql数据源写入测试数据

#### 操作步骤

```sql

-- 设置sql-client 模式
set execution.result-mode=tableau;

select * from mysql_users;

-- 1.0 mysql写入数据
insert into users_cdc (name) values ('cdc01');
```



#### 执行结果

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220902105523934.png" alt="image-20220902105523934" style="zoom:50%;" />



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220902105630959.png" alt="image-20220902105630959" style="zoom:50%;" />



![image-20220902105831116](/Users/zyw/Library/Application Support/typora-user-images/image-20220902105831116.png)

![image-20220902105757873](/Users/zyw/Library/Application Support/typora-user-images/image-20220902105757873.png)



#### 执行说明