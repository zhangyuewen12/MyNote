## Triggers [#](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#triggers)

A `Trigger` determines when a window (as formed by the *window assigner*) is ready to be processed by the *window function*. Each `WindowAssigner` comes with a default `Trigger`. If the default trigger does not fit your needs, you can specify a custom trigger using `trigger(...)`.

The trigger interface has five methods that allow a `Trigger` to react to different events:

- The `onElement()` method is called for each element that is added to a window.
- The `onEventTime()` method is called when a registered event-time timer fires.
- The `onProcessingTime()` method is called when a registered processing-time timer fires.
- The `onMerge()` method is relevant for stateful triggers and merges the states of two triggers when their corresponding windows merge, *e.g.* when using session windows.
- Finally the `clear()` method performs any action needed upon removal of the corresponding window.

Two things to notice about the above methods are:

1. The first three decide how to act on their invocation event by returning a `TriggerResult`. The action can be one of the following:

- `CONTINUE`: do nothing,
- `FIRE`: trigger the computation,
- `PURGE`: clear the elements in the window, and
- `FIRE_AND_PURGE`: trigger the computation and clear the elements in the window afterwards.

2. Any of these methods can be used to register processing- or event-time timers for future actions.

### Fire and Purge [#](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#fire-and-purge)

Once a trigger determines that a window is ready for processing, it fires, *i.e.*, it returns `FIRE` or `FIRE_AND_PURGE`. This is the signal for the window operator to emit the result of the current window. Given a window with a `ProcessWindowFunction` all elements are passed to the `ProcessWindowFunction` (possibly after passing them to an evictor). Windows with `ReduceFunction`, or `AggregateFunction` simply emit their eagerly aggregated result.

When a trigger fires, it can either `FIRE` or `FIRE_AND_PURGE`. While `FIRE` keeps the contents of the window, `FIRE_AND_PURGE` removes its content. By default, the pre-implemented triggers simply `FIRE` without purging the window state.

> Purging will simply remove the contents of the window and will leave any potential meta-information about the window and any trigger state intact.

### Default Triggers of WindowAssigners [#](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#default-triggers-of-windowassigners)

The default `Trigger` of a `WindowAssigner` is appropriate for many use cases. For example, all the event-time window assigners have an `EventTimeTrigger` as default trigger. This trigger simply fires once the watermark passes the end of a window.

The default trigger of the `GlobalWindow` is the `NeverTrigger` which does never fire. Consequently, you always have to define a custom trigger when using a `GlobalWindow`.

> By specifying a trigger using `trigger()` you are overwriting the default trigger of a `WindowAssigner`. For example, if you specify a `CountTrigger` for `TumblingEventTimeWindows` you will no longer get window firings based on the progress of time but only by count. Right now, you have to write your own custom trigger if you want to react based on both time and count.

### Built-in and Custom Triggers [#](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#built-in-and-custom-triggers)

Flink comes with a few built-in triggers.

- The (already mentioned) `EventTimeTrigger` fires based on the progress of event-time as measured by watermarks.
- The `ProcessingTimeTrigger` fires based on processing time.
- The `CountTrigger` fires once the number of elements in a window exceeds the given limit.
- The `PurgingTrigger` takes as argument another trigger and transforms it into a purging one.

If you need to implement a custom trigger, you should check out the abstract [Trigger ](https://github.com/apache/flink/blob/release-1.14//flink-streaming-java/src/main/java/org/apache/flink/streaming/api/windowing/triggers/Trigger.java)class. Please note that the API is still evolving and might change in future versions of Flink.