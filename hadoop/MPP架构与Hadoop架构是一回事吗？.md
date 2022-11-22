计算机领域的很多概念都存在一些传播上的“谬误”。



MPP这个概念就是其中之一。它的“谬误”之处在于，明明叫做“Massively Parallel Processing（大规模并行处理）”，却让非常多的人拿它与大规模并行处理领域最著名的开源框架Hadoop相关框架做对比，这实在是让人困惑——难道Hadoop不是“大规模并行处理”架构了？



很多人在对比两者时，其实并不知道MPP的含义究竟是什么、两者的可比性到底在哪里。实际上，当人们在对比两者时，与其说是对比架构，不如说是对比产品。虽然MPP的原意是“大规模并行处理”，但由于一些历史原因，现在当人们说到MPP架构时，它们实际上指代的是“分布式数据库”，而Hadoop架构指的则是以Hadoop项目为基础的一系列分布式计算和存储框架。不过由于MPP的字面意思，现实中还是经常有人纠结两者到底有什么联系和区别，两者到底是不是同一个层面的概念。



### 到底什么是MPP架构？





MPP架构与Hadoop架构在理论基础上几乎是在讲同一件事，即，把大规模数据的计算和存储分布到不同的独立的节点中去做。



有人可能会问：“既然如此，为什么人们不说Hadoop是MPP（大规模并行处理）架构呢？”





关于这个问题嘛，请先问是不是，再问为什么。





在GreenPlum的官方文档中就写道：“Hadoop就是一种常见的MPP存储与分析工具。Spark也是一种MPP架构。”来看下面的图，更能体会到两者的相似性。



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220804130541370.png" alt="image-20220804130541370" style="zoom:50%;" />

问：这是什么架构？





答：MPP架构。



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220804130625836.png" alt="image-20220804130625836" style="zoom:50%;" />

相信了解过MPP架构的读者对这幅图不会陌生。也许在不同的分布式数据库产品中，节点角色的名称会有差异，但总体而言都是一个主节点加上多个从节点的架构。

但是，还可以有其他答案，比如MapReduce on Yarn：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220804130658437.png" alt="image-20220804130658437" style="zoom:50%;" />

这幅图或许大家有些陌生，但只不过是省略了资源调度的简化版MapReduce运行时架构罢了。

当然，还可以有更多答案，如Spark：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220804130721786.png" alt="image-20220804130721786" style="zoom:50%;" />

自然还可以是Flink：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220804130739579.png" alt="image-20220804130739579" style="zoom:50%;" />

有人可能会说，虽然直观上这些架构长得很像，但是MPP架构中的Master所负责的事情是不是与其他框架不一样？

那么，MPP架构的Master做的什么事呢？它会接收SQL语句，解析它并生成执行计划，将计划分发到各个节点。那么，这与Spark SQL有区别吗？不仅与Spark SQL没有区别，与其他任何Hadoop生态圈类似架构如Hive SQL、Flink SQL都没有区别。对于非SQL的输入，逻辑也是一致的，只是没有了解析SQL的步骤，但还是会生成执行图分发到各个节点去执行，执行结果也可以在主节点进行汇总。





不仅是在计算上没有区别，存储架构上也没有区别。下面是HDFS的架构图：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220804130831853.png" alt="image-20220804130831853" style="zoom:50%;" />

所以回到最初说的那句话——MPP架构与Hadoop架构在理论基础上几乎是在讲同一件事，即，把大规模数据的计算和存储分布到不同的独立的节点中去做。上面的几幅架构图印证了这一点。

既然MPP架构与Hadoop架构本质上是一回事，那么为什么很多人还要将两者分开讨论呢？我们可能经常听到这样的话：“这个项目的架构是MPP架构。”这似乎有意在说：“这可不是Hadoop那一套哦。”



这就与MPP架构的历史有关系。虽然从理论基础上两者是一回事，但是MPP架构与Hadoop架构的发展却是走的两条路线。MPP架构虽然也是指的“大规模并行处理”，但是由于提出者是数据库厂商，所以MPP架构在很多人眼中就成了“分布式数据库”的代名词，它处理的也都是“结构化”的数据，常常作为企业数据仓库的解决方案。



