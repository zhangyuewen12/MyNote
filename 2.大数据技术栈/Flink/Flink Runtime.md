### Flink Runtime



![image-20210414114033554](/Users/zyw/Library/Application Support/typora-user-images/image-20210414114033554.png)



Flink 是可以运行在多种不同的环境中的。

例如，它可以通过单进程多线程的方式直接运行，从而提供调试的能力。它也可以运行在 Yarn 或者 K8S 这种资源管理系统上面，也可以在各种云环境中执行，针对不同的执行环境，Flink 提供了一套统一的分布式作业执行引擎，也就是 Flink Runtime 这层。Flink 在 Runtime 层之上提供了 DataStream 和 DataSet 两套 API，分别用来编写流作业与批作业，以及一组更高级的 API 来简化特定作业的编写。本文主要介绍 Flink Runtime 层的整体架构。



![image-20210414114216153](/Users/zyw/Library/Application Support/typora-user-images/image-20210414114216153.png)

Flink Runtime 层的主要架构如图 2 所示，它展示了一个 Flink 集群的基本结构。Flink Runtime 层的整个架构主要是在 FLIP-6 中实现的，整体来说，它采用了标准 master-slave 的结构，其中左侧白色圈中的部分即是 master，它负责管理整个集群中的资源和作业；而右侧的两个 TaskExecutor 则是 Slave，负责提供具体的资源并实际执行作业。



**其中，Master 部分又包含了三个组件，即 Dispatcher、ResourceManager 和 JobManager**。

**Dispatcher 负责接收用户提供的作业，并且负责为这个新提交的作业拉起一个新的 JobManager 组件**。

**ResourceManager 负责资源的管理，在整个 Flink 集群中只有一个 ResourceManager。**

**JobManager 负责管理作业的执行，在一个 Flink 集群中可能有多个作业同时执行，每个作业都有自己的 JobManager 组件。**

这三个组件都包含在 AppMaster 进程中。





当用户提交作业的时候，提交脚本会首先启动一个 Client进程负责作业的编译与提交。

**1.它首先将用户编写的代码编译为一个 JobGraph，在这个过程，它还会进行一些检查或优化等工作，例如判断哪些 Operator 可以 Chain 到同一个 Task 中。**

**2.然后，Client 将产生的 JobGraph 提交到集群中执行。**此时有两种情况:

一种是类似于 Standalone 这种 Session 模式，AM 会预先启动，此时 Client 直接与 Dispatcher 建立连接并提交作业即可。另一种是 Per-Job 模式，AM 不会预先启动，此时 Client 将首先向资源管理系统 （如Yarn、K8S）申请资源来启动 AM，然后再向 AM 中的 Dispatcher 提交作业。

**3.当作业到 Dispatcher 后，Dispatcher 会首先启动一个 JobManager 组件，然后 JobManager 会向 ResourceManager 申请资源来启动作业中具体的任务。**

Flink 中 TaskExecutor 的资源是通过 Slot 来描述的，一个 Slot 一般可以执行一个具体的 Task，但在一些情况下也可以执行多个相关联的 Task。

**4.ResourceManager 选择到空闲的 Slot 之后，就会通知相应的 TM “将该 Slot 分配分 JobManager XX ”，然后 TaskExecutor 进行相应的记录后，会向 JobManager 进行注册。JobManager 收到 TaskExecutor 注册上来的 Slot 后，就可以实际提交 Task 了。**

TaskExecutor 可能已经启动或者尚未启动。如果是前者，此时 ResourceManager 中已有记录了 TaskExecutor 注册的资源，可以直接选取空闲资源进行分配。否则，ResourceManager 也需要首先向外部资源管理系统申请资源来启动 TaskExecutor，然后等待 TaskExecutor 注册相应资源后再继续选择空闲资源进程分配。

**5.TaskExecutor 收到 JobManager 提交的 Task 之后，会启动一个新的线程来执行该 Task。Task 启动后就会开始进行预先指定的计算，并通过数据 Shuffle 模块互相交换数据。**



### 资源管理与作业调度

**实际上，作业调度可以看做是对资源和任务进行匹配的过程。**

 **1.Flink 中，资源是通过 Slot 来表示的，每个 Slot 可以用来执行不同的 Task。而在另一端，任务即 Job 中实际的 Task，它包含了待执行的用户逻辑。调度的主要目的就是为了给 Task 找到匹配的 Slot。**

逻辑上来说，每个 Slot 都应该有一个向量来描述它所能提供的各种资源的量，每个 Task 也需要相应的说明它所需要的各种资源的量。但是实际上在 1.9 之前，Flink 是不支持细粒度的资源描述的，而是统一的认为每个 Slot 提供的资源和 Task 需要的资源都是相同的。从 1.9 开始，Flink 开始增加对细粒度的资源匹配的支持的实现，但这部分功能目前仍在完善中。



作业调度的基础是首先提供对资源的管理，因此我们首先来看下 Flink 中资源管理的实现。Flink 中的资源是由 TaskExecutor 上的 Slot 来表示的。

![image-20210414115924350](/Users/zyw/Library/Application Support/typora-user-images/image-20210414115924350.png)

如图 4 所示，在 ResourceManager 中，有一个子组件叫做 SlotManager，它维护了当前集群中所有 TaskExecutor 上的 Slot 的信息与状态，如该 Slot 在哪个 TaskExecutor 中，该 Slot 当前是否空闲等。

