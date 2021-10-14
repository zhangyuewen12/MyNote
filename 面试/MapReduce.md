# [Mapreduce执行过程详解](https://www.cnblogs.com/javajetty/p/10755705.html)

MapReduce运行时，首先通过Map读取HDFS中的数据，然后经过拆分，将每个文件中的每行数据分拆成键值对，最后输出作为Reduce的输入，大体执行流程如下图所示：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20211014131048834.png" alt="image-20211014131048834" style="zoom:55%;" />

整个流程图具体来说：每个Mapper任务是一个java进程，它会读取HDFS中的文件，解析成很多的键值对，经过我们覆盖的map方法处理后，转换为很多的键值对再输出，整个Mapper任务的处理过程又可以分为以下几个阶段，如图所示。

![image-20211014131115771](/Users/zyw/Library/Application Support/typora-user-images/image-20211014131115771.png)

**在上图中，把Mapper任务的运行过程分为六个阶段**。

- 第一阶段是把输入文件按照一定的标准分片 (InputSplit)，每个输入片的大小是固定的。默认情况下，输入片(InputSplit)的大小与数据块(Block)的大小是相同的。如果数据块(Block)的大小是默认值64MB，输入文件有两个，一个是32MB，一个是72MB。那么小的文件是一个输入片，大文件会分为两个数据块，那么是两个输入片，一共产生三个输入片。每一个输入片由一个Mapper进程处理，这里的三个输入片，会有三个Mapper进程处理。
- 第二阶段是对输入片中的记录按照一定的规则解析成键值对，有个默认规则是把每一行文本内容解析成键值对，这里的“键”是每一行的起始位置(单位是字节)，“值”是本行的文本内容。
- 第三阶段是调用Mapper类中的map方法，在第二阶段中解析出来的每一个键值对，调用一次map方法，如果有1000个键值对，就会调用1000次map方法，每一次调用map方法会输出零个或者多个键值对。
- 第四阶段是按照一定的规则对第三阶段输出的键值对进行分区，分区是基于键进行的，比如我们的键表示省份(如北京、上海、山东等)，那么就可以按照不同省份进行分区，同一个省份的键值对划分到一个区中。默认情况下只有一个区，分区的数量就是Reducer任务运行的数量，因此默认只有一个Reducer任务。
- 第五阶段是对每个分区中的键值对进行排序。首先，按照键进行排序，对于键相同的键值对，按照值进行排序。比如三个键值 对<2,2>、<1,3>、<2,1>，键和值分别是整数。那么排序后的结果 是<1,3>、<2,1>、<2,2>。如果有第六阶段，那么进入第六阶段；如果没有，直接输出到本地的linux 文件中。
- 第六阶段是对数据进行归约处理，也就是reduce处理，通常情况下的Comber过程，键相等的键值对会调用一次reduce方法，经过这一阶段，数据量会减少，归约后的数据输出到本地的linxu文件中。本阶段默认是没有的，需要用户自己增加这一阶段的代码。

### 二、Reducer任务的执行过程详解

   每个Reducer任务是一个java进程。Reducer任务接收Mapper任务的输出，归约处理后写入到HDFS中，可以分为如下图所示的几个阶段

![image-20211014131355825](/Users/zyw/Library/Application Support/typora-user-images/image-20211014131355825.png)

1、第一阶段是Reducer任务会主动从Mapper任务复制其输出的键值对，Mapper任务可能会有很多，因此Reducer会复制多个Mapper的输出。

2、第二阶段是把复制到Reducer本地数据，全部进行合并，即把分散的数据合并成一个大的数据，再对合并后的数据排序。

3、第三阶段是对排序后的键值对调用reduce方法，键相等的键值对调用一次reduce方法，每次调用会产生零个或者多个键值对，最后把这些输出的键值对写入到HDFS文件中。



### 三、键值对的编号

  在对Mapper任务、Reducer任务的分析过程中，会看到很多阶段都出现了键值对，这里对键值对进行编号，方便理解键值对的变化情况，如下图所示

![image-20211014131552211](/Users/zyw/Library/Application Support/typora-user-images/image-20211014131552211.png)

在上图中，对于Mapper任务输入的键值对，定义为key1和value1，在map方法中处理后，输出的键值对，定义为key2和value2，reduce方法接收key2和value2处理后，输出key3和value3。在下文讨论键值对时，可能把key1和value1简写 为<k1,v1>，key2和value2简写为<k2,v2>，key3和value3简写为<k3,v3>。



