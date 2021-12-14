# 问题一、请说一下Yarn的基本架构

YARN 主要由 ResourceManager、NodeManager、ApplicationMaster 和 Container 等组件

构成。

![image-20211016155119712](/Users/zyw/Library/Application Support/typora-user-images/image-20211016155119712.png)

```
ResourceManager（RM）
负责集群中所有资源的统一管理和分配，接收来自各个节点(NodeManger)的资源汇报信息，并根据收集到资源按照一定的策略分给应用程序。

NodeManager(NM)
NM是Yarn中每个节点的代理，管理Hadoop集群中单个计算节点，包括与RM保持通信、监督Container的生命周期、监控每个Container的使用情况(内存、cpu)、追踪节点健康状态，管理日志和不同应用程序用到的附属服务。

ApplicationMaster(AM)
负责一个Application生命周期内的所有工作。包括:
与RM调度器协商以获取资源；
将得到的资源进一步分配给内部任务；
与NM通信以启动、停止任务；
监控所有任务状态，并在任务运行失败时重新为任务申请资源以重启任务

Container
Container 是 YARN 中的资源抽象，它封装了某个节点上的多 维度资源，如内存、CPU、磁盘、 网络等。
当AM向RM申请资源时，RM为AM返回的资源用Container表示。

```



# 问题二、Yarn作业提交流程

![image-20211018093028828](/Users/zyw/Library/Application Support/typora-user-images/image-20211018093028828.png)

```
作业提交全流程
一、作业提交
第 1 步:Client 调用 job.waitForCompletion 方法，向整个集群提交 MapReduce 作业。 
第 2 步:Client 向 RM 申请一个作业 id。
第 3 步:RM 给 Client 返回该 job 资源的提交路径和作业 id。
第 4 步:Client 提交 jar 包、切片信息和配置文件到指定的资源提交路径。
第 5 步:Client 提交完资源后，向 RM 申请运行 MrAppMaster。
二、作业初始化
第 6 步:当 RM 收到 Client 的请求后，将该 job 添加到容量调度器中。
第 7 步:某一个空闲的 NM 领取到该 Job。
第 8 步:该 NM 创建 Container，并产生 MRAppmaster。
第 9 步:下载 Client 提交的资源到本地
三、任务分配
第 10 步:MrAppMaster 向 RM 申请运行多个 MapTask 任务资源。
第 11 步:RM 将运行 MapTask 任务分配给另外两个 NodeManager，另两个 NodeManager
分别领取任务并创建容器。
四、任务运行
第 12 步:MR 向两个接收到任务的 NodeManager 发送程序启动脚本，这两个 NodeManager 分别启动 MapTask，MapTask 对数据分区排序。
第 13 步:MRAppMaster 等待所有 MapTask 运行完毕后，向 RM 申请容器，运行 ReduceTask。 
第 14 步:ReduceTask 向 MapTask 获取相应分区的数据。
第 15 步:程序运行完毕后，MR 会向 RM 申请注销自己。
五、进度和状态更新
YARN 中的任务将其进度和状态(包括 counter)返回给应用管理器, 客户端每秒(通过 mapreduce.client.progressmonitor.pollinterval 设置)向应用管理器请求进度更新, 展示给用户。
六、作业完成
除了向应用管理器请求作业进度外, 客户端每 5 秒都会通过调用 waitForCompletion()来检查作业是否完成。时间间隔可以通过 mapreduce.client.completion.pollinterval 来设置。作业完成之后, 应用管理器和 Container 会清理工作状态。作业的信息会被作业历史服务器存储 以备之后用户核查。
```



# 问题三、Yarn调度器和调度算法

```
Hadoop 作业调度器主要有三种:FIFO、容量(Capacity Scheduler)和公平(Fair Scheduler)。Apache Hadoop3.1.3 默认的资源调度器是 Capacity Scheduler。


FIFO 调度器(First In First Out):单队列，根据提交作业的先后顺序，先来先服务。


Fair Scheduler
在Fair调度器中，我们不需要预先占用一定的系统资源，Fair调度器会为所有运行的job动态的调整系统资源。当第一个大job提交时，只有这一个job在运行，此时它获得了所有集群资源；当第二个小任务提交后，Fair调度器会分配一半资源给这个小任务，让这两个任务公平的共享集群资源。需要注意的是，在Fair调度器中，从第二个任务提交到获得资源会有一定的延迟，因为它需要等待第一个任务释放占用的Container。小任务执行完成之后也会释放自己占用的资源，大任务又获得了全部的系统资源。最终的效果就是Fair调度器即得到了高的资源利用率又能保证小任务及时完成.
```



## 容量调度器

![image-20211018093249739](/Users/zyw/Library/Application Support/typora-user-images/image-20211018093249739.png)

```
1、多队列:每个队列可配置一定的资源量，每个队列采用FIFO调度策略。 
2、容量保证:管理员可为每个队列设置资源最低保证和资源使用上限
3、灵活性:如果一个队列中的资源有剩余，可以暂时共享给那些需要资源的队列，而一旦该队列有新的应用 程序提交，则其他队列借调的资源会归还给该队列。
4、多租户:
支持多用户共享集群和多应用程序同时运行。 为了防止同一个用户的作业独占队列中的资源，该调度器会对同一用户提交的作业所占资源量进行限定。
```



### Fair Scheduler

在Fair调度器中，我们不需要预先占用一定的系统资源，Fair调度器会为所有运行的job动态的调整系统资源。当第一个大job提交时，只有这一个job在运行，此时它获得了所有集群资源；当第二个小任务提交后，Fair调度器会分配一半资源给这个小任务，让这两个任务公平的共享集群资源。需要注意的是，在Fair调度器中，从第二个任务提交到获得资源会有一定的延迟，因为它需要等待第一个任务释放占用的Container。小任务执行完成之后也会释放自己占用的资源，大任务又获得了全部的系统资源。最终的效果就是Fair调度器即得到了高的资源利用率又能保证小任务及时完成.
