## Task生命周期

During the execution of the ExecutionGraph, each parallel task goes through multiple stages, from *created* to *finished* or *failed*. 

The diagram below illustrates the states and possible transitions between them.

 A task may be executed multiple times (for example in the course of failure recovery). For that reason, the execution of an ExecutionVertex is tracked in an [Execution](https://github.com/apache/flink/blob/master//flink-runtime/src/main/java/org/apache/flink/runtime/executiongraph/Execution.java). 

Each ExecutionVertex has a current Execution, and prior Executions.

![image-20210310100730144](/Users/zyw/Library/Application Support/typora-user-images/image-20210310100730144.png)

