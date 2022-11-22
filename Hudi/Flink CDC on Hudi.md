

## 准备工作

MySQL建表语句如下

```
create table users
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
update users set name = 'hello spark'  where id = 5;
delete from users where id = 5;
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





启动flink集群

```sql
$FLINK_HOME/bin/sql-client.sh embedded 
 
//1.创建 mysql-cdc
CREATE TABLE mysql_users (
                             id BIGINT PRIMARY KEY NOT ENFORCED ,
                             name STRING,
                             birthday TIMESTAMP(3),
                             ts TIMESTAMP(3)
) WITH (
      'connector' = 'mysql-cdc',
      'hostname' = 'localhost',
      'port' = '3306',
      'username' = 'root',
      'password' = '12345678',
      'server-time-zone' = 'Asia/Shanghai',
      'database-name' = 'test',
      'table-name' = 'users'
      );
      
// 2.创建hudi表
CREATE TABLE hudi_users2
(
    id BIGINT PRIMARY KEY NOT ENFORCED,
    name STRING,
    birthday TIMESTAMP(3),
    ts TIMESTAMP(3),
    `partition` VARCHAR(20)
) PARTITIONED BY (`partition`) WITH (
    'connector' = 'hudi',
    'table.type' = 'MERGE_ON_READ',
    'path' = 'hdfs://localhost:9000/user/hudi/warehouse/test.db/hudi_users2',
    'read.streaming.enabled' = 'true',
    'read.streaming.check-interval' = '4' 
);

//3.mysql-cdc 写入hudi ，会提交有一个flink任务
INSERT INTO hudi_users2 SELECT *, DATE_FORMAT(birthday, 'yyyyMMdd') FROM mysql_users;

set 'execution.checkpointing.interval' = '60s';

set execution.result-mode=tableau;

select * from hudi_users2;
```

