### Flink 四层转化流程

Flink 有四层转换流程，第一层为 Program 到 StreamGraph；第二层为 StreamGraph 到 JobGraph；第三层为 JobGraph 到 ExecutionGraph；第四层为 ExecutionGraph 到物理执行计划。通过对 Program 的执行，能够生成一个 DAG 执行图，即逻辑执行图。如下：





第一部分将先讲解四层转化的流程，然后将以详细案例讲解四层的具体转化。

- 第一层 StreamGraph 从 Source 节点开始，每一次 transform 生成一个 StreamNode，两个 StreamNode 通过 StreamEdge 连接在一起,形成 StreamNode 和 StreamEdge 构成的DAG。
- 第二层 JobGraph，依旧从 Source 节点开始，然后去遍历寻找能够嵌到一起的 operator，如果能够嵌到一起则嵌到一起，不能嵌到一起的单独生成 jobVertex，通过 JobEdge 链接上下游 JobVertex，最终形成 JobVertex 层面的 DAG。
- JobVertex DAG 提交到任务以后，从 Source 节点开始排序,根据 JobVertex 生成ExecutionJobVertex，根据 jobVertex的IntermediateDataSet 构建IntermediateResult，然后 IntermediateResult 构建上下游的依赖关系，形成 ExecutionJobVertex 层面的 DAG 即 ExecutionGraph。
- 最后通过 ExecutionGraph 层到物理执行层