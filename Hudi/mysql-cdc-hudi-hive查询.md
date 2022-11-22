

## 准备工作

MySQL建表语句如下

```sql
user test;

create table users
(
    id bigint auto_increment primary key,
    name varchar(20) null,
    birthday timestamp default CURRENT_TIMESTAMP not null,
    ts timestamp default CURRENT_TIMESTAMP not null
);

create table users_sink
(
    id bigint auto_increment primary key,
    name varchar(20) null,
    birthday timestamp default CURRENT_TIMESTAMP not null,
    ts timestamp default CURRENT_TIMESTAMP not null
);

// 随意插入几条数据
insert into users (name) values ('hello1');
insert into users (name) values ('world');
insert into users (name) values ('iceberg');
insert into users (id,name) values (4,'spark');
insert into users (name) values ('hudi');

select * from users;
update users set name = 'hello spark'  where name = 'hello1';
delete from users where name = 'world';
```

#### Step.1 download Flink jar

Hudi works with both Flink 1.13 and Flink 1.14. You can follow the instructions [here](https://flink.apache.org/downloads) for setting up Flink. Then choose the desired Hudi-Flink bundle jar to work with different Flink and Scala versions:

- `hudi-flink1.13-bundle_2.11`
- `hudi-flink1.13-bundle_2.12`
- `hudi-flink1.14-bundle_2.11`
- `hudi-flink1.14-bundle_2.12`

#### Step.2 start Flink cluster

Start a standalone Flink cluster within hadoop environment. Before you start up the cluster, we suggest to config the cluster as follows:

- in `$FLINK_HOME/conf/flink-conf.yaml`, add config option `taskmanager.numberOfTaskSlots: 4`
- in `$FLINK_HOME/conf/flink-conf.yaml`, [add other global configurations according to the characteristics of your task](https://hudi.apache.org/docs/flink_configuration#global-configurations)
- in `$FLINK_HOME/conf/workers`, add item `localhost` as 4 lines so that there are 4 workers on the local cluster





启动flink集群 $FLINK_HOME/bin/sql-client.sh embedded 

```sql
-- 0.测试环境 可设置秒级别（不能太小），生产环境可设置分钟级别。
set execution.checkpointing.interval=30sec;

-- 2. Flink sql cdc DDL 
  CREATE TABLE mysql_users (
      id BIGINT PRIMARY KEY NOT ENFORCED ,
      name STRING,
      birthday TIMESTAMP(3),
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
      'table-name'= 'users'
  );

select * from mysql_users;

-- 3.创建一个临时视图，增加分区列 方便后续同步hive分区表
-- 说明：partition 关键字需要 `` 引起来
create view mycdc_v AS SELECT *, DATE_FORMAT(birthday, 'yyyyMMdd') as `partition` FROM mysql_users;


-- 4.Flinksql 创建 cdc sink hudi文件，并自动同步hive分区表DDL 语句

CREATE TABLE mysqlcdc_sync_hive(
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
'path'= 'hdfs://localhost:9000/user/hudi/mysqlcdc_sync_hive02',
'table.type'= 'MERGE_ON_READ', -- 默认COPY_ON_WRITE,可选MERGE_ON_READ 
'read.streaming.enabled'= 'true', -- 开启流读
'read.streaming.check-interval'= '3' -- 检查间隔，默认60s
);


-- 5. Flink sql mysql cdc数据写入hudi文件数据
insert into mysqlcdc_sync_hive select id,name,birthday,ts,`partition` from mycdc_v;


-- 6. 查询测试
select * from mysqlcdc_sync_hive;
```

