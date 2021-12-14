# 问题一、hive的基本架构，角色，与hdfs的关系？

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20211018141515359.png" alt="image-20211018141515359" style="zoom:50%;" />

```
Drvier组件
　　　　该组件是Hive的核心组件，该组件包括Complier(编译器)、Optimizer(优化器)和Executor(执行器),它们的作用是对Hive SQL语句进行解析、编译优化、生成执行计划，然后调用底层MR计算框架。
　　　　
MetaStore组件
　　　　该组件是Hive用来负责管理元数据的组件。Hive的元数据存储在关系型数据库中，其支持的关系型数据库有Derby和mysql，其中Derby是Hive默认情况下使用的数据库，它内嵌在Hive中，但是该数据库只支持单会话，也就是说只允许一个会话链接，所以在生产中并不适用，其实其实在平时我们的测试环境中也很少使用。在我们日常的团队开发中，需要支持多会话，所以需要一个独立的元数据库，用的最多的也就是Mysql，而且Hive内部对Mysql提供了很好的支持。
　　　　
CLI
　　　　Hive的命令行接口

Thrift Server
　　　　该组件提供JDBC和ODBC接入的能力，用来进行可扩展且跨语言的服务开发。Hive集成了该服务，能让不同的编程语言调用Hive的接口.
　　　　
Hive Web Interface
　　　　该组件是Hive客户端提供的一种通过网页方式访问Hive所提供的服务。这个接口对应Hive的HWI组件
```

Hive通过CLI，JDBC/ODBC或HWI接受相关的Hive SQL查询，并通过Driver组件进行编译，分析优化，最后编程可执行的MapReduce任务，但是具体里面是怎么执行的，看图：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20211018141936622.png" alt="image-20211018141936622" style="zoom:50%;" />



# 问题二、order by和sort by的区别

> 

order by实现的是全局排序，在hive mr引擎中将会只有1个reduce。而使用sort by会起多个reduce，只会在每个reduce中排序，如果不指定分组的话，跑出来的数据看起来是杂乱无章的，如果指定reduce个数是1，那么结果和order by是一致的，如下图，不指定的情况，两种结果对比：
<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20211018142253720.png" alt="image-20211018142253720" style="zoom:50%;" />

order by一般配合group by使用，而group by需要配合聚合函数使用，举个例子

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20211018142322956.png" alt="image-20211018142322956" style="zoom:50%;" />

而sort by分组时需要使用distribute by，和group by类似，但是它不需要配合聚合函数使用，也就不影响原数据的函数，这点和开窗函数有点类似，如下

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20211018142434022.png" alt="image-20211018142434022" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20211018142444891.png" alt="image-20211018142444891" style="zoom:50%;" />