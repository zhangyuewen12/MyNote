# 问题：介绍一下Spark计算框架的结构

Apache Spark is a unified analytics engine for large-scale data processing. 

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230824171500855.png" alt="image-20230824171500855" style="zoom:50%;" />

Spark applications run as independent sets of processes on a cluster, coordinated by the `SparkContext` object in your main program (called the *driver program*).

Specifically, to run on a cluster, the SparkContext can connect to several types of *cluster managers* (either Spark’s own standalone cluster manager, Mesos, YARN or Kubernetes), which allocate resources across applications. Once connected, Spark acquires *executors* on nodes in the cluster, which are processes that run computations and store data for your application. Next, it sends your application code (defined by JAR or Python files passed to SparkContext) to the executors. Finally, SparkContext sends *tasks* to the executors to run.

There are several useful things to note about this architecture:

1. Each application gets its own executor processes, which stay up for the duration of the whole application and run tasks in multiple threads. This has the benefit of isolating applications from each other, on both the scheduling side (each driver schedules its own tasks) and executor side (tasks from different applications run in different JVMs). However, it also means that data cannot be shared across different Spark applications (instances of SparkContext) without writing it to an external storage system.
2. Spark is agnostic to the underlying cluster manager. As long as it can acquire executor processes, and these communicate with each other, it is relatively easy to run it even on a cluster manager that also supports other applications (e.g. Mesos/YARN/Kubernetes).
3. The driver program must listen for and accept incoming connections from its executors throughout its lifetime (e.g., see [spark.driver.port in the network config section](https://spark.apache.org/docs/latest/configuration.html#networking)). As such, the driver program must be network addressable from the worker nodes.
4. Because the driver schedules tasks on the cluster, it should be run close to the worker nodes, preferably on the same local area network. If you’d like to send requests to the cluster remotely, it’s better to open an RPC to the driver and have it submit operations from nearby than to run a driver far away from the worker nodes.





1. **Driver Program（驱动程序）**： Spark应用程序的入口点，由用户编写的主要代码部分。驱动程序定义了Spark应用程序的整体控制流程，包括创建SparkContext、定义作业、任务和数据流，以及调度任务的执行。
2. **SparkContext**： SparkContext是与集群的连接，作为驱动程序与集群之间的通信桥梁。它负责与集群资源管理器（如YARN、Apache Mesos）通信，协调任务的调度和执行。SparkContext还负责分配任务给执行器（Executor）并管理它们的生命周期。
3. **Cluster Manager（集群管理器）**： 集群管理器负责分配资源、监控任务的执行状态以及在集群中启动和停止执行器。常见的集群管理器包括Apache Hadoop YARN、Apache Mesos和Standalone Cluster Manager。
4. **Executor（执行器）**： 执行器是运行在集群节点上的工作单元。每个执行器负责执行分配给它的任务，并管理内存和计算资源。执行器接收驱动程序发送的任务代码和数据，并将计算结果返回给驱动程序。
5. **Task（任务）**： 任务是Spark中最小的执行单元。驱动程序将作业拆分成一系列任务，然后将任务分配给执行器执行。任务通常对应于数据的分区，可以在分布式环境中并行执行。