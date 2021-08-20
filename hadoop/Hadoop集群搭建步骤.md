# HADOOP集群搭建步骤

## 一、修改配置文件

修改core-site.xml

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop150:9000</value>
    </property>
   <!-- 配置 hadoop.tmp.dir 路径到持久化目录 -->
   <property>
        <name>hadoop.tmp.dir</name>
        <value>/root/hadoop_data/tmp</value>
   </property>
</configuration>
```

> 
>
> |                |                          |                                         |
> | -------------- | ------------------------ | --------------------------------------- |
> | hadoop.tmp.dir | /tmp/hadoop-${user.name} | A base for other temporary directories. |
> |                |                          |                                         |
>
> hadoop.tmp.dir是hadoop文件系统依赖的基础配置，很多路径都依赖它。它默认的位置是在/tmp/{$user}下面，但是在/tmp路径下的存储是不安全的，因为linux一次重启，文件就可能被删除。
>
> 按照hadoop Getting Start中Single Node Setup一节中的步骤走下来之后，伪分布式已经运行起来了。怎么更改默认的hadoop.tmp.dir路径，并使其生效?请按照下面的步骤来：
>
> 1、编辑conf/core-site.xml,在里面加上如下属性:
>
> <property>
>   		<name>hadoop.tmp.dir</name>
>   		<value>/home/had/hadoop/data</value>
>  		<description>A base for other temporary directories.</description>
> </property>
>
> 2、停止hadoop:   bin/stop-all.sh
> 3、重新格式化namenode节点。bin/hadoop namenode -format
>
>        注意：此处至关重要，否则namenode会启动不起来。
> 4、启动 bin/start-all.sh
>
> 5、测试bin/hadoop fs -put conf conf

修改hdfs-site.xml

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```



修改mapred-site.xml

```
<configuration>
 <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>

```



修改yarn-site.xml

```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
      <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_HOME,PATH,LANG,TZ,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```



## 二、配置集群works

> Slaves File
> List all worker hostnames or IP addresses in your etc/hadoop/workers file, one per line. Helper scripts (described below) will use the etc/hadoop/workers file to run commands on many hosts at once. It is not used for any of the Java-based Hadoop configuration. In order to use this functionality, ssh trusts (via either passphraseless ssh or some other means, such as Kerberos) must be established for the accounts used to run Hadoo

```
hadoop151
hadoop152
```



## 三、修改变量

 

### 配置hadoop的JAVA_HOME

> Unpack the downloaded Hadoop distribution. In the distribution, edit the file etc/hadoop/hadoop-env.sh to define some parameters as follows:

```
# set to the root of your Java installation
export JAVA_HOME=/usr/java/latest
```



### 修改系统环境修改/etc/profile

```
export HADOOP_HOME=/root/hadoop-3.3.0
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
```

> export HDFS_NAMENODE_USER=root
> export HDFS_DATANODE_USER=root
> export HDFS_SECONDARYNAMENODE_USER=root
>
> 如果不配置这写变量的话，直接使用 $HADOOP_HOME/sbin/start-dfs.sh命令会被拒绝执行

