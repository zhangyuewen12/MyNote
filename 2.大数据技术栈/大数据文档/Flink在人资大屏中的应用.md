# 一、Flink基础介绍

## 1.1 简介https://flink.apache.org/

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221118232828470.png" alt="image-20221118232828470" style="zoom:50%;" />

Flink 的具体定位是一个分布式处理引擎，如图所示，用于对无界和有界数据流进行有状态计算。Flink 被设计在所有常见的集群环境中运行，以内存执行速度和任意规模来执行计算。

```
Flink 功能强大，支持开发和运行多种不同种类的应用程序。它的主要特性包括：批流一体化、精密的状态管理、事件时间支持以及精确一次的状态一致性保障等。在启用高可用选项的情况下，它不存在单点失效问题。事实证明，Flink 已经可以扩展到数千核心，其状态可以达到 TB 级别，且仍能保持高吞吐、低延迟的特性。世界各地有很多要求严苛的流处理应用都运行在 Flink 之上。
核心点：
1、高吞吐，低延迟
2、结果的准确性
3、精确一次的状态一致性保证
4、高可用，支持动态扩展
```



**flink**：Flink是基于事件驱动的，是面向流的处理框架, Flink基于每个事件一行一行地流式处理，是真正的流式计算. 另外他也可以基于流来模拟批进行计算实现批处理。

**spark**：Spark的技术理念是使用微批来模拟流的计算,基于Micro-batch,数据流以时间为单位被切分为一个个批次,通过分布式数据集RDD进行批量处理,是一种伪实时。



# 二、人资大屏Demo演示

业务场景介绍

![](/Users/zyw/Library/Application Support/typora-user-images/image-20230320181758727.png)



## 2.1 数据数据初始化

```sql
use demo;

drop table if exists demo.user_info;
CREATE TABLE demo.user_info(
  id int NOT NULL PRIMARY KEY AUTO_INCREMENT,
  name char(20),
  gender char(10),
  position_level_id varchar(10)
);

drop table if exists demo.position_level_info;
CREATE TABLE demo.position_level_info(
  id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
  position_level_id varchar(10),
  position_level varchar(10)
);

INSERT INTO user_info(name,gender,position_level_id) values ('张三','男','100001');
INSERT INTO user_info(name,gender,position_level_id) values ('李四','男','100001');
INSERT INTO user_info(name,gender,position_level_id) values ('王五','男','100001');
INSERT INTO user_info(name,gender,position_level_id) values ('lili','女','100002');
INSERT INTO user_info(name,gender,position_level_id) values ('Mark','男','100002');
INSERT INTO user_info(name,gender,position_level_id) values ('Anne','女','100003');
INSERT INTO user_info(name,gender,position_level_id) values ('李纯','女','100003');
INSERT INTO user_info(name,gender,position_level_id) values ('孙甜','女','100003');
INSERT INTO user_info(name,gender,position_level_id) values ('蓝心','女','100003');
INSERT INTO user_info(name,gender,position_level_id) values ('赵龙','男','100003');

INSERT INTO position_level_info(position_level_id,position_level) values ('100001','1级');
INSERT INTO position_level_info(position_level_id,position_level) values ('100002','2级');
INSERT INTO position_level_info(position_level_id,position_level) values ('100003','3级');
```



## 2.2 环境部署

### 2.2.1 启动hadoop环境

```
start-all.sh
```

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221206121447361.png" alt="image-20221206121447361" style="zoom:50%;" />



### 2.2.2 启动Flink集群  

```
./bin/yarn-session.sh

Yarn WebUI: 
http://localhost:8088/cluster
```



![image-20221206121547296](/Users/zyw/Library/Application Support/typora-user-images/image-20221206121547296.png)

### 2.2.3 启动Sql-client

```
./bin/sql-client.sh
```



### 2.2.4 启动ES,Kibana

```
/Users/zyw/hadoopapp/elasticsearch-7.1.0
/Users/zyw/hadoopapp/kibaå
```



## 2.3 流作业

