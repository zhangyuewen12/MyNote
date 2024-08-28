# Hive本地环境搭建

## 一、下载解压

```
tar -xzvf hive-x.y.z.tar.gz
```

## 二、配置HIVE_HOME环境变量

```
export PATH=$HIVE_HOME/bin:$PATH
```

## 三、HDFS上创建目录并赋权限

```shell
$HADOOP_HOME/bin/hadoop fs -mkdir       /tmp
$HADOOP_HOME/bin/hadoop fs -mkdir   -p    /user/hive/warehouse
$HADOOP_HOME/bin/hadoop fs -chmod g+w   /tmp
$HADOOP_HOME/bin/hadoop fs -chmod g+w   /user/hive/warehouse
```

> - you must have Hadoop in your path OR
> - `export HADOOP_HOME=<hadoop-install-dir>`

## 四、下载mysql驱动

打开[MySQL官网](https://dev.mysql.com/downloads/connector/j/)选择 Platform Independent --> Download -> 解压 -> 将mysql-connector-java-8.0.19.jar放入到${HIVE_HOME}/lib目录下

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210618144900210.png" alt="image-20210618144900210" style="zoom:50%;" />

## 五、创建数据库

```
在MySQL中创建metastore数据库, 可以通过终端、Navicat等数据库客户端来创建一个数据库。

mysql> create database hive_metastore;

# 可以创建一个用户也可以直接使用root账号，此步骤可以省略
mysql> create user 'hiveuser'@'*' identified by '12345678';
mysql> grant all privileges on hive_metastore.* to 'hiveuser'@'localhost';
mysql> flush privileges;
```





## 六、Hive配置-hive-site.xml

```xml
<property>
<name>javax.jdo.option.ConnectionURL</name>
<value>jdbc:mysql://localhost:3306/hive_metastore?createDatabaseIfNotExist=true</value>
</property>
<property>
<name>javax.jdo.option.ConnectionDriverName</name>
<value>com.mysql.cj.jdbc.Driver</value>
</property>
<!--mysql用户名-->
<property>
<name>javax.jdo.option.ConnectionUserName</name>
<value>root</value>
</property>
<!--mysql密码-->
<property>
<name>javax.jdo.option.ConnectionPassword</name>
<value>root123</value>
</property>

<!-- hive.exec.local.scratchdir 是Hive配置中的一个参数，用于指定本地临时目录路径，这个临时目录用于在执行Hive查询过程中存储一些中间数据和临时文件。这些中间数据和临时文件在查询执行期间可能会被创建、使用和删除。-->
<property>
  <name>hive.exec.local.scratchdir</name>
  <value>${HIVE_HOME}/</value>
  <description>Local scratch space for Hive jobs</description>
</property>
```



## 七、Hadoop配置-core-site.xml

```
<!--配置所有节点的zyw用户都可作为代理用户-->
<property>
    <name>hadoop.proxyuser.zyw.hosts</name>
    <value>*</value>
</property>

<!--配置zyw用户能够代理的用户组为任意组-->
<property>
    <name>hadoop.proxyuser.zyw.groups</name>
    <value>*</value>
</property>
<!--配置zyw用户能够代理的用户为任意用户-->
<property>
    <name>hadoop.proxyuser.zyw.users</name>
 		<value>*</value>
</property>

```



## 八、初始化metastore数据库

```shell
${HIVE_HOME}/bin目录下

1.$HIVE_HOME/bin/schematool -dbType <db type> -initSchema
schematool -initSchema -dbType mysql
OR
schematool -initSchema -dbType derby


```

## 九、启动

```
1. 启动元数据服务
$HIVE_HOME/bin/hive --service  hiveserver2 
2. 启动beeine   
$HIVE_HOME/bin/beeline -u jdbc:hive2://localhost:10000 -n zyw
```



# 报错一

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230825154841884.png" alt="image-20230825154841884" style="zoom:50%;" />

# 报错二

![image-20230829143509446](/Users/zyw/Library/Application Support/typora-user-images/image-20230829143509446.png)

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230825155918604.png" alt="image-20230825155918604" style="zoom:50%;" />



```
<!-- hiveserver2的高可用参数，开启此参数可以提高hiveserver2的启动速度 -->
<property>
    <name>hive.server2.active.passive.ha.enable</name>
    <value>true</value>
</property>
```

