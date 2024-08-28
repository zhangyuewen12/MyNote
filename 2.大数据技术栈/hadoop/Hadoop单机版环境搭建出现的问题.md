# 问题一、there is no HDFS_NAMENODE_USER defined
[问题链接](https://stackoverflow.com/questions/48129029/hdfs-namenode-user-hdfs-datanode-user-hdfs-secondarynamenode-user-not-defined)

```
ERROR: Attempting to operate on hdfs namenode as root
ERROR: but there is no HDFS_NAMENODE_USER defined. Aborting operation.
Starting datanodes
ERROR: Attempting to operate on hdfs datanode as root
ERROR: but there is no HDFS_DATANODE_USER defined. Aborting operation.
Starting secondary namenodes [master]
ERROR: Attempting to operate on hdfs secondarynamenode as root
ERROR: but there is no HDFS_SECONDARYNAMENODE_USER defined. Aborting operation.
Starting resourcemanager
ERROR: Attempting to operate on yarn resourcemanager as root
ERROR: but there is no YARN_RESOURCEMANAGER_USER defined. Aborting operation.
Starting nodemanagers
ERROR: Attempting to operate on yarn nodemanager as root
ERROR: but there is no YARN_NODEMANAGER_USER defined. Aborting operation.
```
解决办法：
```
The root cause of this problem,

hadoop install for different user and you start yarn service for different user. OR
in hadoop config's hadoop-env.sh specified HDFS_NAMENODE_USER and HDFS_DATANODE_USER user is something else.
Hence we need to correct and make it consistent at every place. So a simple solution of this problem is to edit your hadoop-env.sh file and add the user-name for which you want to start the yarn service. So go ahead and edit $HADOOP_HOME/etc/hadoop/hadoop-env.sh by adding the following lines

export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root


Now save and start yarn, hdfs service and check that it works.

我们在hadoop-env.sh文件中也可以找到如下的描述

To prevent accidents, shell commands be (superficially) locked to only allow certain users to execute certain subcommands.

为了防止发生意外，仅（部分）锁定shell命令以仅允许某些用户执行某些子命令。

It uses the format of (command)_(subcommand)_USER.For example, to limit who can execute the namenode command,export HDFS_NAMENODE_USER=hdfs

使用“命令_子命令_用户”，例如，通过使用export HDFS_NAMENODE_USER=hdfs来限制哪个用户可以执行namenode命令。

参考stackoverflow上的相关讨论
```