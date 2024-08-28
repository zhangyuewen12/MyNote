### Watermarks in Parallel Streams [#](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/concepts/time/#watermarks-in-parallel-streams)

Watermarks are generated at, or directly after, source functions. Each parallel subtask of a source function usually generates its watermarks independently. These watermarks define the event time at that particular parallel source.

As the watermarks flow through the streaming program, they advance the event time at the operators where they arrive. Whenever an operator advances its event time, it generates a new watermark downstream for its successor operators.

Some operators consume multiple input streams; a union, for example, or operators following a *keyBy(…)* or *partition(…)* function. Such an operator’s current event time is the minimum of its input streams' event times. As its input streams update their event times, so does the operator.

> 上流有多个输入流式，该算子的事件时间是多个上游流中最小的事件时间的值。

The figure below shows an example of events and watermarks flowing through parallel streams, and operators tracking event time.



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20211117163044166.png" alt="image-20211117163044166" style="zoom:50%;" />