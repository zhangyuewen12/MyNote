## Allowed Lateness [#](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#allowed-lateness)

When working with *event-time* windowing, it can happen that elements arrive late, *i.e.* the watermark that Flink uses to keep track of the progress of event-time is already past the end timestamp of a window to which an element belongs. See [event time](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/event-time/generating_watermarks/) and especially [late elements](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/event-time/generating_watermarks/#late-elements) for a more thorough discussion of how Flink deals with event time.

By default, late elements are dropped when the watermark is past the end of the window. However, Flink allows to specify a maximum *allowed lateness* for window operators. Allowed lateness specifies by how much time elements can be late before they are dropped, and its default value is 0. Elements that arrive after the watermark has passed the end of the window but before it passes the end of the window plus the allowed lateness, are still added to the window. Depending on the trigger used, a late but not dropped element may cause the window to fire again. This is the case for the `EventTimeTrigger`.

In order to make this work, Flink keeps the state of windows until their allowed lateness expires. Once this happens, Flink removes the window and deletes its state, as also described in the [Window Lifecycle](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#window-lifecycle) section.

By default, the allowed lateness is set to `0`. That is, elements that arrive behind the watermark will be dropped.

You can specify an allowed lateness like this:

Java

```java
DataStream<T> input = ...;

input
    .keyBy(<key selector>)
    .window(<window assigner>)
    .allowedLateness(<time>)
    .<windowed transformation>(<window function>);
```

