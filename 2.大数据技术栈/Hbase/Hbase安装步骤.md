# 1. Zookeeper正常部署

首先保证Zookeeper集群的正常部署，并启动之：

```shell
[atguigu@hadoop102 zookeeper-3.4.10]$ bin/zkServer.sh start

[atguigu@hadoop103 zookeeper-3.4.10]$ bin/zkServer.sh start

[atguigu@hadoop104 zookeeper-3.4.10]$ bin/zkServer.sh start
```

# 2. Hadoop正常部署

Hadoop集群的正常部署并启动：

```shell
[atguigu@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh

[atguigu@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh
```



# 3. HBase的安装



```shell
1. 解压HBase到指定目录：
[atguigu@hadoop102 software]$ tar -zxvf hbase-1.3.1-bin.tar.gz -C /opt/module

2. conf/hbase-env.sh修改内容：
export JAVA_HOME=/opt/module/jdk1.8.0_144
export HBASE_MANAGES_ZK=false


3. hbase-site.xml修改内容：
<configuration>
    <!-- NameNode rpc端口 -->
	<property>     
		<name>hbase.rootdir</name>     
		<value>hdfs://hadoop100:9000/hbase</value>   
	</property>

	<property>   
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	</property>

   <!-- 0.98后的新变动，之前版本没有.port,默认端口为60000 -->
	<property>
		<name>hbase.master.port</name>
		<value>16000</value>
	</property>

	<property>   
		<name>hbase.zookeeper.quorum</name>
	     <value>hadoop100:2181,hadoop101:2181,hadoop102:2181</value>
	</property>

	<property>   
		<name>hbase.zookeeper.property.dataDir</name>
	     <value>/opt/module/apache-zookeeper-3.6.3-bin/data</value>
	</property>
</configuration>

4. 修改 regionservers：
hadoop102
hadoop103
hadoop104

5. 建立软连接
软连接hadoop配置文件到hbase：
[atguigu@hadoop102 module]$ ln -s /opt/module/hadoop-2.7.2/etc/hadoop/core-site.xml 
/opt/module/hbase/conf/core-site.xml
[atguigu@hadoop102 module]$ ln -s /opt/module/hadoop-2.7.2/etc/hadoop/hdfs-site.xml 
/opt/module/hbase/conf/hdfs-site.xml
```

# 4. HBase远程发送到其他集群

```shell
[atguigu@hadoop102 module]$ xsync hbase/ 
```



# 5. HBase服务的启动



```shell
1．启动方式1

[atguigu@hadoop102 hbase]$ bin/hbase-daemon.sh start master
[atguigu@hadoop102 hbase]$ bin/hbase-daemon.sh start regionserver
提示：如果集群之间的节点时间不同步，会导致regionserver无法启动，抛出ClockOutOfSyncException异常。
修复提示：

a、同步时间服务
请参看帮助文档：《尚硅谷大数据技术之Hadoop入门》
b、属性：hbase.master.maxclockskew设置更大的值
<property>
        <name>hbase.master.maxclockskew</name>
        <value>180000</value>
        <description>Time difference of regionserver from master</description>
 </property>
 
2. 启动方式2
[atguigu@hadoop102 hbase]$ bin/start-hbase.sh
对应的停止服务：
[atguigu@hadoop102 hbase]$ bin/stop-hbase.sh
```

# 6.查看HBase页面

启动成功后，可以通过“host:port”的方式来访问HBase管理页面，例如：

[http://hadoop100:16010](http://linux01:16010) 

