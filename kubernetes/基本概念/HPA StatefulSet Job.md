**HPA**

Horizontal pod autoscaling 仅仅适用于Deployment和ReplicaSet，根据POd的CPU利用率扩缩容

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210720150832688.png" alt="image-20210720150832688" style="zoom:50%;" />



**StatefulSet**

解决有状态服务的问题。

DaemonSet 确保全部（或者一些Node）上运行一个pod副本。当有Node加入集群时，会为他们新增一个pod。当有Node从集群中移除时，这些Pod也会被回收。删除daemonSet将会删除它所创建的所有pod

相当于：

为所有的集群时启动的节点都创建一个默认启动的程序，守护进程。

典型的用户有 日志收集进程，监控进程等。



**Job**

Job 负责批处理任务，即只执行一次的任务，**它保证批处理任务的一个或多个pod成功结束(保证每个pod必须成功至少一次)**。

Cron Job管理基于时间的job,即

在给定的时间点只运行一次

周期性地在给定时间点运行

