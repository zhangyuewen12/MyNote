

```sql
create table users_cdc(
   id bigint auto_increment primary key,
   name varchar(20) null,
   birthday DATE default '2022-10-01' not null,
   ts timestamp default CURRENT_TIMESTAMP not null
);

insert into users_cdc(name) values('zhansan');
insert into users_cdc(name) values('zhansan');
insert into users_cdc(name) values('zhansan');
insert into users_cdc(name) values('zhansan');

CREATE TABLE mysql_users (
    id BIGINT,
    name String,
    birthday DATE,
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


EXECUTE CDCSOURCE ods_job_test with
(
	'connector' = 'mysql-cdc',
  'hostname' = 'localhost',
  'port' = '3306',
  'username' = 'root',
  'password' = '12345678',
  'scan.startup.mode' = 'initial',
  'table-name' = 'test\.users_cdc',
  'sink.connector' = 'datastream-connector',
  'sink.brokers' = 'localhost:9092'
)
```

