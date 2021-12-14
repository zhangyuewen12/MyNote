## Window Functions [#](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#window-functions)

After defining the window assigner, we need to specify the computation that we want to perform on each of these windows. This is the responsibility of the *window function*, which is used to process the elements of each (possibly keyed) window once the system determines that a window is ready for processing (see [triggers](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#triggers) for how Flink determines when a window is ready).

The window function can be one of `ReduceFunction`, `AggregateFunction`, or `ProcessWindowFunction`. The first two can be executed more efficiently (see [State Size](https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/datastream/operators/windows/#useful-state-size-considerations) section) because Flink can incrementally aggregate the elements for each window as they arrive. A `ProcessWindowFunction` gets an `Iterable` for all the elements contained in a window and additional meta information about the window to which the elements belong.

A windowed transformation with a `ProcessWindowFunction` cannot be executed as efficiently as the other cases because Flink has to buffer *all* elements for a window internally before invoking the function. This can be mitigated by combining a `ProcessWindowFunction` with a `ReduceFunction`, or `AggregateFunction` to get both incremental aggregation of window elements and the additional window metadata that the `ProcessWindowFunction` receives. We will look at examples for each of these variants.



> 两种窗口函数
>
> 1. 增量计算函数  ReduceFunction、AggregateFunction
>
>    只保存中间结果，计算效率高。
>
> 2. 全量计算函数 ProcessWindowFunction
>    先缓存该窗口的所有的元素，等到触发条件后对窗口内的所有元素执行计算。占内存。