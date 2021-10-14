### Spark的基本运行流程

spark的执行流程，简单来说

1. 先建立DAG的逻辑处理流程
2. 根据逻辑处理流程生成物理执行计划
3. 物理执行计划包含具体的计算任务，
4. 最后把task分配到多台机器上执行。



1.application启动之后, 会在本地启动一个Driver进程 用于控制整个流程,(假设我们使用的Standalone模式)

2 首先需要初始化的是SparkContext, SparkContext 要构建出DAGScheduler,TaskScheduler

3 在初始化TastScheduler的时候,它会去连接master,并向master 注册Application ,master 收到信息之后,会调用自己的资源调度算法,在spark集群的work上,启动Executor,并进行资源的分配,       最后将Executor 注册到TaskScheduler, 到这准备工作基本完成了

4 现在可以进行我们编写的的业务了, 一般情况下通过sc.textFile("file")  去加载数据源( 这个过程类似于mr的inputformat加载数据的过程), 去将数据转化为RDD,  

5 DagScheduer  先按照action将程序划分为一至多个job(每一个job对应一个Dag), 之后对DagScheduer按照是否进行shuffer,将job划分为多个Stage  每个Stage过程都是taskset , dag  将taskset交给taskScheduler,由Work中的Executor去执行,  至于tast交给那个executor去执行, 由算法来决定



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20211011212050062.png" alt="image-20211011212050062" style="zoom:50%;" />