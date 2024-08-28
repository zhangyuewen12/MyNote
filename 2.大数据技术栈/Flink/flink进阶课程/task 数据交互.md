# task之间的数据传输以及网络流控

### 编译阶段生成JobGraph

![image-20211124181347616](/Users/zyw/Library/Application Support/typora-user-images/image-20211124181347616.png)

### 运行阶段生成调度ExecutionGraph

![image-20211124181400951](/Users/zyw/Library/Application Support/typora-user-images/image-20211124181400951.png)

### task 数据之间的传输

![image-20211124181430903](/Users/zyw/Library/Application Support/typora-user-images/image-20211124181430903.png)

上图代表了一个简单的 map-reduce 类型的作业，有两个并行的任务。有两个 TaskManager，每个 TaskManager 都分别运行一个 map Task 和一个 reduce Task。我们重点观察 M1 和 R2 这两个 Task 之间的数据传输的发起过程。数据传输用粗箭头表示，消息用细箭头表示。首先，M1 产出了一个 ResultPartition(RP1)（箭头1）。当这个 RP 可以被消费是，会告知 JobManager（箭头2）。JobManager 会通知想要接收这个 RP 分区数据的接收者（tasks R1 and R2）当前分区数据已经准备好。如果接受放还没有被调度，这将会触发对应任务的部署（箭头 3a，3b）。接着，接受方会从 RP 中请求数据（箭头 4a，4b）。这将会初始化 Task 之间的数据传输（5a,5b）,数据传输可能是本地的(5a)，也可能是通过 TaskManager 的网络栈进行（5b）

对于一个 RP 什么时候告知 JobManager 当前已经出于可用状态，在这个过程中是有充分的自由度的：例如，如果在 RP1 在告知 JM 之前已经完整地产出了所有的数据（甚至可能写入了本地文件），那么相应的数据传输更类似于 Batch 的批交换；如果 RP1 在第一条记录产出时就告知 JM，那么就是 Streaming 流交换。

![image-20211124181454080](/Users/zyw/Library/Application Support/typora-user-images/image-20211124181454080.png)

1. `ResultPartition as RP` 和 `ResultSubpartition as RS`
    ExecutionGraph 还是 JobManager 中用于描述作业拓扑的一种逻辑上的数据结构，其中表示并行子任务的 `ExecutionVertex` 会被调度到 `TaskManager` 中执行，一个 Task 对应一个 ExecutionVertex。同 ExecutionVertex 的输出结果 IntermediateResultPartition 相对应的则是 `ResultPartition`。IntermediateResultPartition 可能会有多个 ExecutionEdge 作为消费者，那么在 Task 这里，ResultPartition 就会被拆分为多个 `ResultSubpartition`，下游每一个需要从当前 ResultPartition 消费数据的 Task 都会有一个专属的 `ResultSubpartition`。
    `ResultPartitionType`指定了`ResultPartition` 的不同属性，这些属性包括是否流水线模式、是否会产生反压以及是否限制使用的 Network buffer 的数量。`enum ResultPartitionType` 有三个枚举值：
    BLOCKING：非流水线模式，无反压，不限制使用的网络缓冲的数量
    PIPELINED：流水线模式，有反压，不限制使用的网络缓冲的数量
    PIPELINED_BOUNDED：流水线模式，有反压，限制使用的网络缓冲的数量

2. `InputGate as IG` 和 `InputChannel as IC`
    在 `Task` 中，`InputGate`是对输入的封装，`InputGate` 是和 `JobGraph` 中 `JobEdge` 一一对应的。也就是说，`InputGate` 实际上对应的是该 `Task` 依赖的上游算子（包含多个并行子任务），每个 `InputGate` 消费了一个或多个 `ResultPartition`。`InputGate` 由 `InputChannel` 构成，`InputChannel` 和`ExecutionEdge` 一一对应；也就是说， `InputChannel` 和 `ResultSubpartition` 一一相连，一个 `InputChannel`接收一个`ResultSubpartition` 的输出。根据读取的`ResultSubpartition` 的位置，`InputChannel` 有 `LocalInputChannel` 和 `RemoteInputChannel` 两种不同的实现。