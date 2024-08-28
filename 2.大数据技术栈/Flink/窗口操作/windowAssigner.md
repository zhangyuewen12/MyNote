> **Keyed Windows**
>
> ```
> stream
>        .keyBy(...)               <-  keyed versus non-keyed windows
>        .window(...)              <-  required: "assigner"
>       [.trigger(...)]            <-  optional: "trigger" (else default trigger)
>       [.evictor(...)]            <-  optional: "evictor" (else no evictor)
>       [.allowedLateness(...)]    <-  optional: "lateness" (else zero)
>       [.sideOutputLateData(...)] <-  optional: "output tag" (else no side output for late data)
>        .reduce/aggregate/apply()      <-  required: "function"
>       [.getSideOutput(...)]      <-  optional: "output tag"
> ```
>
> **Non-Keyed Windows**
>
> ```
> stream
>        .windowAll(...)           <-  required: "assigner"
>       [.trigger(...)]            <-  optional: "trigger" (else default trigger)
>       [.evictor(...)]            <-  optional: "evictor" (else no evictor)
>       [.allowedLateness(...)]    <-  optional: "lateness" (else zero)
>       [.sideOutputLateData(...)] <-  optional: "output tag" (else no side output for late data)
>        .reduce/aggregate/apply()      <-  required: "function"
>       [.getSideOutput(...)]      <-  optional: "output tag
> ```



## Window Assigners [#](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#window-assigners)

After specifying whether your stream is keyed or not, the next step is to define a *window assigner*. The window assigner defines how elements are assigned to windows. This is done by specifying the `WindowAssigner` of your choice in the `window(...)` (for *keyed* streams) or the `windowAll()` (for *non-keyed* streams) call.

> 第二步，就是制定窗口分配器，窗口分配器 决定元素 被分配到哪个或哪几个窗口中。

A `WindowAssigner` is responsible for assigning each incoming element to one or more windows. Flink comes with pre-defined window assigners for the most common use cases, namely *tumbling windows*, *sliding windows*, *session windows* and *global windows*. You can also implement a custom window assigner by extending the `WindowAssigner` class. All built-in window assigners (except the global windows) assign elements to windows based on time, which can either be processing time or event time. Please take a look at our section on [event time](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/concepts/time/) to learn about the difference between processing time and event time and how timestamps and watermarks are generated.

Time-based windows have a *start timestamp* (inclusive) and an *end timestamp* (exclusive) that together describe the size of the window. In code, Flink uses `TimeWindow` when working with time-based windows which has methods for querying the start- and end-timestamp and also an additional method `maxTimestamp()` that returns the largest allowed timestamp for a given windows.

In the following, we show how Flink’s pre-defined window assigners work and how they are used in a DataStream program. The following figures visualize the workings of each assigner. The purple circles represent elements of the stream, which are partitioned by some key (in this case *user 1*, *user 2* and *user 3*). The x-axis shows the progress of time.



> A *tumbling windows* assigner
>
> The *sliding windows* assigner 
>
> The *session windows* assigner 