## shuffle

  mapreduce确保每个reduce的输入都是按照键值排序的，系统执行排序，将map的输入作为reduce的输入过程称之为shuffle过程。shuffle也是我们优化的重点部分。shuffle流程图如下图所示

![image-20211014131826663](/Users/zyw/Library/Application Support/typora-user-images/image-20211014131826663.png)

### map端

在生成map之前，会计算文件分片的大小：计算源码详见：[hadoop2.7作业提交详解之文件分片](https://www.cnblogs.com/zsql/p/11276584.html)

  然后会根据分片的大小计算map的个数，对每一个分片都会产生一个map作业，或者是一个文件（小于分片大小*1.1）生成一个map作业，然后通过自定的map方法进行自定义的逻辑计算，计算完毕后会写到本地磁盘。

   在这里不是直接写入磁盘，为了保证IO效率，采用了先写入内存的环形缓冲区，并做一次预排序（快速排序）。缓冲区的大小默认为100MB（可通过修改配置项mpareduce.task.io.sort.mb进行修改），当写入内存缓冲区的大小到达一定比例时，默认为80%（可通过mapreduce.map.sort.spill.percent配置项修改）,将启动一个溢写线程将内存缓冲区的内容溢写到磁盘（spill to disk），这个溢写线程是独立的，不影响map向缓冲区写结果的线程，在溢写到磁盘的过程中，map继续输入到缓冲中，如果期间缓冲区被填满，则map写会被阻塞到溢写磁盘过程完成。溢写是通过轮询的方式将缓冲区中的内存写入到本地mapreduce.cluster.local.dir目录下。在溢写到磁盘之前，我们会知道reduce的数量，然后会根据reduce的数量划分分区，默认根据hashpartition对溢写的数据写入到相对应的分区。在每个分区中，后台线程会根据key进行排序，所以溢写到磁盘的文件是分区且排序的。如果有combiner函数，它在排序后的输出运行，使得map输出更紧凑。减少写到磁盘的数据和传输给reduce的数据。

  每次环形换冲区的内存达到阈值时，就会溢写到一个新的文件，因此当一个map溢写完之后，本地会存在多个分区且排序的文件。在map完成之前会把这些文件合并成一个分区且排序(归并排序)的文件，可以通过参数mapreduce.task.io.sort.factor控制每次可以合并多少个文件。

  在map溢写磁盘的过程中，对数据进行压缩可以提交速度的传输，减少磁盘io，减少存储。默认情况下不压缩，使用参数mapreduce.map.output.compress控制，压缩算法使用mapreduce.map.output.compress.codec参数控制。



### 2.2、reduce端

​    map任务完成后，监控作业状态的application master便知道map的执行情况，并启动reduce任务，application master并且知道map输出和主机之间的对应映射关系，reduce轮询application master便知道主机所要复制的数据。

   **一个Map任务的输出，可能被多个Reduce任务抓取**。**每个Reduce任务可能需要多个Map任务的输出作为其特殊的输入文件，**而每个Map任务的完成时间可能不同，当有一个Map任务完成时，Reduce任务就开始运行。Reduce任务根据分区号在多个Map输出中抓取（fetch）对应分区的数据，这个过程也就是Shuffle的copy过程。reduce有少量的复制线程，因此能够并行的复制map的输出，默认为5个线程。可以通过参数mapreduce.reduce.shuffle.parallelcopies控制。

  这个复制过程和map写入磁盘过程类似，也有阀值和内存大小，阀值一样可以在配置文件里配置，而内存大小是直接使用reduce的tasktracker的内存大小，复制时候reduce还会进行排序操作和合并文件操作。

  如果map输出很小，则会被复制到Reducer所在节点的内存缓冲区，缓冲区的大小可以通过mapred-site.xml文件中的mapreduce.reduce.shuffle.input.buffer.percent指定。一旦Reducer所在节点的内存缓冲区达到阀值，或者缓冲区中的文件数达到阀值，则合并溢写到磁盘。

  如果map输出较大，则直接被复制到Reducer所在节点的磁盘中。随着Reducer所在节点的磁盘中溢写文件增多，后台线程会将它们合并为更大且有序的文件。当完成复制map输出，进入sort阶段。这个阶段通过归并排序逐步将多个map输出小文件合并成大文件。最后几个通过归并合并成的大文件作为reduce的输出。