而Hadoop生态圈是根正苗红伴随着“大数据”兴起而发展起来的概念，它所要解决的是大规模数据量的存储和计算，它的提出者也并非数据库厂商，而是有着C端数据的互联网企业。因此Hadoop架构虽然也解决“大规模并行处理”，但没有了数据库那一套东西的限制，处理的也大多是“非结构化”的数据（自然在最初阶段也少了相关的优化）。当然，Hadoop生态圈也要考虑“结构化”的数据，这时Hive就成了Hadoop生态圈的数据仓库解决方案。但是，Hadoop、Spark等框架的理论基础与分布式数据库仍然是一样的。



广义上讲，MPP架构是一种更高层次的概念，它的含义就是字面含义，但是它本身并没有规定如何去实现。Hadoop相关框架和各个分布式数据库产品则是具体的实现。狭义上讲，**MPP架构成了分布式数据库这种体系架构的代名词**，而Hadoop架构指的是以Hadoop框架为基础的一套生态圈。



本文并不想仅仅从较高层次的架构设计来说明两者是一回事，这样还是缺乏说服力。下面，我们从分布式计算框架中最重要的过程——Shuffle——来展示两者更多的相似性。

### 数据重分区



Shuffle是分布式计算框架中最重要的概念与过程之一。在MPP架构（分布式数据库）中，这个数据重分区的过程与Hadoop相关框架在计算中的数据重分区过程也是一致的。



无论是Hadoop MapReduce，还是Spark或Flink，由于业务的需求，往往需要在计算过程中对数据进行Hash分区，再进行Join操作。这个过程中不同的框架会有不同的优化，但是归根到底，可以总结为两种方式。



其中一种方式就是直接将两个数据源的数据进行分区后，分别传输到下游任务中做Join。这就是一般的“Hash Join”。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220804131147352.png" alt="image-20220804131147352" style="zoom:50%;" />

另一种方式是，当其中一个数据源数据较少时，可以将该数据源的数据分发到所有节点上，与这些节点上的另一个数据源的数据进行Join。这种方式叫做“Broadcast Join”。它的好处是，数据源数据较多的一方不需要进行网络传输。

以上是Hadoop相关框架的实现。下面用一个具体的例子来看MPP架构对这一过程的思考。



在MPP架构中，数据往往会先指定分区Key，数据就按照分区Key分布在各个节点中。





现在假设有三张表，其中两张为大表，一张为小表：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220804131356492.png" alt="image-20220804131356492" style="zoom:50%;" />

很自然地，订单表会选择订单ID为做分区Key，产品表会选择产品ID作为分区Key，客户表会选择客户ID作为分区Key。给这些表中添加一些数据，并且执行一个查询语句：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220804131449777.png" alt="image-20220804131449777" style="zoom:50%;" />

首先，订单表要与客户表做Join，Join Key是客户ID。这种操作在Hadoop生态圈的分布式计算框架中，相当于对两个表做了Hash分区的操作。不过由于客户表已经按照客户ID提前做好了分区，所以这时只需要对订单表做重分区。在MPP架构中，会产生如下的结果：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220804131700926.png" alt="image-20220804131700926" style="zoom:50%;" />

此时，订单表整个表的数据会发生重分区，由此产生网络IO。这种情况相当于Hadoop架构中的“Hash Join”。





接着，需要让结果与产品表按照产品ID做Join。这时，因为之前产生的结果的分区Key不是产品ID，看起来又需要将整个数据进行重分区。不过，注意到产品表是个小表，所以此时只需要将该表广播到各个节点即可。结果如下：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220804131945961.png" alt="image-20220804131945961" style="zoom:50%;" />

在这个过程中，就只有小表的数据发生了网络IO。这就相当于Hadoop架构中的“Broadcast Join”。两者还有区别吗？前文在MPP架构的概念、历史以及技术细节上与Hadoop架构做了对比，了解到了两者一些极为相似的地方，而且在广义上讲，Hadoop就是MPP架构的一种实现。

然而前文也讲到，由于传播上的谬误，现在人们说到MPP架构，主要指的是分布式数据库，它处理的是结构化的数据，而Hadoop生态圈是由“大数据”这套概念发展而来，最初处理的都是非结构化的数据。以此为出发点，两者到底在发展过程中产生了多大的区别呢？





对比的维度有很多，比如很多人会说，MPP架构的平台封闭、拥有成熟的人才市场，而Hadoop架构平台开放、人才专业培训较少等。但这些并不是本质的区别。这里还是以技术指标作为维度来进行对比。