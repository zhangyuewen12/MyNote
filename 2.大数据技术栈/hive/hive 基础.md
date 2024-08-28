#  hive 基础
## 一、什么是hive？
1 Hive简介

Hive是由Facebook开源，基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。

那为什么会有Hive呢？它是为了解决什么问题而诞生的呢？

下面通过一个案例，来快速了解一下Hive。

例如：需求，统计单词出现个数。

（1）在Hadoop课程中我们用MapReduce程序实现的，当时需要写Mapper、Reducer和Driver三个类，并实现对应逻辑，相对繁琐。

test表

**id****列**

 

atguigu

atguigu

ss

ss

jiao

banzhang

xue

hadoop

（2）如果通过Hive SQL实现，一行就搞定了，简单方便，容易理解。

select count(*) from test group by id;

## 二、Hive的体系结构
<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230829133640543.png" alt="image-20230829133640543" style="zoom:50%;" />

**1****）用户接口：Client**

CLI（command-line interface）、JDBC/ODBC。

说明：JDBC和ODBC的区别。

（1）JDBC的移植性比ODBC好；（通常情况下，安装完ODBC驱动程序之后，还需要经过确定的配置才能够应用。而不相同的配置在不相同数据库服务器之间不能够通用。所以，安装一次就需要再配置一次。JDBC只需要选取适当的JDBC数据库驱动程序，就不需要额外的配置。在安装过程中，JDBC数据库驱动程序会自己完成有关的配置。）

（2）两者使用的语言不同，JDBC在Java编程时使用，ODBC一般在C/C++编程时使用。

**2****）元数据：Metastore**

元数据包括：数据库（默认是default）、表名、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等。

默认存储在自带的derby数据库中，由于derby数据库只支持单客户端访问，生产环境中为了多人开发，推荐使用MySQL存储Metastore。

**3****）驱动器：Driver**

（1）解析器（SQLParser）：将SQL字符串转换成抽象语法树（AST）

（2）语义分析（Semantic Analyzer）：将AST进一步划分为QeuryBlock

（3）逻辑计划生成器（Logical Plan Gen）：将语法树生成逻辑计划

（4）逻辑优化器（Logical Optimizer）：对逻辑计划进行优化

（5）物理计划生成器（Physical Plan Gen）：根据优化后的逻辑计划生成物理计划

（6）物理优化器（Physical Optimizer）：对物理计划进行优化

（7）执行器（Execution）：执行该计划，得到查询结果并返回给客户端

**4****）****Hadoop**

使用HDFS进行存储，可以选择MapReduce/Tez/Spark进行计算。





### hiveserver2服务

Hive的hiveserver2服务的作用是提供jdbc/odbc接口，为用户提供远程访问Hive数据的功能，例如用户期望在个人电脑中访问远程服务中的Hive数据，就需要用到Hiveserver2。

![image-20230829135557325](/Users/zyw/Library/Application Support/typora-user-images/image-20230829135557325.png)