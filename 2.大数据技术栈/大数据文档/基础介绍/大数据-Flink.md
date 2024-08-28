# Flink基础介绍

## 一、引言

### 1.1 什么是Flink？

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230802161113223.png" alt="image-20230802161113223" style="zoom:50%;" />

Flink是一种开源的流式处理和批处理计算框架，由Apache Software Foundation支持。它旨在实现高性能、高可用性的数据流处理和批处理应用程序。Flink的特点是将流处理和批处理的能力进行统一，使得开发人员可以无缝地在实时和离线数据上进行计算和处理。

具体来说，Flink具有以下特点：

1. **事件驱动模型：** Flink基于事件驱动模型，它可以实时处理来自不同数据源的事件流，并以有序的方式处理这些事件。

2. **精确一次处理语义：** Flink支持事件时间处理，能够处理事件产生的时间，而不仅仅是接收数据的时间，从而实现更准确的处理结果。

3. **状态管理：** Flink允许在流处理应用程序中维护状态，可以将状态信息保存在内存或外部存储中，以便进行容错和恢复。

4. **窗口处理：** Flink支持时间窗口和其他类型的窗口处理，允许按时间或其他条件对数据流进行切分和处理。

5. **高性能和低延迟：** Flink的设计目标是提供高性能和低延迟的数据处理能力，使得实时数据分析成为可能。

6. **灵活性和扩展性：** Flink支持丰富的API和编程模型，开发人员可以根据应用程序的需求进行定制和扩展。

7. **容错性：** Flink通过分布式快照技术保证应用程序状态的一致性，使得在节点故障时能够快速恢复和继续处理数据。

Flink被广泛应用于实时数据分析、流式ETL（Extract, Transform, Load）、作业调度、事件驱动型应用等场景。它在处理大规模数据和复杂计算任务方面表现出色，成为流处理和批处理的领先框架之一。

## 二、Flink的核心概念

### 2.1 流处理和批处理的统一

"批流统一"包含以下几个方面：

1. **相同的编程模型：** Flink提供了统一的API和编程模型，可以在处理流式数据和批处理数据时使用相同的代码逻辑，开发人员无需学习不同的框架和语法。

2. **统一的数据处理方式：** 在Flink中，批处理和流处理都是通过数据流（DataStream）来进行处理的。批处理数据被视为有限的、有序的数据流，而流式数据则是无限的、持续产生的数据流。这种统一的数据处理方式使得在处理不同类型的数据时更加灵活和高效。

3. **共享的状态管理：** 在流处理和批处理中，都可能需要维护状态信息，例如聚合操作的中间结果。Flink提供了一套统一的状态管理机制，可以在流处理和批处理中共享状态，并支持状态的快照和恢复，确保在任务失败时数据的一致性。

4. **统一的处理语义：** Flink引入了事件时间处理（Event Time）的概念，无论是批处理还是流处理，都能处理事件产生的时间，而不仅仅是接收数据的时间。这种精确的处理语义对于实现一致性和准确性非常重要。

### 2.2 事件时间和处理时间

在Flink中，事件时间（Event Time）和处理时间（Processing Time）是两种不同的时间概念，用于处理数据流中的事件和指定数据的时间语义。这两种时间概念对于实时数据处理具有不同的用途和含义。

1. **事件时间（Event Time）：**
   - 事件时间是数据产生的实际时间戳。它是由数据源或数据本身记录的时间，在数据流中可以理解为事件发生的时间。
   - 事件时间在流式数据中具有重要作用，尤其当数据具有乱序、延迟或时序不确定性时，使用事件时间可以确保数据处理的准确性和一致性。
   - Flink在处理事件时间时，会根据事件的时间戳对数据进行排序和分配到相应的时间窗口，从而实现精确的窗口计算。

2. **处理时间（Processing Time）：**
   - 处理时间是数据到达处理程序的时间，即处理程序处理数据的时间。它是Flink执行数据处理的机器本地时间，与数据的产生时间无关。
   - 处理时间是一种简单和高效的时间处理方式，因为不需要考虑事件的实际产生时间，只关心数据到达处理程序的顺序即可。
   - 在使用处理时间的场景中，Flink将数据立即处理，不考虑事件的乱序或延迟，因此在某些情况下可能会导致结果的不准确性。

在Flink中，可以根据应用程序的需求选择合适的时间语义。通常，事件时间适用于需要处理乱序事件或需要对数据进行基于事件时间的窗口计算的场景，而处理时间适用于对延迟敏感性不高、简单业务逻辑的应用程序。

为了使用事件时间，需要确保数据流中的事件包含合适的时间戳，并配置水位线（Watermark）来指示事件时间的进展。水位线是事件时间处理的重要概念，用于告知Flink事件时间的进展，以便触发窗口计算和处理延迟数据。

总结来说，事件时间和处理时间是Flink中处理数据流的两种不同时间概念，分别用于实现精确和简单的数据处理，开发人员可以根据业务需求选择合适的时间语义。

### 2.3 状态管理

#### 2.3.1 什么是状态？

在数据流处理中，许多操作只是简单地逐个查看单个事件（例如事件解析器），但有些操作需要跨多个事件记住信息（例如窗口操作符）。这些操作被称为有状态的。

一些有状态操作的示例包括：

1. 当应用程序搜索特定的事件模式时，状态将存储目前为止遇到的事件序列。
2. 当按分钟/小时/天对事件进行聚合时，状态保存待处理的聚合结果。
3. 当对数据流中的数据点进行机器学习模型训练时，状态保存当前版本的模型参数。
4. 当需要管理历史数据时，状态可以高效地访问过去发生的事件。

Flink需要了解状态以使用检查点和保存点来实现容错性。

对状态的了解还允许重新调整Flink应用程序，这意味着Flink会自动重新分配状态以适应并行实例。

可查询状态允许您在运行时从Flink外部访问状态。

在处理状态时，阅读Flink的状态后端也可能是有用的。Flink提供了不同的状态后端，用于指定状态的存储方式和位置。

#### 2.3.2 Keyed State 键控状态

键控状态是根据特定键（key）对数据进行划分的状态。每个key都有其独立的状态，并且可以被不同算子处理。键控状态在处理基于键的操作时非常有用，比如窗口操作、按键分组的聚合操作等。

Keyed state is maintained in what can be thought of as an embedded key/value store. The state is partitioned and distributed strictly together with the streams that are read by the stateful operators. Hence, access to the key/value state is only possible on *keyed streams*, i.e. after a keyed/partitioned data exchange, and is restricted to the values associated with the current event’s key. Aligning the keys of streams and state makes sure that all state updates are local operations, guaranteeing consistency without transaction overhead. This alignment also allows Flink to redistribute the state and adjust the stream partitioning transparently.

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230804164613383.png" alt="image-20230804164613383" style="zoom:50%;" />

Keyed State is further organized into so-called *Key Groups*. Key Groups are the atomic unit by which Flink can redistribute Keyed State; there are exactly as many Key Groups as the defined maximum parallelism. During execution each parallel instance of a keyed operator works with the keys for one or more Key Groups.

#### 2.3.3 State Persistence [#](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/concepts/stateful-stream-processing/#state-persistence)

Flink implements fault tolerance using a combination of **stream replay** and **checkpointing**. A checkpoint marks a specific point in each of the input streams along with the corresponding state for each of the operators. A streaming dataflow can be resumed from a checkpoint while maintaining consistency *(exactly-once processing semantics)* by restoring the state of the operators and replaying the records from the point of the checkpoint.

The checkpoint interval is a means of trading off the overhead of fault tolerance during execution with the recovery time (the number of records that need to be replayed).

The fault tolerance mechanism continuously draws snapshots of the distributed streaming data flow. For streaming applications with small state, these snapshots are very light-weight and can be drawn frequently without much impact on performance. The state of the streaming applications is stored at a configurable place, usually in a distributed file system.

In case of a program failure (due to machine-, network-, or software failure), Flink stops the distributed streaming dataflow. The system then restarts the operators and resets them to the latest successful checkpoint. The input streams are reset to the point of the state snapshot. Any records that are processed as part of the restarted parallel dataflow are guaranteed to not have affected the previously checkpointed state.

## 三、Flink架构

### 3.1 Flink运行架构

