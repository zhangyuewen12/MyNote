# Hadoop伪分布式搭建

## 一、配置文件

### 0.profile.sh

```shell
export HADOOP_HOME=/Users/zyw/hadoopapp/hadoop-3.1.1
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export JAVA_LIBRARY_PATH=$HADOOP_HOME/lib/native
export HDFS_NAMENODE_USER=zyw
export HDFS_DATANODE_USER=zyw
export HDFS_SECONDARYNAMENODE_USER=zyw
export YARN_RESORUCENAMAGER_USER=zyw
export YARN_NODEMANAGER_USER=zyw
```



### 1. core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/Users/zyw/hadoopapp/data/tmp</value>
    </property>
</configuration>
```



### 2. hdfs-site.xml

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <!-- Determines where on the local filesystem an DFS data node should store its blocks.  -->
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/Users/zyw/hadoopapp/data/datanode</value>
  </property>
  <property>
    <name>dfs.namenode.data.dir</name>
    <value>/Users/zyw/hadoopapp/data/namenode</value>
  </property>
<configuration>
```



### 3. yarn-site.xml

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
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





### 4. mapred-site.xml

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



### 5. hadoop-env.sh 、yarn-env.sh

```
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_281.jdk/Contents/Home
```



## 二、启动

```
启动前格式化名称节点，启动全部进程

命令：hdfs namenode -format

          (格式化名称节点，即格式化HDFS的NameNode)

start-all.sh  （启动全部进程）

或者 start-dfs.sh（启动namenode，secondnamenode，datanode进程。）

格式化名称节点：

注意：

（1）多次格式化可能会报错，直接删除data文件夹重新格式化即可，会自动生成data文件夹

（2）格式化时进行到

Re-format filesystem in Storage Directory root= /home/elf/setup/hadoop-3.1.3/tmp/dfs/name; location= null ? (Y or N)  

时一定要输入大写的 Y，否则可能失败。

```

