链接https://mp.weixin.qq.com/s/5kv2KNI5-_ZazSc5J1STkQ



环境说明

```
hadoop-3.3.1
hive-3.1.2
flink-1.13.5 scala-2.12
flink-sql-connector-mysql-cdc-2.1.1.jar、flink-format-changelog-json-2.1.1.jar
hudi-0.10.0

flink 环境说明
1. 添加jar包
    1.flink-sql-connector-mysql-cdc-2.1.1.jar
    2.flink-format-changelog-json-2.1.1.jar
    3.hudi-flink-bundle_2.12-0.10.0.jar
```

Flink 环境启动

```
1. 启动hdfs服务
start-dfs.sh
http://localhost:9870/
2. 启动yarn服务
start-yarn.sh
http://localhost:8088/
3. 启动flink集群
$FLINK_HOME/bin/yarn-session.sh -nm ys-hudi01 -d

2. 启动sql $FLINK_HOME/bin/sql-client.sh embedded -j ./lib/hudi-flink-bundle_2.11-0.10.0.jar shell
```

Hive 环境启动

```
1.启动hiveserver2
$HIVE_HOME/bin/hiveserver2

2.启动beeine
 2.1  beeline 
 2.2 !connect jdbc:hive2://localhost:10000
```



### **Mysql 创建 mysql sources 表 DDL**

```sql
create table users_cdc(
   id bigint auto_increment primary key,
   name varchar(20) null,
   birthday timestamp default CURRENT_TIMESTAMP not null,
   ts timestamp default CURRENT_TIMESTAMP not null
);

insert into users_cdc(name) values("zhangsan");
```



#### **Flink sql cdc DDL 语句**

```sql
$FLINK_HOME/bin/sql-client.sh embedded -j ./lib/hudi-flink-bundle_2.11-0.10.0.jar shell

CREATE TABLE mysql_users (
    id BIGINT,
    name String,
    birthday TIMESTAMP(3),
    ts TIMESTAMP(3),
    PRIMARY KEY (id) NOT ENFORCED
) WITH (
'connector'= 'mysql-cdc',
'hostname'= 'localhost',
'port'= '3306',
'username'= 'root',
'password'= '12345678',
'server-time-zone'= 'Asia/Shanghai',
'debezium.snapshot.mode'='initial',
'database-name'= 'test',
'table-name'= 'users_cdc'
);

Flink SQL> select * from mysql_users;


-- 创建一个临时视图，增加分区列方便后续同步 Hive 分区表
Flink SQL> create view mycdc_v AS SELECT *, DATE_FORMAT(birthday, 'yyyyMMdd') as `partition` FROM mysql_users;
[INFO] Execute statement succeed.


-- 查询临时视图
Flink SQL> select * from mycdc_v;

-- 设置 checkpoint 间隔时间，存储路径已在 flink-conf 配置设置全局路径
Flink SQL> set execution.checkpointing.interval=30sec;
[INFO] Session property has been set.



-- Flinksql 创建 cdc sink hudi 文件，并自动同步 Hive 分区表 DDL 语句
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
'path'= 'hdfs://localhost:9000/user/hive/warehouse/hudi/mysqlcdc_sync_hive01'
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
, 'hive_sync.enable'= 'true'-- 开启自动同步hive
, 'hive_sync.mode'= 'hms'-- 自动同步hive模式，默认jdbc模式
-- , 'hive_sync.metastore.uris'= 'thrift://hadoop:9083'-- hive metastore地址
, 'hive_sync.jdbc_url'= 'jdbc:hive2://localhost:10000'-- hiveServer地址
, 'hive_sync.table'= 'mysqlcdc_sync_hive01'-- hive 新建表名
, 'hive_sync.db'= 'odl'-- hive 新建数据库名
, 'hive_sync.username'= ''-- HMS 用户名
, 'hive_sync.password'= ''-- HMS 密码
, 'hive_sync.support_timestamp'= 'true'-- 兼容hive timestamp类型
);

-- Flink sql mysql cdc 数据写入 Hudi 文件数据
Flink SQL> insert into mysqlcdc_sync_hive01 select id,name,birthday,ts,`partition` from mycdc_v;

-- 查询hudi对应的表
Flink SQL> select * from mysqlcdc_sync_hive01;
```



```sql
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
'path'= 'hdfs://localhost:9000/user/hive/warehouse/odl.db/mysqlcdc_sync_hive01', 'hoodie.datasource.write.recordkey.field'= 'id', 
'read.streaming.enabled' = 'true',
'table.type'= 'MERGE_ON_READ',
'hive_sync.support_timestamp'= 'true'
);
```



### **Hive 访问 Hudi 数据**

**说明：**需要引入 hudi-hadoop-mr-bundle-0.10.0-SNAPSHOT.jar

####  

```
#### **引入 Hudi 依赖 jar 方式：**

**
**

1. 引入到 $HIVE_HOME/lib 下；

2. 引入到 $HIVE_HOME/auxlib 自定义第三方依赖 修改 hive-site.xml 配置文件；

3. Hive shell 命令行引入 Session 级别有效；
其中（1）和（3）配置完后需要重启 hive-server 服务;


hive> select * from mysqlcdc_sync_hive01_ro; --已查询到mysq insert的一条数据
```

