
Spark编程模型（一)


本节主要内容

* Spark重要概念
* 弹性分布式数据集（RDD）基础

## Spark重要概念


（1）Spark运行模式

目前最为常用的Spark运行模式有： 
- local：本地线程方式运行，主要用于开发调试Spark应用程序 
- Standalone：利用Spark自带的资源管理与调度器运行Spark集群，采用Master/Slave结构，为解决单点故障，可以采用ZooKeeper实现高可靠（High Availability，HA) 
- Apache Mesos ：运行在著名的Mesos资源管理框架基础之上，该集群运行模式将资源管理交给Mesos，Spark只负责进行任务调度和计算 
- Hadoop YARN : 集群运行在Yarn资源管理器上，资源管理交给Yarn，Spark只负责进行任务调度和计算 
Spark运行模式中Hadoop YARN的集群运行方式最为常用，本课程中的第一节便是采用Hadoop YARN的方式进行Spark集群搭建。如此Spark便与Hadoop生态圈完美搭配，组成强大的集群，可谓无所不能。

2）Spark组件（Components）

一个完整的Spark应用程序，如前一节当中SparkWordCount程序，在提交集群运行时，它涉及到如下图所示的组件
![](http://img.blog.csdn.net/20150920083018462)

各Spark应用程序以相互独立的进程集合运行于集群之上，由SparkContext对象进行协调，SparkContext对象可以视为Spark应用程序的入口，被称为driver program，SparkContext可以与不同种类的集群资源管理器(Cluster Manager），例如Hadoop Yarn、Mesos等 进行通信，从而分配到程序运行所需的资源，获取到集群运行所需的资源后，SparkContext将得到集群中其它工作节点（Worker Node） 上对应的Executors （不同的Spark应用程序有不同的Executor，它们之间也是独立的进程，Executor为应用程序提供分布式计算及数据存储功能），之后SparkContext将应用程序代码分发到各Executors，最后将任务（Task）分配给executors执行。

> Application（Spark应用程序）	
> 运行于Spark上的用户程序，由集群上的一个driver program（包含SparkContext对象）和多个executor线程组成


> Application jar（Spark应用程序JAR包）  
> Jar包中包含了用户Spark应用程序，如果Jar包要提交到集群中运行，不需要将其它的Spark依赖包打包进行
>
>  Driver program	  
> 包含main方法的程序，负责创建SparkContext对象
> 
> Cluster manager	
> 集群资源管理器，例如Mesos，Hadoop Yarn
> 
> Deploy mode	
> 部署模式，用于区别driver program的运行方式:集群模式(cluter mode)，driver在集群内部启动；客户端模式（client mode），driver进程从集群外部启动
> 
> Worker node	
> 工作节点， 集群中可以运行Spark应用程序的节点
> 
> Executor	
> Worker node上的进程，该进程用于执行具体的Spark应用程序任务，负责任务间的数据维护（数据在内存中或磁盘上)。不同的Spark应用程序有不同的Executor
> 
> Task	
> 运行于Executor中的任务单元，Spark应用程序最终被划分为经过优化后的多个任务的集合（在下一节中将详细阐述）
> 
> Job	
> 由多个任务构建的并行计算任务，具体为Spark中的action操作，如collect,save等)
> 
> Stage	 
> 每个job将被拆分为更小的task集合，这些任务集合被称为stage，各stage相互独立（类似于MapReduce中的map stage和reduce stage），由于它由多个task集合构成，因此也称为TaskSet
> 

