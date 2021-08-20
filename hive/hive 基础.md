#  hive 基础
## 一、什么是hive？
* 构建在Hadoop上的数据仓库平台，为数据仓库管理提供了许多功能
* 
* 起源自facebook由Jeff Hammerbacher领导的团队
* 
* 2008年facebook把hive项目贡献给Apache
* 
* 定义了一种类SQL语言HiveQL。可以看成是仍SQL到Map-Reduce的映射器
* 
* 提供Hive shell、JDBC/ODBC、Thrift客户端等连接
*

## 二、Hive的体系结构
![](https://github.com/MrQuJL/hadoop-guide/raw/master/11-Hive%E5%9F%BA%E7%A1%80/imgs/hivearc.png)

* 用户接口主要有三个：CLI，JDBC/ODBC和 WebUI

CLI，即Shell命令行
JDBC/ODBC 是 Hive 的Java，与使用传统数据库JDBC的方式类似
WebGUI是通过浏览器访问 Hive

* Hive 将元数据存储在数据库中(metastore)，目前只支持 mysql、derby。Hive 中的元数据包括表的名字，表的列和分区及其属性，表的属性（是否为外部表等），表的数据所在目录等
* 解释器、编译器、优化器完成 HQL 查询语句从词法分析、语法分析、编译、优化以及查询计划（plan）的生成。生成的查询计划存储在 HDFS 中，并在随后有 MapReduce 调用执行
* Hive 的数据存储在 HDFS 中，大部分的查询由 MapReduce 完成（包含 * 的查询，比如 select * from table 不会生成 MapRedcue 任务）