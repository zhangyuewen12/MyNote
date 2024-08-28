# yarn查看hdfs container日志

# yarn container本地日志位置



flink程序里面打印的日志会输出到taskmanager.log，job结束后，yarn日志会聚合到hdfs，
running的时候能看taskmanager日志，但是flink history server的界面上就是看不了，
找到了hdfs上taskmanager的位置，纠结要不要去看下flink history server源码读的哪儿…



# flink taskmanager日志地址

运行时查询日志，显示正常
http://ip:8088/proxy/application_1606889053812_25820/#/task-manager/container_e03_1606889053812_25891_01_000002/logs

# yarn container本地日志位置

```
ls -hl  /data02/yarn/container-logs/application_1606889053812_25820/container_e03_1606889053812_25891_01_000002/
total 72K
-rw-r----- 1 geosmart yarn 740 Feb  1 15:46 taskmanager.err
-rw-r----- 1 geosmart yarn 62K Feb  1 15:47 taskmanager.log
-rw-r----- 1 geosmart yarn 442 Feb  1 15:47 taskmanager.out
```

# yarn已开启日志聚合

```
yarn.log-aggregation-enable=true
yarn.log-aggregation.retain-seconds=604800
```

# yarn查看hdfs container日志

job执行结束后，taskmanager日志聚合到了hdfs目录：

path: /tmp/logs/{user_id}/logs/{application_id}/

```
hdfs dfs -ls /tmp/logs/geosmart/logs/application_1606889053812_25820/
Found 2 items
-rw-r-----   3 geosmart hadoop      75341 2021-02-01 16:11 /tmp/logs/geosmart/logs/application_1606889053812_25820/ip_8041

-rw-r-----   3 geosmart hadoop      57557 2021-02-01 16:11 /tmp/logs/geosmart/logs/application_1606889053812_25820/hz-hadoop-test-199-151-41_8041

```



# 通过yarn logs 查看container日志

yarn logs 命令可以查看具体的应用 或容器启动日志

yarn logs -appOwner {user} -applicationId {application_id} -show_container_log_info

```
yarn logs  -appOwner geosmart -applicationId application_1606889053812_25820  -show_container_log_info
```



eg: 

```
1.yarn logs -applicationId {application_id} -show_application_log_info
Container: containerid1
Container: containerid1
Container: containerid1
2.yarn logs 
```

