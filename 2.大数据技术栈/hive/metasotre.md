## **一、 metadata 、metastore**

Apache hive作为一款基于hadoop的数据仓库工具，其最大的魅力在于可以将位于hdfs的结构化数据映射成为一张表，然后提供了类sql的语法让用户使用来开展对数据的分析，是一款非常棒的离线批处理数仓工具。

在hive的具体使用中，首先面临的问题便是如何定义表结构信息，跟结构化的数据映射成功。所谓的映射指的是一种对应关系。在hive中我们需要描述清楚表跟文件之间的映射关系、列和字段之间的关系等等信息。

**我们把这些描述映射关系的数据的称之为hive的元数据。该数据十分重要，因为只有通过查询它才可以确定用户编写sql和最终操作文件之间的关系。**



为此我们需要搞清楚下述两个知识：

**Metadata**即元数据。元数据包含用Hive创建的database、table、表的字段等元信息。元数据存储在关系型数据库中。如hive内置的Derby、第三方如MySQL等。

**Metastore**即元数据服务，作用是：客户端连接metastore服务，metastore再去连接MySQL数据库来存取元数据。有了metastore服务，就可以有多个客户端同时连接，而且这些客户端不需要知道MySQL数据库的用户名和密码，只需要连接metastore 服务即可。

## **二、 metastore三种配置方式**

\1. 内嵌模式

内嵌模式使用的是内嵌的Derby数据库来存储元数据，也不需要额外起Metastore服务。数据库和Metastore服务都嵌入在主Hive Server进程中。这个是默认的，配置简单，但是一次只能一个客户端连接，适用于用来实验，不适用于生产环境。

解压hive安装包 bin/hive 启动即可使用

缺点：不同路径启动hive，每一个hive拥有一套自己的元数据，无法共享。

\2. 本地模式

本地模式采用外部数据库来存储元数据，目前支持的数据库有：MySQL、Postgres、Oracle、MS SQL Server.在这里我们使用MySQL。

本地模式不需要单独起metastore服务，用的是跟hive在同一个进程里的metastore服务。也就是说当你启动一个hive 服务，里面默认会帮我们启动一个metastore服务。

hive根据hive.metastore.uris 参数值来判断，如果为空，则为本地模式。

缺点是：每启动一次hive服务，都内置启动了一个metastore。

**3. 远程模式**

远程模式下，需要单独起metastore服务，然后每个客户端都在配置文件里配置连接到该metastore服务。远程模式的metastore服务和hive运行在不同的进程里。

在生产环境中，建议用远程模式来配置Hive Metastore。

在这种情况下，其他依赖hive的软件都可以通过Metastore访问hive。

远程模式下，需要配置hive.metastore.uris 参数来指定metastore服务运行的机器ip和端口，并且需要单独手动启动metastore服务。



## **三、 Hive Client、Beeline Client**

课程中采用远程模式部署hive的metastore服务。使用hive自带的客户端进行连接访问。

在node-1机器上，解压hive的安装包。

注意：以下两件事在启动hive之前必须确保正常完成。

1、选择某台机器提前安装mysql,确保具有远程访问的权限。

2、启动hadoop集群 确保集群正常健康

上传mysql jdbc的驱动包到hive的安装包lib目录下。

修改hive的配置文件。具体如下：

tar zxvf hive-1.1.0-cdh5.14.0.tar.gz

mv hive-1.1.0-cdh5.14.0 hive

cd hive/conf/

mv hive-env.sh.template hive-env.sh

vim conf/hive-env.sh

export HADOOP_HOME=/export/servers/hadoop-2.6.0-cdh5.14.0

vim hive-site.xml 内容如下：

```
<configuration>
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://node-1:3306/hive?createDatabaseIfNotExist=true</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
</property>
 
<property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
</property>
 
<property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>hadoop</value>
</property>
 
<property>
    <name>hive.server2.thrift.bind.host</name>
    <value>node-1</value>
</property>
 
<property>
    <name>hive.metastore.uris</name>
    <value>thrift://node-1:9083</value>
</property>
</configuration>
```

**1. metastore 的启动方式**

##### 前台启动：

```
hive --service metastore
```



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220315124005397.png" alt="image-20220315124005397" style="zoom:50%;" />

该启动方式下，进程会一直占用shell终端前台。如果ctrl+c结束进程，则hive metastore服务也会同时关闭。

##### 后台启动

```
hive --service metastore &
```

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220315124110299.png" alt="image-20220315124110299" style="zoom:50%;" />

推荐使用后台启动的方式。

后台启动的情况下，如果想关闭metastore服务 使用jps查看进程，kill -9 进程号即可。



**2. 第一代客户端Hive Client**

在hive安装包的bin目录下，有hive提供的第一代客户端 bin/hive。使用该客户端可以访问hive的metastore服务。从而达到操作hive的目的。

如果需要在其他机器上通过该客户端访问hive metastore服务，只需要在该机器的hive-site.xml配置中添加metastore服务地址即可。

上传hive安装包到另一个机器上，比如node-3：

tar zxvf hive-1.1.0-cdh5.14.0.tar.gz

mv hive-1.1.0-cdh5.14.0 hive

cd hive/conf/

mv hive-env.sh.template hive-env.sh

vim conf/hive-env.sh

export HADOOP_HOME=/export/servers/hadoop-2.6.0-cdh5.14.0

vim hive-site.xml 内容如下：

```
<configuration>
<property>
    <name>hive.server2.thrift.bind.host</name>
    <value>node-1</value>
</property>
 
<property>
    <name>hive.metastore.uris</name>
    <value>thrift://node-1:9083</value>
</property>
</configuration>
```

使用下面的命令启动hive的客户端：

./bin/hive

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220315124213186.png" alt="image-20220315124213186" style="zoom:50%;" />

可以发现官方提示：第一代客户端已经不推荐使用了。

**3、 第二代客户端Hive Beeline Client**

hive经过发展，推出了第二代客户端beeline，但是beeline客户端不是直接访问metastore服务的，而是需要单独启动hiveserver2服务。

在hive运行的服务器上，首先启动metastore服务，然后启动hiveserver2服务。

```
nohup /export/servers/hive/bin/hive --service metastore &

nohup /export/servers/hive/bin/hive --service hiveserver2 &
```



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220315124255869.png" alt="image-20220315124255869" style="zoom:50%;" />

在node-3上使用beeline客户端进行连接访问。

/export/servers/hive/bin/beeline

Beeline version 1.1.0-cdh5.14.0 by Apache Hive

beeline> ! connect jdbc:hive2://node-1:10000

Enter username for jdbc:hive2://node-1:10000: root

Enter password for jdbc:hive2://node-1:10000: *******

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220315124311295.png" alt="image-20220315124311295" style="zoom:50%;" />