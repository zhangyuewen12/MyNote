**Keyed Windows**

```
stream
       .keyBy(...)               <-  keyed versus non-keyed windows
       .window(...)              <-  required: "assigner"
      [.trigger(...)]            <-  optional: "trigger" (else default trigger)
      [.evictor(...)]            <-  optional: "evictor" (else no evictor)
      [.allowedLateness(...)]    <-  optional: "lateness" (else zero)
      [.sideOutputLateData(...)] <-  optional: "output tag" (else no side output for late data)
       .reduce/aggregate/apply()      <-  required: "function"
      [.getSideOutput(...)]      <-  optional: "output tag"
```



In a nutshell, a window is **created** as soon as the first element that should belong to this window arrives, and the window is **completely removed** when the time (event or processing time) passes its end timestamp plus the user-specified `allowed lateness` (see [Allowed Lateness](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#allowed-lateness)). Flink guarantees removal only for time-based windows and not for other types, *e.g.* global windows (see [Window Assigners](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#window-assigners)). For example, with an event-time-based windowing strategy that creates non-overlapping (or tumbling) windows every 5 minutes and has an allowed lateness of 1 min, Flink will create a new window for the interval between `12:00` and `12:05` when the first element with a timestamp that falls into this interval arrives, and it will remove it when the watermark passes the `12:06` timestamp.

In addition, each window will have a `Trigger` (see [Triggers](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#triggers)) and a function (`ProcessWindowFunction`, `ReduceFunction`, or `AggregateFunction`) (see [Window Functions](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#window-functions)) attached to it. The function will contain the computation to be applied to the contents of the window, while the `Trigger` specifies the conditions under which the window is considered ready for the function to be applied. A triggering policy might be something like “when the number of elements in the window is more than 4”, or “when the watermark passes the end of the window”. A trigger can also decide to purge a window’s contents any time between its creation and removal. Purging in this case only refers to the elements in the window, and *not* the window metadata. This means that new data can still be added to that window.

Apart from the above, you can specify an `Evictor` (see [Evictors](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#evictors)) which will be able to remove elements from the window after the trigger fires and before and/or after the function is applied.

In the following we go into more detail for each of the components above. We start with the required parts in the above snippet (see [Keyed vs Non-Keyed Windows](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#keyed-vs-non-keyed-windows), [Window Assigner](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#window-assigner), and [Window Function](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#window-function)) before moving to the optional ones.

