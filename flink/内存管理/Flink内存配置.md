Flink内存配置

**flink 分别提供了通用和细粒度的内存配置，来满足不同用户的需求。**

## 1.配置总内存

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210806155422846.png" alt="image-20210806155422846" style="zoom:50%;" />

Total Process Memory: 包括 flink 应用消耗的内存(Total Flink Memory) 和 JVM 消耗的内存

Total Flink Memory: 包括 JVM heap, managed memory 和 direct memory

The simplest way to setup memory in Flink is to configure either of the two following options:

最简单的方式是配置以下两个参数中的一个:

| **Component**        | **Option for TaskManager**                                   | **Option for JobManager**                                    |
| :------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Total Flink memory   | [`taskmanager.memory.flink.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#taskmanager-memory-flink-size) | [`jobmanager.memory.flink.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#jobmanager-memory-flink-size) |
| Total process memory | [`taskmanager.memory.process.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#taskmanager-memory-process-size) | [`jobmanager.memory.process.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#jobmanager-memory-process-size) |

The rest of the memory components will be adjusted automatically, based on default values or additionally configured options.

Configuring *total Flink memory* is better suited for [standalone deployments](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/resource-providers/standalone/overview/) where you want to declare how much memory is given to Flink itself. The *total Flink memory* splits up into *JVM Heap* and *Off-heap* memory.

If you configure *total process memory* you declare how much memory in total should be assigned to the Flink *JVM process*. For the containerized deployments it corresponds to the size of the requested container,

Another way to set up the memory is to configure the required internal components of the *total Flink memory* which are specific to the concrete Flink process. 

One of the three mentioned ways has to be used to configure Flink’s memory (except for local execution), or the Flink startup will fail. This means that one of the following option subsets, which do not have default values, have to be configured explicitly:

**Explicitly configuring both *total process memory* and *total Flink memory* is not recommended. It may lead to deployment failures due to potential memory configuration conflicts. Configuring other memory components also requires caution as it can produce further configuration conflicts.**

| **for TaskManager:**                                         | **for JobManager:**                                          |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`taskmanager.memory.flink.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#taskmanager-memory-flink-size) | [`jobmanager.memory.flink.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#jobmanager-memory-flink-size) |
| [`taskmanager.memory.process.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#taskmanager-memory-process-size) | [`jobmanager.memory.process.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#jobmanager-memory-process-size) |
| [`taskmanager.memory.task.heap.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#taskmanager-memory-task-heap-size) and [`taskmanager.memory.managed.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#taskmanager-memory-managed-size) | [`jobmanager.memory.heap.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#jobmanager-memory-heap-size) |

> 第一种配置方式是直接配置 total process memory或者 total flink memory。
>
> 其中: total process memory包括total flink memory和 JVM消耗的内存。
>
> ​		  total flink memory 包括 JVM heap 、managed memeory、direct memory
>
> 1. 当配置了以上两个参数中的一个后剩下的内存组件会根据默认值或者其他的参数自动配置完成。
>
> 2. 配置total Flink memory 更是适合独立部署模式，申明给Flink自身的内存大小。配置total process memory更适合容器化部署，表示给Flink JVM 进程分配多少内存，对应于容器化部署中请求的容器大小。
>
> 3. 另一种设置内存的方法是配置特定于具体Flink流程的*total Flink memory*所需的内部组件。
>
> 4. 对于以上的三种设置必须设置其中一种(除了本地执行模式)，否则flink将会启动失败。、
>
>    注意： 同时配置total process memory和total flink memory 是不被允许的，可以回带来部署失败。由于潜在的配置冲突。
>
>    配置其他的内存组件也需要注意这个问题。

## 2. Configure Heap and Managed Memory

As mentioned before in [total memory description](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_setup_tm/#configure-total-memory), another way to setup memory in Flink is to specify explicitly both [task heap](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_setup_tm/#task-operator-heap-memory) and [managed memory](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_setup_tm/#managed-memory). It gives more control over the available JVM Heap to Flink’s tasks and its [managed memory](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_setup_tm/#managed-memory).

The rest of the memory components will be adjusted automatically, based on default values or additionally configured options. [Here](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_setup_tm/#detailed-memory-model) are more details about the other memory components.

### Task (Operator) Heap Memory [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_setup_tm/#task-operator-heap-memory)

If you want to guarantee that a certain amount of JVM Heap is available for your user code, you can set the *task heap memory* explicitly ([`taskmanager.memory.task.heap.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#taskmanager-memory-task-heap-size)). It will be added to the JVM Heap size and will be dedicated to Flink’s operators running the user code.

### Managed Memory [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_setup_tm/#managed-memory)

*Managed memory* is managed by Flink and is allocated as native memory (off-heap). The following workloads use *managed memory*:

- Streaming jobs can use it for [RocksDB state backend](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/state_backends/#the-rocksdbstatebackend).
- Both streaming and batch jobs can use it for sorting, hash tables, caching of intermediate results.
- Both streaming and batch jobs can use it for executing [User Defined Functions in Python processes](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/python/table/udfs/python_udfs/).

The size of *managed memory* can be

- either configured explicitly via [`taskmanager.memory.managed.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#taskmanager-memory-managed-size)
- or computed as a fraction of *total Flink memory* via [`taskmanager.memory.managed.fraction`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#taskmanager-memory-managed-fraction).

*Size* will override *fraction*, if both are set. If neither *size* nor *fraction* is explicitly configured, the [default fraction](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#taskmanager-memory-managed-fraction) will be used.

> 第二种配置方式是明确配置 task heap和 managed memory。剩下的参数通过默认配置或其他参数自动配置。
>
> 1. task heap 或者叫做 operator heap 是 **执行用户代码 需要的JVM 堆内存**。可以通过taskmanager.memory.task.heap.size参数配置。
>
> 2. managed memory 叫做管理内存。它分配在本地内存(堆外内存)。
>
>    主要作用是
>
>    1. 存储流作业中状态的存储.rockdb
>    2. 存储流作业或者批作业中的  中间变量存储。
>    3. 存储python 进程中用户自定义函数
>
>    配置参数：
>
>    1. 可以通过taskmanager.memory.managed.size明确指定
>    2. 或者通过taskmanager.memory.managed.fraction *  total flink memory计算得出。

## 3.配置 Off-Heap Memory (direct or native)

用户申请的 off-heap 被算做 task off-heap memory，通过 taskmanager.memory.task.off-heap.size 配置。
`注意`：用户也可以调整 framework off-heap memory，即 flink 框架使用的堆外内存。这个是高级配置，最好确定需要时才进行调整。

flink 将 framework off-heap memory 和 task off-heap memory 纳入 JVM 的 direct memory 限制参数中：

```
-XX:MaxDirectMemorySize = Framework  Off-heap + Task Off-Heap + Network Memory
```

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210806162425780.png" alt="image-20210806162425780" style="zoom:50%;" />

