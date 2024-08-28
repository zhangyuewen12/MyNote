## Evictors [#](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#evictors)

Flink’s windowing model allows specifying an optional `Evictor` in addition to the `WindowAssigner` and the `Trigger`. This can be done using the `evictor(...)` method (shown in the beginning of this document). The evictor has the ability to remove elements from a window *after* the trigger fires and *before and/or after* the window function is applied. To do so, the `Evictor` interface has two methods:

> 驱逐器  
>
> 可以在窗口函数之前或者之后，清除数据	。

```
/**
 * Optionally evicts elements. Called before windowing function.
 *
 * @param elements The elements currently in the pane.
 * @param size The current number of elements in the pane.
 * @param window The {@link Window}
 * @param evictorContext The context for the Evictor
 */
void evictBefore(Iterable<TimestampedValue<T>> elements, int size, W window, EvictorContext evictorContext);

/**
 * Optionally evicts elements. Called after windowing function.
 *
 * @param elements The elements currently in the pane.
 * @param size The current number of elements in the pane.
 * @param window The {@link Window}
 * @param evictorContext The context for the Evictor
 */
void evictAfter(Iterable<TimestampedValue<T>> elements, int size, W window, EvictorContext evictorContext);
```

The `evictBefore()` contains the eviction logic to be applied before the window function, while the `evictAfter()` contains the one to be applied after the window function. Elements evicted before the application of the window function will not be processed by it.

Flink comes with three pre-implemented evictors. These are:

- `CountEvictor`: keeps up to a user-specified number of elements from the window and discards the remaining ones from the beginning of the window buffer.

  > 保留指定数量的元素，从窗口头部开始的丢弃元素

- `DeltaEvictor`: takes a `DeltaFunction` and a `threshold`, computes the delta between the last element in the window buffer and each of the remaining ones, and removes the ones with a delta greater or equal to the threshold.

  > 

- `TimeEvictor`: takes as argument an `interval` in milliseconds and for a given window, it finds the maximum timestamp `max_ts` among its elements and removes all the elements with timestamps smaller than `max_ts - interval`.

By default, all the pre-implemented evictors apply their logic before the window function.

> Specifying an evictor prevents any pre-aggregation, as all the elements of a window have to be passed to the evictor before applying the computation. This means windows with evictors will create significantly more state.

Flink provides no guarantees about the order of the elements within a window. This implies that although an evictor may remove elements from the beginning of the window, these are not necessarily the ones that arrive first or last.