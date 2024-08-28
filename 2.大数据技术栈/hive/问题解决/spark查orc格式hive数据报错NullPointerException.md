https://blog.csdn.net/m0_37813354/article/details/104635667



然后问题解决

补充：

![image-20220422144656016](/Users/zyw/Library/Application Support/typora-user-images/image-20220422144656016.png)

hive.exec.orc.split.strategy参数控制在读取ORC表时生成split的策略。BI策略以文件为粒度进行split划分；ETL策略会将文件进行切分，多个stripe组成一个split；HYBRID策略为：当文件的平均大小大于hadoop最大split值（默认256 * 1024 * 1024）时使用ETL策略，否则使用BI策略。 --- 对于一些较大的ORC表，可能其footer较大，ETL策略可能会导致其从hdfs拉取大量的数据来切分split，甚至会导致driver端OOM，因此这类表的读取建议使用BI策略。对于一些较小的尤其有数据倾斜的表（这里的数据倾斜指大量stripe存储于少数文件中），建议使用ETL策略。--- 另外，spark.hadoop.mapreduce.input.fileinputformat.split.minsize参数可以控制在ORC切分时stripe的合并处理。具体逻辑是，当几个stripe的大小小于spark.hadoop.mapreduce.input.fileinputformat.split.minsize时，会合并到一个task中处理。可以适当调小该值，以此增大读ORC表的并发。