1. 当 JobManger 来为特定 Task 申请资源的时候，根据当前是 Per-job 还是 Session 模式，ResourceManager 可能会去申请资源来启动新的 TaskExecutor。
2. 当 TaskExecutor 启动之后，它会通过服务发现找到当前活跃的 ResourceManager 并进行注册。在注册信息中，会包含该 TaskExecutor中所有 Slot 的信息。
3. ResourceManager 收到注册信息后，其中的 SlotManager 就会记录下相应的 Slot 信息。
4. 当 JobManager 为某个 Task 来申请资源时， SlotManager 就会从当前空闲的 Slot 中按一定规则选择一个空闲的 Slot 进行分配。当分配完成后，如第 2 节所述，RM 会首先向 TaskManager 发送 RPC 要求将选定的 Slot 分配给特定的 JobManager。
5. TaskManager 如果还没有执行过该 JobManager 的 Task 的话，它需要首先向相应的 JobManager 建立连接，然后发送提供 Slot 的 RPC 请求。
6. 在 JobManager 中，所有 Task 的请求会缓存到 SlotPool 中。当有 Slot 被提供之后，SlotPool 会从缓存的请求中选择相应的请求并结束相应的请求过程。
7. 当 Task 结束之后，无论是正常结束还是异常结束，都会通知 JobManager 相应的结束状态。
8. 然后在 TaskManager 端将 Slot 标记为**已占用但未执行任务的状态。**
9. JobManager 会首先将相应的 Slot 缓存到 SlotPool 中，但不会立即释放。这种方式避免了如果将 Slot 直接还给 ResourceManager，在任务异常结束之后需要重启时，需要立刻重新申请 Slot 的问题。通过延时释放，Failover 的 Task 可以尽快调度回原来的 TaskManager，从而加快 Failover 的速度。当 SlotPool 中缓存的 Slot 超过指定的时间仍未使用时，SlotPool 就会发起释放该 Slot 的过程。
10. 与申请 Slot 的过程对应，SlotPool 会首先通知 TaskManager 来释放该 Slot，然后 TaskExecutor 通知 ResourceManager 该 Slot 已经被释放，从而最终完成释放的逻辑。
11. 除了正常的通信逻辑外，在 ResourceManager 和 TaskExecutor 之间还存在定时的心跳消息来同步 Slot 的状态。在分布式系统中，消息的丢失、错乱不可避免，这些问题会在分布式系统的组件中引入不一致状态，如果没有定时消息，那么组件无法从这些不一致状态中恢复。此外，当组件之间长时间未收到对方的心跳时，就会认为对应的组件已经失效，并进入到 Failover 的流程。



![image-20210414120754120](/Users/zyw/Library/Application Support/typora-user-images/image-20210414120754120.png)

在 Slot 管理基础上，Flink 可以将 Task 调度到相应的 Slot 当中。如上文所述，Flink 尚未完全引入细粒度的资源匹配，**默认情况下，每个 Slot 可以分配给一个 Task。但是，这种方式在某些情况下会导致资源利用率不高。**如图 5 所示，假如 A、B、C 依次执行计算逻辑，那么给 A、B、C 分配分配单独的 Slot 就会导致资源利用率不高。为了解决这一问题，Flink 提供了 Share Slot 的机制。如图 5 所示，基于 Share Slot，**每个 Slot 中可以部署来自不同 JobVertex 的多个任务，但是不能部署来自同一个 JobVertex 的 Task**。如图5所示，每个 Slot 中最多可以部署同一个 A、B 或 C 的 Task，但是可以同时部署 A、B 和 C 的各一个 Task。当单个 Task 占用资源较少时，Share Slot 可以提高资源利用率。 此外，Share Slot 也提供了一种简单的保持负载均衡的方式。





基于上述 Slot 管理和分配的逻辑，JobManager 负责维护作业中 Task执行的状态。

1.如上文所述，Client 端会向 JobManager 提交一个 JobGraph，它代表了作业的逻辑结构。

2.JobManager 会根据 JobGraph 按并发展开，从而得到 JobManager 中关键的 ExecutionGraph。

ExecutionGraph 的结构下图所示，与 JobGraph 相比，ExecutionGraph 中对于每个 Task 与中间结果等均创建了对应的对象，从而可以维护这些实体的信息与状态。

![image-20210414120923741](/Users/zyw/Library/Application Support/typora-user-images/image-20210414120923741.png)



在一个 Flink Job 中是包含多个 Task 的，因此另一个关键的问题是在 Flink 中按什么顺序来调度 Task。如图 7 所示，目前 Flink 提供了两种基本的调度逻辑，即 Eager 调度与 Lazy From Source。Eager 调度如其名子所示，它会在作业启动时申请资源将所有的 Task 调度起来。这种调度算法主要用来调度可能没有终止的流作业。与之对应，Lazy From Source 则是从 Source 开始，按拓扑顺序来进行调度。简单来说，Lazy From Source 会先调度没有上游任务的 Source 任务，当这些任务执行完成时，它会将输出数据缓存到内存或者写入到磁盘中。然后，对于后续的任务，当它的前驱任务全部执行完成后，Flink 就会将这些任务调度起来。这些任务会从读取上游缓存的输出数据进行自己的计算。这一过程继续进行直到所有的任务完成计算。

![image-20210414121010939](/Users/zyw/Library/Application Support/typora-user-images/image-20210414121010939.png)

