# Flink 中几种 Time 详解

Processing Time、Event Time 和 Ingestion Time。

- **Processing time:** Processing time refers to the system time of the machine that is executing the respective operation.
- When a streaming program runs on processing time, all time-based operations (like time windows) will use the system clock of the machines that run the respective operator. An hourly processing time window will include all records that arrived at a specific operator between the times when the system clock indicated the full hour. For example, if an application begins running at 9:15am, the first hourly processing time window will include events processed between 9:15am and 10:00am, the next window will include events processed between 10:00am and 11:00am, and so on.
- Processing time is the simplest notion of time and requires no coordination between streams and machines. It provides the best performance and the lowest latency. However, in distributed and asynchronous environments processing time does not provide determinism, because it is susceptible to the speed at which records arrive in the system (for example from the message queue), to the speed at which the records flow between operators inside the system, and to outages (scheduled, or otherwise).
- processing time 与运行每个操作的机器的系统时间有关。基于时间的操作(如时间窗口)将使用当时机器的系统时间，
- 但是，在分布式和异步的环境下，Processing Time 不能提供确定性，因为它容易受到事件到达系统的速度（例如从消息队列）、事件在系统内操作流动的速度以及中断的影响。





- **Event time:** Event time is the time that each individual event occurred on its producing device. This time is typically embedded within the records before they enter Flink, and that *event timestamp* can be extracted from each record. In event time, the progress of time depends on the data, not on any wall clocks. Event time programs must specify how to generate *Event Time Watermarks*, which is the mechanism that signals progress in event time. This watermarking mechanism is described in a later section, [below](https://ci.apache.org/projects/flink/flink-docs-release-1.12/concepts/timely-stream-processing.html#event-time-and-watermarks).
- In a perfect world, event time processing would yield completely consistent and deterministic results, regardless of when events arrive, or their ordering. However, unless the events are known to arrive in-order (by timestamp), event time processing incurs some latency while waiting for out-of-order events. As it is only possible to wait for a finite period of time, this places a limit on how deterministic event time applications can be.
- Assuming all of the data has arrived, event time operations will behave as expected, and produce correct and consistent results even when working with out-of-order or late events, or when reprocessing historic data. For example, an hourly event time window will contain all records that carry an event timestamp that falls into that hour, regardless of the order in which they arrive, or when they are processed. (See the section on [late events](https://ci.apache.org/projects/flink/flink-docs-release-1.12/concepts/timely-stream-processing.html#late-elements) for more information.)
- Note that sometimes when event time programs are processing live data in real-time, they will use some *processing time* operations in order to guarantee that they are progressing in a timely fashion.
- Event Time 是事件发生的时间，一般就是数据本身携带的时间。这个时间通常是在事件到达 Flink 之前就确定的，并且可以从每个事件中获取到事件时间戳。在 Event Time 中，时间取决于数据，而跟其他没什么关系。Event Time 程序必须指定如何生成 Event Time 水印，这是表示 Event Time 进度的机制。
- 完美的说，无论事件什么时候到达或者其怎么排序，最后处理 Event Time 将产生完全一致和确定的结果。但是，除非事件按照已知顺序（按照事件的时间）到达，否则处理 Event Time 时将会因为要等待一些无序事件而产生一些延迟。由于只能等待一段有限的时间，因此就难以保证处理 Event Time 将产生完全一致和确定的结果。
- 假设所有数据都已到达， Event Time 操作将按照预期运行，即使在处理无序事件、延迟事件、重新处理历史数据时也会产生正确且一致的结果。 例如，每小时事件时间窗口将包含带有落入该小时的事件时间戳的所有记录，无论它们到达的顺序如何。