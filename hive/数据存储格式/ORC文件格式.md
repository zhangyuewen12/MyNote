### **Orc 格式**

Orc (Optimized Row Columnar)是 Hive 0.11 版里引入的新的存储格式。如下图所示可以看到每个 Orc 文件由 1 个或多个 stripe 组成，每个 stripe 一般为 HDFS 的块大小，每一个 stripe 包含多条记录，这些记录按照列进行独立存储，对应到 Parquet中的 row group 的概念。每个 Stripe 里有三部分组成，分别是 Index Data，Row Data，Stripe Footer：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220424121829342.png" alt="image-20220424121829342" style="zoom:50%;" />



1）Index Data：一个轻量级的 index，默认是每隔 1W 行做一个索引。这里做的索引应该只是记录某行的各字段在 Row Data 中的 offset。 

2）Row Data：存的是具体的数据，先取部分行，然后对这些行按列进行存储。对每个列进行了编码，分成多个 Stream 来存储。 

3）Stripe Footer：存的是各个 Stream 的类型，长度等信息。每个文件有一个 File Footer，这里面存的是每个 Stripe 的行数，每个 Column 的数据类

型信息等；每个文件的尾部是一个 PostScript，这里面记录了整个文件的压缩类型以及FileFooter 的长度信息等。在读取文件时，会 seek 到文件尾部读 PostScript，从里面解析到File Footer 长度，再读 FileFooter，从里面解析到各个 Stripe 信息，再读各个 Stripe，即从后往前读



**index data** includes min and max values for each column and the row positions within each column. 

**stripe footer** contains a directory of stream locations.

 **row data** is used in table scans.



二、ORC文件格式

​    ORC文件也是以二进制方式存储的，所以是不可以直接读取，ORC文件也是自解析的，它包含许多的元数据，这些元数据都是同构ProtoBuffer进行序列化的。ORC的文件结构如下图



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220422143748340.png" alt="image-20220422143748340" style="zoom:50%;" />

  在ORC文件中保存了三个层级的统计信息，分别为**文件级别、stripe级别和row group**级别的，他们都可以用来根据Search ARGuments（**谓词下推条件**）判断是否可以跳过某些数据，在统计信息中都包含成员数和是否有null值，并且对于不同类型的数据设置一些特定的统计信息



（1）file level

　　在ORC文件的末尾会记录文件级别的统计信息，会记录整个文件中columns的统计信息。这些信息主要用于查询的优化，也可以为一些简单的聚合查询比如max, min, sum输出结果。

（2）stripe level

　　ORC文件会保存每个字段stripe级别的统计信息，ORC reader使用这些统计信息来确定对于一个查询语句来说，需要读入哪些stripe中的记录。比如说某个stripe的字段max(a)=10，min(a)=3，那么当where条件为a >10或者a <3时，那么这个stripe中的所有记录在查询语句执行时不会被读入。

（3）row level

　　为了进一步的避免读入不必要的数据，在逻辑上将一个**column的index以一个给定的值(默认为10000，可由参数配置)分割为多个index组（也就是对每一列分组，然后基于分组数据建立索引，这样能近一步减少不必要的查询）**。以10000条记录为一个组，对数据进行统计（比如min,max等）。Hive查询引擎会将where条件中的约束传递给ORC reader，这些reader根据组级别的统计信息，过滤掉不必要的数据。如果该值设置的太小，就会保存更多的统计信息，用户需要根据自己数据的特点权衡一个合理的值