Flink 运行时由两种类型的进程组成：一个 *JobManager* 和一个或者多个 *TaskManager*。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230802162408776.png" alt="image-20230802162408776" style="zoom:50%;" />

*Client* 不是运行时和程序执行的一部分，而是用于准备数据流并将其发送给 JobManager。之后，客户端可以断开连接（*分离模式*），或保持连接来接收进程报告（*附加模式*）。客户端可以作为触发执行 Java/Scala 程序的一部分运行，也可以在命令行进程`./bin/flink run ...`中运行。

可以通过多种方式启动 JobManager 和 TaskManager：直接在机器上作为[standalone 集群](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/deployment/resource-providers/standalone/overview/)启动、在容器中启动、或者通过[YARN](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/deployment/resource-providers/yarn/)等资源框架管理并启动。TaskManager 连接到 JobManagers，宣布自己可用，并被分配工作。

### 3.1 JobManager和TaskManager

#### 3.1.1 JobManager

*JobManager* 具有许多与协调 Flink 应用程序的分布式执行有关的职责：它决定何时调度下一个 task（或一组 task）、对完成的 task 或执行失败做出反应、协调 checkpoint、并且协调从失败中恢复等等。这个进程由三个不同的组件组成：

- **ResourceManager**

  *ResourceManager* 负责 Flink 集群中的资源提供、回收、分配 - 它管理 **task slots**，这是 Flink 集群中资源调度的单位（请参考[TaskManagers](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/concepts/flink-architecture/#taskmanagers)）。Flink 为不同的环境和资源提供者（例如 YARN、Kubernetes 和 standalone 部署）实现了对应的 ResourceManager。在 standalone 设置中，ResourceManager 只能分配可用 TaskManager 的 slots，而不能自行启动新的 TaskManager。

- **Dispatcher**

  *Dispatcher* 提供了一个 REST 接口，用来提交 Flink 应用程序执行，并为每个提交的作业启动一个新的 JobMaster。它还运行 Flink WebUI 用来提供作业执行信息。

- **JobMaster**

  *JobMaster* 负责管理单个[JobGraph](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/concepts/glossary/#logical-graph)的执行。Flink 集群中可以同时运行多个作业，每个作业都有自己的 JobMaster。

始终至少有一个 JobManager。高可用（HA）设置中可能有多个 JobManager，其中一个始终是 *leader*，其他的则是 *standby*（请参考 [高可用（HA）](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/deployment/ha/overview/)）。

#### 3.1.2 TaskManagers

*TaskManager*（也称为 *worker*）执行作业流的 task，并且缓存和交换数据流。

必须始终至少有一个 TaskManager。在 TaskManager 中资源调度的最小单位是 task *slot*。TaskManager 中 task slot 的数量表示并发处理 task 的数量。请注意一个 task slot 中可以执行多个算子（请参考[Tasks 和算子链](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/concepts/flink-architecture/#tasks-and-operator-chains)）。

### 3.2 JobGraph和执行图

当用户提交一个Flink程序时，Flink会经历四个转换流程，从高层逻辑执行图（DAG）到最终的物理执行计划，具体过程如下：

1. **Program到StreamGraph：**
   - 在这一层，用户通过Flink提供的API编写Flink程序，包括DataStream API或DataSet API。这些API允许用户定义数据源、转换操作、窗口函数、算子等。
   - 用户编写的程序被转换为StreamGraph，StreamGraph是一个逻辑执行图，它表示Flink程序的逻辑结构和数据流的转换关系。

2. **StreamGraph到JobGraph：**
   - 在这一层，StreamGraph被转换为JobGraph。JobGraph是一个中间表示，它包含了StreamGraph中的所有算子（操作符）和数据流，但并不涉及具体的执行细节。
   - 在这个阶段，Flink对JobGraph进行优化，包括算子的链式优化、任务并行度的设置等，以提高作业执行效率。

3. **JobGraph到ExecutionGraph：**
   - 在这一层，JobGraph被转换为ExecutionGraph，它是一个更具体的执行计划，包含了任务的调度和执行细节。
   - ExecutionGraph将JobGraph中的算子划分为更细粒度的子任务（Subtask），并根据任务之间的数据依赖关系构建有向无环图（DAG）。
   - ExecutionGraph中的任务将会在TaskManager上执行，执行过程中会经过数据的输入、转换和输出等操作。

4. **ExecutionGraph到物理执行计划：**
   - 在这一层，ExecutionGraph被转换为最终的物理执行计划，即实际的作业执行计划。
   - 物理执行计划包含了具体的资源分配、任务的调度和并行执行策略，以实现作业的最优执行。
   - 最终的物理执行计划被提交给资源管理器，由资源管理器分配资源并在集群中启动任务执行。

总结来说，Flink的四层转换流程从高层的逻辑执行图到最终的物理执行计划，确保了作业在集群中高效地执行。这个过程中，Flink对作业进行了优化和转换，以保证作业的性能和可靠性。用户可以通过Flink的API编写程序，而Flink会将其转换为最终的物理执行计划，并在集群中执行。

#### 实例

以并行度为2（其中Source并行度为1）的 SocketTextStreamWordCount 为例，四层执行图的演变过程如下图所示：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230802163550332.png" alt="image-20230802163550332" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230802163612192.png" alt="image-20230802163612192" style="zoom:50%;" />

- StreamGraph：根据用户通过 Stream API 编写的代码生成的最初的图。
  1）StreamNode：用来代表 operator 的类，并具有所有相关的属性，如并发度、入边和出边等。
  2）StreamEdge：表示连接两个StreamNode的边。
- JobGraph：StreamGraph经过优化后生成了 JobGraph，提交给JobManager 的数据结构。
  - JobVertex：经过优化后符合条件的多个StreamNode可能会chain在一起生成一个JobVertex，即一个JobVertex包含一个或多个operator，JobVertex的输入是JobEdge，输出是IntermediateDataSet。
  - intermediateDataSet：表示JobVertex的输出，即经过operator处理产生的数据集。producer是JobVertex，consumer是JobEdge。
  - JobEdge：代表了job graph中的一条数据传输通道。source 是 IntermediateDataSet，target 是 JobVertex。即数据通过JobEdge由IntermediateDataSet传递给目标JobVertex。
- ExecutionGraph：JobManager 根据 JobGraph 生成ExecutionGraph。ExecutionGraph是JobGraph的并行化版本，是调度层最核心的数据结构。
  - ExecutionJobVertex：和JobGraph中的JobVertex一一对应。每一个ExecutionJobVertex都有和并发度一样多的 ExecutionVertex。
  - ExecutionVertex：表示ExecutionJobVertex的其中一个并发子任务，输入是ExecutionEdge，输出是IntermediateResultPartition。
  - IntermediateResult：和JobGraph中的IntermediateDataSet一一对应。一个IntermediateResult包含多个IntermediateResultPartition，其个数等于该operator的并发度
  - IntermediateResultPartition：表示ExecutionVertex的一个输出分区，producer是ExecutionVertex，consumer是若干个ExecutionEdge。
  - ExecutionEdge：表示ExecutionVertex的输入，source是IntermediateResultPartition，target是ExecutionVertex。source和target都只能是一个。
  - Execution：是执行一个 ExecutionVertex 的一次尝试。当发生故障或者数据需要重算的情况下 ExecutionVertex 可能会有多个 ExecutionAttemptID。一个 Execution 通过 ExecutionAttemptID 来唯一标识。JM和TM之间关于 task 的部署和 task status 的更新都是通过 ExecutionAttemptID 来确定消息接受者。
- 物理执行图：JobManager 根据 ExecutionGraph 对 Job 进行调度后，在各个TaskManager 上部署 Task 后形成的“图”，并不是一个具体的数据结构。
  - Task：Execution被调度后在分配的 TaskManager 中启动对应的 Task。Task 包裹了具有用户执行逻辑的 operator。
  - ResultPartition：代表由一个Task的生成的数据，和ExecutionGraph中的IntermediateResultPartition一一对应。
  - ResultSubpartition：是ResultPartition的一个子分区。每个ResultPartition包含多个ResultSubpartition，其数目要由下游消费 Task 数和 DistributionPattern 来决定。
  - InputGate：代表Task的输入封装，和JobGraph中JobEdge一一对应。每个InputGate消费了一个或多个的ResultPartition。
  - InputChannel：每个InputGate会包含一个以上的InputChannel，和ExecutionGraph中的ExecutionEdge一一对应，也和ResultSubpartition一对一地相连，即一个InputChannel接收一个ResultSubpartition的输出。

### 3.3 Flink的高可用性机制

Flink提供了多种高可用性机制，确保在发生故障或节点失效时，作业能够保持可靠运行。以下是Flink的高可用性机制：

1. **JobManager高可用：** 
   - Flink支持将多个JobManager配置成高可用模式。其中一个JobManager被选举为Leader，负责接收和处理作业提交请求、作业调度、任务分配等操作。
   - 其他JobManager作为Standby备份，与Leader保持通信，并持续地复制Leader的元数据。在Leader失效时，Standby中的一个将会成为新的Leader，保证作业管理的连续性。

2. **Checkpoint机制：**
   - Flink的Checkpoint是一种用于实现容错性的机制。它将作业的状态（包括算子状态和任务进度）定期保存到持久化存储中，称为Checkpoint状态。
   - 在作业执行过程中，Flink定期生成Checkpoint，并记录每个Checkpoint的元数据，以及生成Checkpoint的数据流的水位线。
   - 当作业发生故障时，Flink可以使用最近一次成功的Checkpoint来恢复作业状态，并从该状态处继续进行处理，从而实现容错性。

3. **保存点（Savepoint）：**
   - 保存点是一种手动触发的Checkpoint。它允许用户主动保存作业的状态，并生成一个保存点。
   - 保存点可以用于将作业恢复到某个特定的状态，也可以用于将作业迁移到不同的集群或版本，实现版本升级和迁移。

4. **ZooKeeper集群：**
   - Flink使用ZooKeeper作为分布式协调服务，在高可用模式下用于Leader选举和元数据的管理。
   - 在高可用模式下，JobManager和ResourceManager会与ZooKeeper通信，确保Leader的选举和元数据的一致性。

5. **TaskManager失败处理：**
   - 当TaskManager发生故障或失效时，Flink会重新分配该TaskManager上的任务到其他正常运行的TaskManager上，以保证任务的连续执行。
   - 此外，Flink还会将任务的状态和数据重新分配到新的TaskManager上，以保持容错性和数据一致性。

通过以上高可用性机制，Flink能够有效地处理节点故障、任务失败等问题，并保证作业的稳定和可靠运行。高可用性机制是Flink在分布式环境中保证数据处理可靠性的重要保障。

## 四、**Flink编程模型**

### 4.1 数据转换和算子

#### Map DataStream → DataStream

Takes one element and produces one element. A map function that doubles the values of the input stream:

```java
DataStream<Integer> dataStream = //...
dataStream.map(new MapFunction<Integer, Integer>() {
    @Override
    public Integer map(Integer value) throws Exception {
        return 2 * value;
    }
});
```

#### FlatMap DataStream → DataStream 

Takes one element and produces zero, one, or more elements. A flatmap function that splits sentences to words:

```java
dataStream.flatMap(new FlatMapFunction<String, String>() {
    @Override
    public void flatMap(String value, Collector<String> out)
        throws Exception {
        for(String word: value.split(" ")){
            out.collect(word);
        }
    }
});
```

#### Filter DataStream → DataStream 

Evaluates a boolean function for each element and retains those for which the function returns true. A filter that filters out zero values:

```java
dataStream.filter(new FilterFunction<Integer>() {
    @Override
    public boolean filter(Integer value) throws Exception {
        return value != 0;
    }
});
```

#### KeyBy DataStream → KeyedStream 

Logically partitions a stream into disjoint partitions. All records with the same key are assigned to the same partition. Internally, *keyBy()* is implemented with hash partitioning. There are different ways to [specify keys](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/dev/datastream/fault-tolerance/state/#keyed-datastream).

```java
dataStream.keyBy(value -> value.getSomeKey());
dataStream.keyBy(value -> value.f0);
```

> A type **cannot be a key if**:
>
> 1. it is a POJO type but does not override the `hashCode()` method and relies on the `Object.hashCode()` implementation.
> 2. it is an array of any type.

#### Reduce KeyedStream → DataStream 

A “rolling” reduce on a keyed data stream. Combines the current element with the last reduced value and emits the new value.

A reduce function that creates a stream of partial sums:

```java
keyedStream.reduce(new ReduceFunction<Integer>() {
    @Override
    public Integer reduce(Integer value1, Integer value2)
    throws Exception {
        return value1 + value2;
    }
});
```

#### Window KeyedStream → WindowedStream 

Windows can be defined on already partitioned KeyedStreams. Windows group the data in each key according to some characteristic (e.g., the data that arrived within the last 5 seconds). See [windows](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/dev/datastream/operators/windows/) for a complete description of windows.

```java
dataStream
  .keyBy(value -> value.f0)
  .window(TumblingEventTimeWindows.of(Time.seconds(5))); 
```

### 4.2 自定义函数和处理器

在 Apache Flink 中，自定义函数和处理器是您可以使用的重要概念，它们允许您根据特定需求编写自己的逻辑来处理数据流。以下是关于Flink中自定义函数和处理器的介绍：

**自定义函数（Custom Functions）**：

自定义函数是一种您可以编写的功能，用于对数据流或数据集中的元素进行特定的操作。Flink支持各种类型的自定义函数，包括以下几种：

1. **MapFunction**：用于对输入元素进行一对一的映射操作，生成一个新的元素。例如，您可以在`MapFunction`中实现一个元素的转换逻辑。

2. **FlatMapFunction**：用于将一个输入元素映射到零个、一个或多个输出元素。这在拆分、展开等情况下非常有用。

3. **FilterFunction**：用于根据特定条件过滤数据流中的元素。

4. **RichMapFunction**、**RichFlatMapFunction**、**RichFilterFunction**：这些函数在功能上与上述函数类似，但它们还允许您在函数中访问运行时上下文和状态，这对于状态管理和配置共享很有用。

5. **ProcessFunction**：这是一个高级自定义函数，允许您对数据流上的每个元素进行更细粒度的控制。您可以管理状态、定时器和侧输出等。

6. **AggregateFunction**：用于在窗口或键上执行聚合操作，例如求和、平均值等。

7. **KeyedProcessFunction**：类似于`ProcessFunction`，但是可以根据键（key）对数据流进行处理，允许您在状态管理和事件时间处理方面更精细地控制。

**自定义处理器（Custom Processors）**：

自定义处理器是在 Flink 中用于高级流处理逻辑的一种方式。最常见的方式是使用`ProcessFunction`或`KeyedProcessFunction`来编写自定义处理逻辑，这允许您在数据流上执行更复杂的操作，如状态管理、定时器和侧输出。自定义处理器通常用于以下情况：

1. **状态管理**：自定义处理器允许您在处理每个元素时维护状态。这在需要跨元素维护数据的场景中非常有用，如会话窗口或累积操作。

2. **定时器**：您可以设置定时器，在指定的时间点触发特定操作。这在需要基于时间的操作（如超时处理）时非常有用。

3. **侧输出**：自定义处理器允许您将数据发送到侧输出流，这对于根据特定条件将数据发送到不同的流中非常有用。

#### 4.2.1样例一：使用自定义MapFunction

当使用自定义 `MapFunction` 函数时，您可以通过该函数对输入的元素进行映射操作，然后输出转换后的元素。在 WordCount 样例中，我们将使用自定义的 `MapFunction` 函数来将文本分割成单词，并为每个单词赋予初始计数值 1。以下是一个使用自定义 `MapFunction` 函数的 WordCount 示例：

```
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class WordCountCustomMapFunction {

    public static void main(String[] args) throws Exception {
        // 设置执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // 输入数据流
        DataStream<String> textStream = env.fromElements(
                "Hello Flink",
                "Flink is powerful",
                "Streaming processing with Flink"
        );

        // 使用自定义 MapFunction 进行单词分割和计数
        DataStream<Tuple2<String, Integer>> wordCounts = textStream
                .map(new WordCountMapFunction());

        // 打印结果
        wordCounts.print();

        // 执行程序
        env.execute("WordCountCustomMapFunction");
    }

    // 自定义 MapFunction
    public static class WordCountMapFunction implements MapFunction<String, Tuple2<String, Integer>> {
        @Override
        public Tuple2<String, Integer> map(String value) throws Exception {
            // 将输入文本分割成单词
            String[] words = value.toLowerCase().split(" ");
            
            // 为每个单词赋予初始计数值 1
            Tuple2<String, Integer> tuple = new Tuple2<>();
            for (String word : words) {
                if (!word.isEmpty()) {
                    tuple.f0 = word;
                    tuple.f1 = 1;
                    return tuple;
                }
            }
            
            return null;
        }
    }
}

```

#### 4.2.2 样例二：使用自定义处理器

```java
import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.util.Collector;

public class AgeFilterSideOutputExample {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<User> userStream = env.fromElements(
                new User("Alice", 28),
                new User("Bob", 35),
                new User("Charlie", 22),
                new User("David", 42)
        );

        // 使用自定义处理器将年龄大于30的用户发送到侧输出流
        SingleOutputStreamOperator<User> mainOutput = userStream.process(new AgeFilterProcessFunction());
        DataStream<User> sideOutput = mainOutput.getSideOutput(new OutputTag<User>("side-output") {});

        mainOutput.print("Main Output");
        sideOutput.print("Side Output");

        env.execute("AgeFilterSideOutputExample");
    }

    public static class User {
        public String name;
        public int age;

        public User(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public String toString() {
            return "User{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }

    public static class AgeFilterProcessFunction extends ProcessFunction<User, User> {
        private final OutputTag<User> sideOutputTag = new OutputTag<User>("side-output") {};

        @Override
        public void processElement(User user, Context context, Collector<User> collector) throws Exception {
            if (user.age > 30) {
                // 将年龄大于30的用户发送到侧输出流
                context.output(sideOutputTag, user);
            } else {
                // 将其他用户发送到主输出流
                collector.collect(user);
            }
        }
    }
}

```

在上述示例中，我们首先定义了一个简单的 `User` 类来表示用户，包括姓名和年龄。然后，我们从这些用户数据创建一个数据流。我们使用了一个自定义的 `AgeFilterProcessFunction` 处理器来根据用户的年龄将数据分成主输出流和侧输出流。大于30岁的用户会被发送到侧输出流，其余用户会发送到主输出流。

最后，我们通过打印主输出流和侧输出流中的元素来验证结果

## 五、**Flink的数据源和数据接收**

### 5.1 读取文件数据

当使用 Apache Flink 进行数据处理时，可以通过以下样例代码来读取文件数据。在这个示例中，我们将演示如何使用 Flink 的 DataSet API 从文本文件中读取数据，并对读取的数据进行简单的处理。

首先，确保你已经正确配置了 Flink 环境和依赖项。接下来，你可以创建一个类来实现文件数据的读取和处理。以下是一个基本的示例代码：

```java
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.core.fs.Path;

public class FileReadExample {
    public static void main(String[] args) throws Exception {
        // 创建执行环境
        final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        // 定义文件路径
        String filePath = "path/to/your/input/file.txt";

        // 读取文件数据
        DataSet<String> textFile = env.readTextFile(filePath);

        // 对每行数据进行处理，例如拆分单词并计数
        DataSet<Tuple2<String, Integer>> wordCounts = textFile
            .flatMap((String line, Collector<Tuple2<String, Integer>> out) -> {
                // 拆分每行数据为单词
                String[] words = line.split("\\s+");
                for (String word : words) {
                    // 将每个单词输出为Tuple2(word, 1)
                    out.collect(new Tuple2<>(word, 1));
                }
            })
            .groupBy(0) // 按单词分组
            .sum(1);     // 对每个单词的计数值求和

        // 打印处理结果
        wordCounts.print();

        // 执行任务
        env.execute("File Read Example");
    }
}
```

在这个示例中，我们首先创建了 Flink 的执行环境（`ExecutionEnvironment`）。然后，我们指定要读取的文件路径，使用 `readTextFile` 方法从文件中读取数据。接下来，我们使用 `flatMap` 对每行数据进行处理，将每个单词映射为一个 Tuple2 对象，其中包含单词和计数值。然后，我们使用 `groupBy` 按单词进行分组，再使用 `sum` 求和计算每个单词的出现次数。

最后，我们通过调用 `print` 方法来打印处理结果，并通过调用 `env.execute` 来执行任务。

请注意，这只是一个简单的示例，实际上 Flink 支持更多复杂的操作和数据处理模式。根据你的实际需求，你可以进一步探索 Flink 的功能和API。确保将示例代码中的文件路径替换为你实际的文件路径。

### 5.2 连接外部系统

使用 Flink 连接 Kafka 并读取数据的示例代码如下所示。在这个示例中，我们将使用 Flink 的 DataStream API 来读取 Kafka 主题中的数据。

首先，请确保你已经正确配置了 Flink 和 Kafka 连接器的依赖项。然后，你可以创建一个类来实现从 Kafka 主题中读取数据并进行简单处理。以下是一个基本的示例代码：

```java
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;

import java.util.Properties;

public class KafkaReadExample {
    public static void main(String[] args) throws Exception {
        // 创建执行环境
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // Kafka 相关配置
        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "kafka-broker1:9092,kafka-broker2:9092");
        properties.setProperty("group.id", "flink-consumer-group");

        // 定义要消费的 Kafka 主题
        String topic = "your-kafka-topic";

        // 创建 Kafka 消费者连接器
        FlinkKafkaConsumer<String> kafkaConsumer = new FlinkKafkaConsumer<>(
            topic, new SimpleStringSchema(), properties);

        // 从 Kafka 主题读取数据
        DataStream<String> kafkaStream = env.addSource(kafkaConsumer);

        // 对数据进行处理，例如打印每条消息
        kafkaStream.print();

        // 执行任务
        env.execute("Kafka Read Example");
    }
}
```

在这个示例中，我们首先创建了 Flink 的流式执行环境（`StreamExecutionEnvironment`）。然后，我们设置了 Kafka 的相关配置，如 Kafka 服务器地址和消费者组 ID。接下来，我们指定要消费的 Kafka 主题。然后，我们创建了一个 `FlinkKafkaConsumer` 实例，指定要从 Kafka 主题中读取的数据类型以及相关的配置。

通过调用 `env.addSource(kafkaConsumer)`，我们将 Kafka 消费者连接器添加到执行环境中，从而创建了一个 Kafka 数据流。然后，我们可以对数据流进行各种处理，例如使用 `print` 方法打印每条消息。

最后，我们调用 `env.execute` 来执行任务。

请确保将示例代码中的 Kafka 服务器地址、消费者组 ID 和要消费的 Kafka 主题替换为你实际的配置。

### 5.3 自定义数据源

使用自定义数据源来读取外部数据是 Flink 中常见的操作之一。自定义数据源可以帮助你适应各种非常规数据输入，例如从数据库、API、文件系统等获取数据。以下是一个基本的示例，演示了如何创建一个自定义数据源来读取外部数据：

```java
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.Random;

public class CustomDataSourceExample {
    public static void main(String[] args) throws Exception {
        // 创建执行环境
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // 添加自定义数据源
        DataStream<String> customStream = env.addSource(new CustomDataSource());

        // 处理数据，例如打印每条消息
        customStream.print();

        // 执行任务
        env.execute("Custom Data Source Example");
    }

    // 自定义数据源实现
    public static class CustomDataSource implements SourceFunction<String> {
        private volatile boolean isRunning = true;

        @Override
        public void run(SourceContext<String> ctx) throws Exception {
            // 使用一个循环模拟数据源产生数据
            Random random = new Random();
            while (isRunning) {
                String data = "Data " + random.nextInt(100);
                ctx.collect(data);

                // 控制数据生成速度
                Thread.sleep(1000);
            }
        }

        @Override
        public void cancel() {
            isRunning = false;
        }
    }
}
```

在这个示例中，我们首先创建了 Flink 的流式执行环境（`StreamExecutionEnvironment`）。然后，我们创建了一个自定义数据源类 `CustomDataSource`，它实现了 `SourceFunction<String>` 接口。在 `run` 方法中，我们使用一个循环来模拟数据源不断产生数据。数据产生后，我们通过 `ctx.collect(data)` 将数据发送到 Flink 数据流中。

在主函数中，我们通过 `env.addSource(new CustomDataSource())` 将自定义数据源添加到执行环境中。然后，我们可以对数据流进行各种处理，例如使用 `print` 方法打印每条消息。

最后，我们通过调用 `env.execute` 来执行任务。

这个示例展示了如何创建一个简单的自定义数据源。实际中，你可以根据需要从各种数据源中读取数据，并根据自己的业务逻辑对数据进行处理。

## 六、Flink的窗口和时间处理

### 6.1 滚动窗口

A *tumbling windows* assigner assigns each element to a window of a specified *window size*. Tumbling windows have a fixed size and do not overlap. For example, if you specify a tumbling window with a size of 5 minutes, the current window will be evaluated and a new window will be started every five minutes as illustrated by the following figure.

![image-20230807161212756](/Users/zyw/Library/Application Support/typora-user-images/image-20230807161212756.png)

```java
DataStream<T> input = ...;

// tumbling event-time windows
input
    .keyBy(<key selector>)
    .window(TumblingEventTimeWindows.of(Time.seconds(5)))
    .<windowed transformation>(<window function>);

// tumbling processing-time windows
input
    .keyBy(<key selector>)
    .window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
    .<windowed transformation>(<window function>);

// daily tumbling event-time windows offset by -8 hours.
input
    .keyBy(<key selector>)
    .window(TumblingEventTimeWindows.of(Time.days(1), Time.hours(-8)))
    .<windowed transformation>(<window function>);
```

Time intervals can be specified by using one of `Time.milliseconds(x)`, `Time.seconds(x)`, `Time.minutes(x)`, and so on.

As shown in the last example, tumbling window assigners also take an optional `offset` parameter that can be used to change the alignment of windows. For example, without offsets hourly tumbling windows are aligned with epoch, that is you will get windows such as `1:00:00.000 - 1:59:59.999`, `2:00:00.000 - 2:59:59.999` and so on. If you want to change that you can give an offset. With an offset of 15 minutes you would, for example, get `1:15:00.000 - 2:14:59.999`, `2:15:00.000 - 3:14:59.999` etc. An important use case for offsets is to adjust windows to timezones other than UTC-0. For example, in China you would have to specify an offset of `Time.hours(-8)`.

### 6.2 滑动窗口

The *sliding windows* assigner assigns elements to windows of fixed length. Similar to a tumbling windows assigner, the size of the windows is configured by the *window size* parameter. An additional *window slide* parameter controls how frequently a sliding window is started. Hence, sliding windows can be overlapping if the slide is smaller than the window size. In this case elements are assigned to multiple windows.

For example, you could have windows of size 10 minutes that slides by 5 minutes. With this you get every 5 minutes a window that contains the events that arrived during the last 10 minutes as depicted by the following figure.

![image-20230807161255785](/Users/zyw/Library/Application Support/typora-user-images/image-20230807161255785.png)

```java
DataStream<T> input = ...;

// sliding event-time windows
input
    .keyBy(<key selector>)
    .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))
    .<windowed transformation>(<window function>);

// sliding processing-time windows
input
    .keyBy(<key selector>)
    .window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(5)))
    .<windowed transformation>(<window function>);

// sliding processing-time windows offset by -8 hours
input
    .keyBy(<key selector>)
    .window(SlidingProcessingTimeWindows.of(Time.hours(12), Time.hours(1), Time.hours(-8)))
    .<windowed transformation>(<window function>);
```

Time intervals can be specified by using one of `Time.milliseconds(x)`, `Time.seconds(x)`, `Time.minutes(x)`, and so on.

As shown in the last example, sliding window assigners also take an optional `offset` parameter that can be used to change the alignment of windows. For example, without offsets hourly windows sliding by 30 minutes are aligned with epoch, that is you will get windows such as `1:00:00.000 - 1:59:59.999`, `1:30:00.000 - 2:29:59.999` and so on. If you want to change that you can give an offset. With an offset of 15 minutes you would, for example, get `1:15:00.000 - 2:14:59.999`, `1:45:00.000 - 2:44:59.999` etc. An important use case for offsets is to adjust windows to timezones other than UTC-0. For example, in China you would have to specify an offset of `Time.hours(-8)`.

### 6.3 会话窗口

The *session windows* assigner groups elements by sessions of activity. Session windows do not overlap and do not have a fixed start and end time, in contrast to *tumbling windows* and *sliding windows*. Instead a session window closes when it does not receive elements for a certain period of time, *i.e.*, when a gap of inactivity occurred. A session window assigner can be configured with either a static *session gap* or with a *session gap extractor* function which defines how long the period of inactivity is. When this period expires, the current session closes and subsequent elements are assigned to a new session window.

![image-20230807161338777](/Users/zyw/Library/Application Support/typora-user-images/image-20230807161338777.png)

```java
DataStream<T> input = ...;

// event-time session windows with static gap
input
    .keyBy(<key selector>)
    .window(EventTimeSessionWindows.withGap(Time.minutes(10)))
    .<windowed transformation>(<window function>);
    
// event-time session windows with dynamic gap
input
    .keyBy(<key selector>)
    .window(EventTimeSessionWindows.withDynamicGap((element) -> {
        // determine and return session gap
    }))
    .<windowed transformation>(<window function>);

// processing-time session windows with static gap
input
    .keyBy(<key selector>)
    .window(ProcessingTimeSessionWindows.withGap(Time.minutes(10)))
    .<windowed transformation>(<window function>);
    
// processing-time session windows with dynamic gap
input
    .keyBy(<key selector>)
    .window(ProcessingTimeSessionWindows.withDynamicGap((element) -> {
        // determine and return session gap
    }))
    .<windowed transformation>(<window function>);
```

Static gaps can be specified by using one of `Time.milliseconds(x)`, `Time.seconds(x)`, `Time.minutes(x)`, and so on.

## 七、**Flink的状态管理**

### 7.1 状态后端(State Backend)

#### 7.1.1 The HashMapStateBackend

The *HashMapStateBackend* holds data internally as objects on the Java heap. Key/value state and window operators hold hash tables that store the values, triggers, etc.

The HashMapStateBackend is encouraged for:

- Jobs with large state, long windows, large key/value states.
- All high-availability setups.

It is also recommended to set [managed memory](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/memory/mem_setup_tm/#managed-memory) to zero. This will ensure that the maximum amount of memory is allocated for user code on the JVM.

Unlike EmbeddedRocksDBStateBackend, the HashMapStateBackend stores data as objects on the heap so that it is unsafe to reuse objects.

#### 7.1.2 The EmbeddedRocksDBStateBackend 

The EmbeddedRocksDBStateBackend holds in-flight data in a [RocksDB](http://rocksdb.org/) database that is (per default) stored in the TaskManager local data directories. Unlike storing java objects in `HashMapStateBackend`, data is stored as serialized byte arrays, which are mainly defined by the type serializer, resulting in key comparisons being byte-wise instead of using Java’s `hashCode()` and `equals()` methods.

The EmbeddedRocksDBStateBackend always performs asynchronous snapshots.

Limitations of the EmbeddedRocksDBStateBackend:

- As RocksDB’s JNI bridge API is based on byte[], the maximum supported size per key and per value is 2^31 bytes each. States that use merge operations in RocksDB (e.g. ListState) can silently accumulate value sizes > 2^31 bytes and will then fail on their next retrieval. This is currently a limitation of RocksDB JNI.

The EmbeddedRocksDBStateBackend is encouraged for:

- Jobs with very large state, long windows, large key/value states.
- All high-availability setups.

Note that the amount of state that you can keep is only limited by the amount of disk space available. This allows keeping very large state, compared to the HashMapStateBackend that keeps state in memory. This also means, however, that the maximum throughput that can be achieved will be lower with this state backend. All reads/writes from/to this backend have to go through de-/serialization to retrieve/store the state objects, which is also more expensive than always working with the on-heap representation as the heap-based backends are doing. It’s safe for EmbeddedRocksDBStateBackend to reuse objects due to the de-/serialization.

Check also recommendations about the [task executor memory configuration](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/memory/mem_tuning/#rocksdb-state-backend) for the EmbeddedRocksDBStateBackend.

EmbeddedRocksDBStateBackend is currently the only backend that offers incremental checkpoints (see [here](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/ops/state/large_state_tuning/)).

Certain RocksDB native metrics are available but disabled by default, you can find full documentation [here](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/config/#rocksdb-native-metrics)

The total memory amount of RocksDB instance(s) per slot can also be bounded, please refer to documentation [here](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/ops/state/large_state_tuning/#bounding-rocksdb-memory-usage) for details.

#### 7.1.3 Choose The Right State Backend

When deciding between `HashMapStateBackend` and `RocksDB`, it is a choice between performance and scalability. `HashMapStateBackend` is very fast as each state access and update operates on objects on the Java heap; however, state size is limited by available memory within the cluster. On the other hand, `RocksDB` can scale based on available disk space and is the only state backend to support incremental snapshots. However, each state access and update requires (de-)serialization and potentially reading from disk which leads to average performance that is an order of magnitude slower than the memory state backends.

### 7.2 CheckPoint

#### 7.2.1 Checkpoint Storage 

When checkpointing is enabled, managed state is persisted to ensure consistent recovery in case of failures. Where the state is persisted during checkpointing depends on the chosen **Checkpoint Storage**.

Available Checkpoint Storage Options 

Out of the box, Flink bundles these checkpoint storage types:

- *JobManagerCheckpointStorage*
- *FileSystemCheckpointStorage*

#### 7.2.2 The JobManagerCheckpointStorage

The *JobManagerCheckpointStorage* stores checkpoint snapshots in the JobManager’s heap.

It can be configured to fail the checkpoint if it goes over a certain size to avoid `OutOfMemoryError`’s on the JobManager. To set this feature, users can instantiate a `JobManagerCheckpointStorage` with the corresponding max size:

```java
new JobManagerCheckpointStorage(MAX_MEM_STATE_SIZE);
```

Limitations of the `JobManagerCheckpointStorage`:

- The size of each individual state is by default limited to 5 MB. This value can be increased in the constructor of the `JobManagerCheckpointStorage`.
- Irrespective of the configured maximal state size, the state cannot be larger than the Akka frame size (see [Configuration](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/config/)).
- The aggregate state must fit into the JobManager memory.

The JobManagerCheckpointStorage is encouraged for:

- Local development and debugging
- Jobs that use very little state, such as jobs that consist only of record-at-a-time functions (Map, FlatMap, Filter, …). The Kafka Consumer requires very little state.

#### 7.2.3 The FileSystemCheckpointStorage

The *FileSystemCheckpointStorage* is configured with a file system URL (type, address, path), such as “hdfs://namenode:40010/flink/checkpoints” or “file:///data/flink/checkpoints”.

Upon checkpointing, it writes state snapshots into files in the configured file system and directory. Minimal metadata is stored in the JobManager’s memory (or, in high-availability mode, in the metadata checkpoint).

If a checkpoint directory is specified, `FileSystemCheckpointStorage` will be used to persist checkpoint snapshots.

The `FileSystemCheckpointStorage` is encouraged for:

- All high-availability setups.

It is also recommended to set [managed memory](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/memory/mem_setup_tm/#managed-memory) to zero. This will ensure that the maximum amount of memory is allocated for user code on the JVM.

### 7.3 SavePoint

What is a Savepoint?

A Savepoint is a consistent image of the execution state of a streaming job, created via Flink’s [checkpointing mechanism](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/learn-flink/fault_tolerance/). You can use Savepoints to stop-and-resume, fork, or update your Flink jobs. Savepoints consist of two parts: a directory with (typically large) binary files on stable storage (e.g. HDFS, S3, …) and a (relatively small) meta data file. The files on stable storage represent the net data of the job’s execution state image. The meta data file of a Savepoint contains (primarily) pointers to all files on stable storage that are part of the Savepoint, in form of relative paths.

## 八、**Flink的WaterMark**

### 8.1 WaterMark介绍

Apache Flink 中的 Watermark 是一种时间概念，用于处理事件时间（Event Time）数据流中的延迟和乱序问题。在事件时间处理中，数据可能以不同的时间顺序到达，甚至可能会有一些延迟。为了在处理这些情况时保持一致性，Flink 引入了 Watermark 机制。

Watermark 可以被看作是事件时间的进度指示器，它告诉 Flink 在哪个事件时间点之前的数据已经全部到达，从而帮助 Flink 控制窗口操作和处理迟到的事件。

以下是 Watermark 的一些关键概念和作用：

1. **事件时间（Event Time）**：每条数据都有一个与之相关的事件时间戳，表示数据发生的实际时间。与之相对的是处理时间（Processing Time）和摄取时间（Ingestion Time）。

2. **Watermark**：Watermark 是一种特殊的数据，它带有一个时间戳，表示在该时间点之前的事件时间数据已经全部到达。Watermark 是用来衡量事件时间进展的重要指标，也可以视为“延迟指示器”。

3. **乱序数据和延迟**：在事件时间处理中，数据可能以不同的顺序到达，也可能出现延迟。Watermark 可以帮助系统在处理乱序数据和迟到数据时进行适当的处理。

4. **窗口操作**：Flink 中的窗口操作（如滚动窗口、滑动窗口等）通常需要在确定窗口的结束时间前等待一段时间，以便处理迟到的数据。Watermark 可以帮助确定合适的窗口关闭时间，以平衡实时性和数据准确性。

5. **迟到事件处理**：通过设置窗口的允许迟到时间，Flink 可以容忍在 Watermark 到达后仍然到达的事件。迟到的事件可以根据其事件时间被分配到之前的窗口或特殊的迟到窗口进行处理。

使用 Watermark 机制，Flink 可以在事件时间处理中有效地处理乱序数据和延迟，从而提供更准确的计算结果。要在 Flink 中使用 Watermark，通常需要在数据源中插入正确的 Watermark，或者通过 `assignTimestampsAndWatermarks` 函数为数据流分配时间戳和 Watermark。

以下是一个简单的示例，演示了如何在 Flink 中使用 Watermark 机制：

```java
DataStream<MyEvent> events = ...; // 获取事件数据流
DataStream<MyEvent> withTimestampsAndWatermarks = events
    .assignTimestampsAndWatermarks(new MyWatermarkGenerator());

DataStream<MyEvent> windowedStream = withTimestampsAndWatermarks
    .keyBy(event -> event.getKey())
    .window(TumblingEventTimeWindows.of(Time.minutes(10)))
    .reduce((event1, event2) -> event1.combine(event2));
```

在这个示例中，`assignTimestampsAndWatermarks` 函数会分配事件的时间戳和 Watermark。然后，我们将数据流按事件的某个键分组，使用窗口操作来对数据进行处理。

总而言之，Watermark 机制是 Flink 在处理事件时间数据中的一个关键组成部分，可以有效解决乱序和延迟问题，从而提供准确的计算结果。

### 8.2 样例

以下是一个示例代码，演示如何在 Flink 中从 Kafka 中读取用户购买行为数据，并为数据流设置 Watermark。假设你已经在项目中添加了 Flink 和 Kafka 连接器的依赖。

```java
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.TimeCharacteristic;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.streaming.api.windowing.time.Time;

import java.util.Properties;

public class KafkaWatermarkExample {
    public static void main(String[] args) throws Exception {
        // 创建执行环境
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        
        // 设置事件时间为处理时间
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        // Kafka 相关配置
        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "kafka-broker1:9092,kafka-broker2:9092");
        properties.setProperty("group.id", "flink-consumer-group");

        // 定义要消费的 Kafka 主题
        String topic = "user-purchase-topic";

        // 创建 Kafka 消费者连接器
        FlinkKafkaConsumer<String> kafkaConsumer = new FlinkKafkaConsumer<>(
            topic, new SimpleStringSchema(), properties);

        // 从 Kafka 主题读取数据
        DataStream<String> kafkaStream = env.addSource(kafkaConsumer);

        // 解析用户购买行为数据并设置 Watermark
        DataStream<UserPurchaseEvent> purchaseStream = kafkaStream
            .map(line -> {
                String[] fields = line.split(",");
                return new UserPurchaseEvent(fields[0], fields[1], Long.parseLong(fields[2]));
            })
            .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<UserPurchaseEvent>(Time.seconds(10)) {
                @Override
                public long extractTimestamp(UserPurchaseEvent event) {
                    return event.getTimestamp();
                }
            });

        // 在这里可以继续对购买行为数据进行处理，例如窗口操作等

        // 执行任务
        env.execute("Kafka Watermark Example");
    }

    // 用户购买事件类
    public static class UserPurchaseEvent {
        private String userId;
        private String product;
        private long timestamp;

        public UserPurchaseEvent(String userId, String product, long timestamp) {
            this.userId = userId;
            this.product = product;
            this.timestamp = timestamp;
        }

        public String getUserId() {
            return userId;
        }

        public String getProduct() {
            return product;
        }

        public long getTimestamp() {
            return timestamp;
        }
    }
}
```

在这个示例中，我们使用 `BoundedOutOfOrdernessTimestampExtractor` 来为购买行为数据分配时间戳和 Watermark。通过 `assignTimestampsAndWatermarks` 方法，我们将时间戳和 Watermark 添加到数据流中。

在上面的样例中，使用了 `BoundedOutOfOrdernessTimestampExtractor` 来为事件分配时间戳和 Watermark，并设置了一个允许的最大延迟时间。在这个示例中，允许的最大延迟时间是 `Time.seconds(10)`，即10秒。这意味着在事件时间顺序上，Flink 将在当前事件时间减去10秒的时间点之前的数据到达之前，保持 Watermark 的进度。

请注意，实际中的购买行为数据解析可能会根据数据格式和业务逻辑进行调整。同时，根据实际需求，你可以在购买行为数据流上执行窗口操作等进一步的数据处理。

最后，通过调用 `env.execute` 来执行任务。确保将示例代码中的 Kafka 服务器地址、消费者组 ID、主题和数据解析逻辑根据实际情况进行替换。

除了使用 `BoundedOutOfOrdernessTimestampExtractor` 这种方式外，还有其他方式可以为数据流分配 Watermark。以下是一些常见的方式：

1. **AscendingTimestampExtractor**：这个提取器适用于按照事件时间递增顺序到达的数据流，不需要考虑乱序。它会简单地将事件时间作为时间戳，并使用时间戳本身作为 Watermark。

   ```java
   DataStream<MyEvent> events = ...;
   DataStream<MyEvent> withTimestampsAndWatermarks = events.assignTimestampsAndWatermarks(
       new AscendingTimestampExtractor<MyEvent>() {
           @Override
           public long extractAscendingTimestamp(MyEvent event) {
               return event.getTimestamp();
           }
       }
   );
   ```

2. **Periodic Watermark Generator**：你可以实现自己的定期 Watermark 生成器，以一定的时间间隔生成 Watermark。这种方法适用于没有固定延迟的数据流。

   ```java
   DataStream<MyEvent> events = ...;
   DataStream<MyEvent> withWatermarks = events.assignTimestampsAndWatermarks(
       new MyPeriodicWatermarkGenerator()
   );
   ```

3. **Punctuated Watermark Generator**：你可以实现自己的断点式 Watermark 生成器，根据某些条件来生成 Watermark。这种方法适用于特定事件触发的数据流。

   ```java
   DataStream<MyEvent> events = ...;
   DataStream<MyEvent> withWatermarks = events.assignTimestampsAndWatermarks(
       new MyPunctuatedWatermarkGenerator()
   );
   ```

无论哪种方式，Watermark 都有助于在事件时间处理中处理乱序和延迟数据，以确保准确性和一致性。你可以根据数据流的特点选择适合的 Watermark 生成方式。

## 九、**Flink的部署和运行**

### 9.1 Flink集群的部署

#### 9.1.1 部署概述

下图显示了每个Flink集群的构建块。始终有一个客户端在运行。它接收Flink应用程序的代码，将其转换成JobGraph并提交给JobManager。

JobManager将工作分发到TaskManagers，实际的操作符（例如源、转换和接收器）在TaskManagers上运行。

在部署Flink时，通常对每个构建块都有多个选项可用。我们已经将它们列在图下面的表格中。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230807112937147.png" alt="image-20230807112937147" style="zoom:50%;" />

| Component                                | Purpose                                                      | Implementations                                              |
| :--------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Flink Client                             | Compiles batch or streaming applications into a dataflow graph, which it then submits to the JobManager. | [Command Line Interface](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/cli/)[REST Endpoint](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/ops/rest_api/)[SQL Client](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/dev/table/sqlclient/)[Python REPL](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/repls/python_shell/) |
| JobManager                               | JobManager is the name of the central work coordination component of Flink. It has implementations for different resource providers, which differ on high-availability, resource allocation behavior and supported job submission modes. JobManager [modes for job submissions](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/overview/#deployment-modes):**Application Mode**: runs the cluster exclusively for one application. The job's main method (or client) gets executed on the JobManager. Calling `execute`/`executeAsync` multiple times in an application is supported.**Per-Job Mode**: runs the cluster exclusively for one job. The job's main method (or client) runs only prior to the cluster creation.**Session Mode**: one JobManager instance manages multiple jobs sharing the same cluster of TaskManagers | [Standalone](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/resource-providers/standalone/) (this is the barebone mode that requires just JVMs to be launched. Deployment with [Docker, Docker Swarm / Compose](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/resource-providers/standalone/docker/), [non-native Kubernetes](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/resource-providers/standalone/kubernetes/) and other models is possible through manual setup in this mode)[Kubernetes](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/resource-providers/native_kubernetes/)[YARN](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/resource-providers/yarn/) |
| TaskManager                              | TaskManagers are the services actually performing the work of a Flink job. |                                                              |
| High Availability Service Provider       | Flink's JobManager can be run in high availability mode which allows Flink to recover from JobManager faults. In order to failover faster, multiple standby JobManagers can be started to act as backups. | [Zookeeper](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/ha/zookeeper_ha/)[Kubernetes HA](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/ha/kubernetes_ha/) |
| File Storage and Persistency             | For checkpointing (recovery mechanism for streaming jobs) Flink relies on external file storage systems | See [FileSystems](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/filesystems/overview/) page. |
| Resource Provider                        | Flink can be deployed through different Resource Provider Frameworks, such as Kubernetes or YARN. | See [JobManager](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/overview/#jmimpls) implementations above. |
| Metrics Storage                          | Flink components report internal metrics and Flink jobs can report additional, job specific metrics as well. | See [Metrics Reporter](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/deployment/metric_reporters/) page. |
| Application-level data sources and sinks | While application-level data sources and sinks are not technically part of the deployment of Flink cluster components, they should be considered when planning a new Flink production deployment. Colocating frequently used data with Flink can have significant performance benefits | For example:Apache KafkaAmazon S3ElasticsearchApache CassandraSee [Connectors](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/connectors/datastream/overview/) page. |

#### 9.1.2 部署方式

Flink可以以以下三种方式之一执行应用程序：

- 应用程序模式（Application Mode）
- 会话模式（Session Mode）
- 单作业模式（已弃用）

上述模式在以下方面有所区别：

- 集群生命周期和资源隔离保证
- 应用程序的`main()`方法是在客户端还是在集群上执行。

![image-20230807113323547](/Users/zyw/Library/Application Support/typora-user-images/image-20230807113323547.png)

##### Application Mode

在所有其他模式下，应用程序的`main()`方法是在客户端端执行的。这个过程包括在本地下载应用程序的依赖项，在执行`main()`方法以提取Flink运行时能够理解的应用程序表示（即`JobGraph`），然后将依赖项和`JobGraph`传输到集群。这使得客户端成为一个重量级的资源消耗者，因为它可能需要大量的网络带宽来下载依赖项并将二进制文件传输到集群，以及CPU周期来执行`main()`方法。当客户端在多个用户之间共享时，这个问题可能更为突出。

在此基础上，**应用程序模式（Application Mode）**为每个提交的应用程序创建一个集群，但这次应用程序的`main()`方法由**JobManager**执行。每个应用程序创建一个集群可以看作是创建一个会话集群，仅在特定应用程序的作业之间共享，并在应用程序完成后关闭。通过这种架构，应用程序模式提供了与**单作业模式（Per-Job Mode）**相同的资源隔离和负载均衡保证，但是以整个应用程序的粒度进行。

应用程序模式的前提是用户的JAR包已经位于所有需要访问它的Flink组件（*JobManager*、*TaskManager*）的类路径（`usrlib`文件夹）上。换句话说，您的应用程序随Flink分发捆绑在一起。这样一来，应用程序模式可以通过不通过RPC将用户的JAR包分发到Flink组件，从而加快部署/恢复过程，使其不像其他部署模式那样需要这样做。

> The application mode assumes that the user jars are bundled with the Flink distribution.
>
> Executing the `main()` method on the cluster may have other implications for your code, such as any paths you register in your environment using the `registerCachedFile()` must be accessible by the JobManager of your application.

与**单作业模式（已弃用）**相比，**应用程序模式**允许提交由多个作业组成的应用程序。作业执行的顺序不受部署模式影响，而受启动作业时所使用的调用方式影响。使用`execute()`方法（阻塞方式），会建立一个顺序，这将导致"下一个"作业的执行被推迟，直到"当前"作业完成。而使用`executeAsync()`方法（非阻塞方式），将导致"下一个"作业在"当前"作业完成之前启动。

> The Application Mode allows for multi-`execute()` applications but High-Availability is not supported in these cases. High-Availability in Application Mode is only supported for single-`execute()` applications.
>
> Additionally, when any of multiple running jobs in Application Mode (submitted for example using `executeAsync()`) gets cancelled, all jobs will be stopped and the JobManager will shut down. Regular job completions (by the sources shutting down) are supported.

##### Session Mode

会话模式（Session Mode）假设已经运行了一个集群，并利用该集群的资源来执行任何提交的应用程序。在同一个（会话）集群中执行的应用程序使用相同的资源，并因此竞争相同的资源。这样做的好处是，您不需要为每个提交的作业启动一个完整的集群，从而避免了资源开销。但是，如果其中一个作业表现不佳或导致某个TaskManager崩溃，那么所有运行在该TaskManager上的作业都将受到故障的影响。除了对导致故障的作业产生负面影响之外，这还意味着潜在的大规模恢复过程，所有正在重新启动的作业都会并发访问文件系统，使其对其他服务不可用。此外，单个集群运行多个作业意味着JobManager负载更重，因为它负责对集群中所有作业进行记录和管理。

##### Per-Job Mode (deprecated) 

> Per-job mode is only supported by YARN and has been deprecated in Flink 1.15. It will be dropped in [FLINK-26000](https://issues.apache.org/jira/browse/FLINK-26000). Please consider application mode to launch a dedicated cluster per-job on YARN.

通过为每个作业创建独立的集群，单作业模式确保了作业之间的资源隔离，防止一个作业的问题影响到其他作业。当作业结束后，集群会通过为每个作业创建独立的集群，单作业模式确保了作业之间的资源隔离，防止一个作业的问题影响到其他作业。当作业结束后，集群会被销毁，从而释放资源并保持整洁。这样做使得作业之间不会相互干扰，提高了整个系统的稳定性和可靠性。同时，由于每个作业都有自己的JobManager，负载也得到了平衡，减少了单一JobManager的压力。

总之，单作业模式为每个作业提供了独立的集群资源，确保了良好的资源隔离和负载均衡，是一种在性能和可靠性方面都较优的部署方式。

##### 总结

### 9.2 Flink作业提交和监控

在会话模式中，一个集群可以运行多个作业，这些作业共享相同的资源，可以利用资源的共享来减少资源开销。但是，如果某个作业表现不佳或者造成集群故障，可能会影响到共享资源的其他作业。

而在应用程序模式中，每个应用程序有自己独立的集群，这意味着资源隔离更好，每个应用程序的作业只影响其自身的资源。但这样也会导致为每个应用程序启动一个集群，造成资源的重复使用，可能导致资源浪费。

因此，在选择部署模式时需要权衡资源隔离和资源开销之间的权衡。如果资源隔离是更重要的考虑因素，可以选择应用程序模式。如果资源开销更为关键，可以选择会话模式。

## 十、Flink与其他流处理框架的对比

- Flink vs. Spark Streaming

  Flink和Spark Streaming是两种流处理框架，它们在实时数据处理和批处理方面有许多共同点，但也存在一些重要的区别。以下是Flink和Spark Streaming的主要区别：

  1. **处理模式：**
     - Flink：Flink是一个真正的流处理引擎，它支持事件时间处理和处理时间处理，可以对无限数据流进行实时处理。Flink的流处理和批处理能力统一在一个引擎中，使得可以实现更高的灵活性和准确性。
     - Spark Streaming：Spark Streaming是基于微批处理的流处理框架，将数据流切分成短小的微批数据，并以一定时间间隔进行处理。虽然称为流处理，但实际上是批处理模式，处理速度相对较慢，不适合对低延迟的实时数据进行处理。

  2. **事件处理精度：**
     - Flink：Flink支持事件时间处理，可以处理数据流中事件的实际产生时间，从而实现更准确的处理结果。Flink使用水位线（Watermark）来追踪事件时间的进展，确保窗口的正确触发和处理。
     - Spark Streaming：Spark Streaming只支持处理时间处理，只关注数据到达处理程序的时间，不能处理数据流中事件的实际产生时间，因此可能导致一定程度上的延迟。

  3. **内存管理和执行引擎：**
     - Flink：Flink采用了内存管理技术，将数据流处理的状态保存在堆上，通过异步快照实现容错，支持更高效的数据处理。Flink执行引擎采用了基于事件驱动的处理模式，对于窗口计算和状态管理有更好的支持。
     - Spark Streaming：Spark Streaming使用批处理引擎来处理微批数据，数据流被切分成微批数据后，每个微批数据都是独立的Spark批处理作业，状态保存在磁盘上，容错性相对较差。

  4. **性能和延迟：**
     - Flink：由于其真正的流处理模式和事件时间处理，Flink可以实现更低的延迟和更高的吞吐量，特别适用于对延迟敏感的实时数据处理场景。
     - Spark Streaming：由于采用微批处理模式，Spark Streaming的延迟相对较高，不适合对低延迟的实时数据进行处理，通常用于对批量数据进行近实时处理。

  综上所述，Flink和Spark Streaming在实时数据处理方面有着不同的特点和适用场景。Flink适合对低延迟、高准确性的实时数据进行处理，而Spark Streaming适用于对大规模数据的近实时处理。选择哪种框架取决于具体的业务需求和数据处理场景。

## 十二、**结论**

### 12.1Flink的优势和局限性

**Flink的优势：**

1. **真正的流处理：** Flink是一个真正的流处理引擎，支持事件时间处理和处理时间处理，能够对无限数据流进行实时处理，实现低延迟和高吞吐量。

2. **事件驱动模型：** Flink采用事件驱动模型，具有更好的窗口计算和状态管理能力，支持更复杂的数据流处理逻辑。

3. **灵活性和一致性：** Flink的批流统一特性使得开发人员可以使用相同的API进行批处理和流处理，实现更灵活的数据处理需求。同时，Flink保证了数据处理的一致性和准确性。

4. **精确的事件时间处理：** Flink支持事件时间处理，并提供水位线（Watermark）机制，确保准确的窗口计算和数据处理。

5. **高可用性和容错性：** Flink支持JobManager的高可用模式和Checkpoint机制，能够有效处理故障，保证作业的稳定执行。

6. **内存管理和执行性能：** Flink采用内存管理技术，能够高效地处理大规模数据，同时执行引擎的事件驱动模型有助于提高执行性能。

**Flink的局限性：**

1. **学习曲线：** Flink相比一些其他流处理框架来说，学习曲线可能较陡，特别是对于新手来说，掌握Flink的API和核心概念可能需要一些时间。

2. **生态系统：** 相对于一些成熟的流处理框架，Flink的生态系统可能相对较小，虽然社区在不断发展，但与其他框架相比，可能有一些功能和工具的支持较少。

3. **资源消耗：** Flink在一些特定场景下可能需要较大的资源消耗，特别是在处理窗口计算和状态管理时，可能需要更多的内存和计算资源。

4. **批处理性能：** 尽管Flink支持批处理和流处理统一，但在一些纯批处理的场景下，其性能可能不如一些专为批处理优化的框架。

尽管Flink在很多方面都表现出色，但在选择框架时仍应根据具体业务需求和场景来进行评估。对于需要低延迟、高准确性的实时数据处理场景，Flink是一个非常优秀的选择。