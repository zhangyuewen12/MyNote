# Set up Flink’s Process Memory [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_setup/#set-up-flinks-process-memory)

Apache Flink provides efficient workloads on top of the JVM by tightly controlling the memory usage of its various components. While the community strives to offer sensible defaults to all configurations, the full breadth of applications that users deploy on Flink means this isn’t always possible. To provide the most production value to our users, Flink allows both high level and fine-grained tuning of memory allocation within clusters.

The further described memory configuration is applicable starting with the release version *1.10* for TaskManager and *1.11* for JobManager processes. If you upgrade Flink from earlier versions, check the [migration guide](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_migration/) because many changes were introduced with the *1.10* and *1.11* releases.

>ApacheFlink通过严格控制其各个组件的内存使用，在JVM之上提供高效的工作负载。虽然社区努力为所有配置提供合理的默认设置，但用户在Flink上部署的应用程序的范围很广，这意味着这并不总是可能的。为了向用户提供最大接近生产的参数，Flink允许对集群内的内存分配进行高级别和细粒度的调整。

## Configure Total Memory [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_setup/#configure-total-memory)

The *total process memory* of Flink JVM processes consists of memory consumed by the Flink application (*total Flink memory*) and by the JVM to run the process. The *total Flink memory* consumption includes usage of *JVM Heap* and *Off-heap* (*Direct* or *Native*) memory.

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210805161104441.png" alt="image-20210805161104441" style="zoom:50%;" />

The simplest way to setup memory in Flink is to configure either of the two following options:

| **Component**        | **Option for TaskManager**                                   | **Option for JobManager**                                    |
| :------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Total Flink memory   | [`taskmanager.memory.flink.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#taskmanager-memory-flink-size) | [`jobmanager.memory.flink.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#jobmanager-memory-flink-size) |
| Total process memory | [`taskmanager.memory.process.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#taskmanager-memory-process-size) | [`jobmanager.memory.process.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#jobmanager-memory-process-size) |

>Flink JVM的总进程内存由两个部分组成。1. Flink应用程序(total flink memory) 2. JVM运行进程需要的内存
>
>total flink memory 包含 1. JVM heap 2.Off-heap(Direct or navive)