```sql
set execution.checkpointing.interval=5s;

-- source1.用户信息表
CREATE TABLE user_info (
    id BIGINT,
  	name String,
  	gender String,
  	position_level_id String,
    PRIMARY KEY (id) NOT ENFORCED
) WITH (
'connector'= 'mysql-cdc',
'hostname'= 'localhost',
'port'= '3306',
'username'= 'root',
'password'= '12345678',
'server-time-zone'= 'Asia/Shanghai',
'debezium.snapshot.mode'='initial',
'database-name'= 'demo',
'table-name'= 'user_info'
);

-- source2.职级信息表
CREATE TABLE position_level_info (
    id BIGINT,
  	position_level_id String,
 		position_level String,
    PRIMARY KEY (id) NOT ENFORCED
) WITH (
'connector'= 'mysql-cdc',
'hostname'= 'localhost',
'port'= '3306',
'username'= 'root',
'password'= '12345678',
'server-time-zone'= 'Asia/Shanghai',
'debezium.snapshot.mode'='initial',
'database-name'= 'demo',
'table-name'= 'position_level_info'
);



-- 建立一个视图
create view v_user_info_model as 
select
a.name as name,
a.gender as gender,
b.position_level_id as position_level_id,
b.position_level as position_level
from 
user_info a
left join 
position_level_info b
on a.position_level_id = b.position_level_id;



-- sink1. 用户性别统计结果表
CREATE TABLE sink_user_gender_analysis (
  gender STRING,
  num BIGINT,
  PRIMARY KEY (gender) NOT ENFORCED
) WITH (
  'connector' = 'elasticsearch-7',
  'hosts' = 'http://localhost:9200',
  'index' = 'user_gender'
);

-- sink2. 用户职级统计结果表
CREATE TABLE sink_user_position_level_analysis (
  position_level String,
  num BIGINT,
  PRIMARY KEY (position_level) NOT ENFORCED
) WITH (
  'connector' = 'elasticsearch-7',
  'hosts' = 'http://localhost:9200',
  'index' = 'user_position_level'
);

-- 分析不同性别的人数
insert into sink_user_gender_analysis
select
gender,
count(1) as num
from v_user_info_model
group by
gender;

-- 分析不同职级的人数
insert into sink_user_position_level_analysis
select
position_level,
count(1) as number
from 
v_user_info_model
group by 
position_level;
```



## 2.4 需求验证

```sql
-- 1. 人员入职入职、离职
INSERT INTO user_info(name,gender,position_level_id) values ('test','男','100001');
delete from user_info where name = 'test';

-- 2. 职级变更
update user_info set position_level_id ='100002' where id = 1;
```



```
GET /user_gender/_search
{
  "query": {
    "match_all": {}
  }
}

GET /user_position_level/_search
{
  "query": {
    "match_all": {}
  }
}

```





## 2.5 总结

> ## 批处理
>
> - 批处理主要操作大容量静态数据集，并在计算过程完成后返回结果。
>
>   可以认为，处理的是用同一个固定时间间隔分组的数据点集合。批处理模式中使用的数据集通常符合下列特征：
>
>   - 有界：批处理数据集代表数据的有限集合
>   - 持久：数据通常始终存储在某种类型的持久存储位置中
>   - 大量：批处理操作通常是处理极为海量数据集的唯一方法
>
> 
>
> ## 流处理
>
> - 流数据可以对对随时进入系统的数据进行计算。流处理方式无需针对整个数据集执行操作，而是对通过系统传输的每个数据执行操作。流处理中的数据集是 “无边界 ” 的， 这就产生了集合重要的影响：
>   - 可以处理几乎无限量的数据，但同一时间只能处理一条数据，不同记录间只维持最少量的状态
>   - 处理工作是基于事件的，除非明确停止否则没有 “尽头”
>   - 处理结果可用，并会随着新数据的抵达继续更新。



Flink基于事件流的计算

![image-20221206191752552](/Users/zyw/Library/Application Support/typora-user-images/image-20221206191752552.png)