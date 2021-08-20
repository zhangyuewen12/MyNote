# 一、Flink支持哪几种窗口函数

Flink1.13支持三种窗口函数

>Apache Flink provides 3 built-in windowing TVFs: TUMBLE, `HOP` and `CUMULATE`. The return value of windowing TVF is a new relation that includes all columns of original relation as well as additional 3 columns named “window_start”, “window_end”, “window_time” to indicate the assigned window. 

## TUMBLE

```sql
TUMBLE(TABLE data, DESCRIPTOR(timecol), size)
三个参数:1. 表名 2. 时间属性的列 3.窗口大小
```

## HOP 

```sql
HOP(TABLE data, DESCRIPTOR(timecol), slide, size [, offset ])
参数：1.表名 2. 时间属性的列 3. 滑动步长 4. 窗口大小 5. 偏移量
```

## CUMULATE

```sql
CUMULATE(TABLE data, DESCRIPTOR(timecol), step, size)
参数：1. 表名 2.时间属性的列 3.每次累加操作的窗口大小 4.窗口大小
```

## Session Windows

 (will be supported soon)

## 窗口聚合操作

Window aggregations are defined in the `GROUP BY` clause contains “window_start” and “window_end” columns of the relation applied [Windowing TVF](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-tvf/). Just like queries with regular `GROUP BY` clauses, queries with a group by window aggregation will compute a single result row per group.

```sql
SELECT ...
FROM <windowed_table> -- relation applied windowing TVF
GROUP BY window_start, window_end, ...
```



# 二、Flink怎么保证一致性原则

## Checkpointing机制

Every function and operator in Flink can be **stateful** (see [working with state](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/concepts/stateful-stream-processing/) for details). Stateful functions store data across the processing of individual elements/events, making state a critical building block for any type of more elaborate operation.

In order to make state fault tolerant, Flink needs to **checkpoint** the state. Checkpoints allow Flink to recover state and positions in the streams to give the application the same semantics as a failure-free execution.

## Source支持快照机制，Sink需要开启checkpointing 机制

Flink can guarantee exactly-once state updates to user-defined state only when the source participates in the snapshotting mechanism.

To guarantee end-to-end exactly-once record delivery (in addition to exactly-once state semantics), the data sink needs to take part in the checkpointing mechanism. 

# 三、Flink的watermarker是什么作用

## 作用

WaterMark 用于处理乱序时间，而正确地处理乱序时间，通常是用Watermark机制结合窗口来实现。

>In order to work with *event time*, Flink needs to know the events *timestamps*, meaning each element in the stream needs to have its event timestamp *assigned*. This is usually done by accessing/extracting the timestamp from some field in the element by using a `TimestampAssigner`.
>
>Timestamp assignment goes hand-in-hand with generating watermarks, which tell the system about progress in event time. You can configure this by specifying a `WatermarkGenerator`.
>
>The Flink API expects a `WatermarkStrategy` that contains both a `TimestampAssigner` and `WatermarkGenerator`. A number of common strategies are available out of the box as static methods on `WatermarkStrategy`, but users can also build their own strategies when required.

对于事件时间语义，需要知道每个element对应的具体的时间戳，主要包含两个:

1. TimestampAssigner 主要作用是为element分配timestamp
2. WatermarkGenerator 主要作用是产生watermaker,watermark 用于告诉系统处理进度。

WatermarkStrategy 中同时包含了TimestampAssigner和WatermarkGenerator。

## Watermark的生成方式

1) directly on sources 

The first option is preferable, because it allows sources to exploit knowledge about shards/partitions/splits in the watermarking logic. Sources can usually then track watermarks at a finer level and the overall watermark produced by a source will be more accurate.

2) after non-source operation.

The second option (setting a `WatermarkStrategy` after arbitrary operations) should only be used if you cannot set a strategy directly on the source。

## Watermark的生成策略

```java
/**
 * The {@code WatermarkGenerator} generates watermarks either based on events or
 * periodically (in a fixed interval).
 *
 * <p><b>Note:</b> This WatermarkGenerator subsumes the previous distinction between the
 * {@code AssignerWithPunctuatedWatermarks} and the {@code AssignerWithPeriodicWatermarks}.
 */
@Public
public interface WatermarkGenerator<T> {

    /**
     * Called for every event, allows the watermark generator to examine 
     * and remember the event timestamps, or to emit a watermark based on
     * the event itself.
     */
    void onEvent(T event, long eventTimestamp, WatermarkOutput output);

    /**
     * Called periodically, and might emit a new watermark, or not.
     *
     * <p>The interval in which this method is called and Watermarks 
     * are generated depends on {@link ExecutionConfig#getAutoWatermarkInterval()}.
     */
    void onPeriodicEmit(WatermarkOutput output);
}
```

There are two different styles of watermark generation: *periodic* and *punctuated*.

A periodic generator usually observes the incoming events via `onEvent()` and then emits a watermark when the framework calls `onPeriodicEmit()`.

A puncutated generator will look at events in `onEvent()` and wait for special *marker events* or *punctuations* that carry watermark information in the stream. When it sees one of these events it emits a watermark immediately. Usually, punctuated generators don’t emit a watermark from `onPeriodicEmit()`.

## Periodic WatermarkGenerato

```java
/**
 * This generator generates watermarks assuming that elements arrive out of order,
 * but only to a certain degree. The latest elements for a certain timestamp t will arrive
 * at most n milliseconds after the earliest elements for timestamp t.
 */
public class BoundedOutOfOrdernessGenerator implements WatermarkGenerator<MyEvent> {

    private final long maxOutOfOrderness = 3500; // 3.5 seconds

    private long currentMaxTimestamp;

    @Override
    public void onEvent(MyEvent event, long eventTimestamp, WatermarkOutput output) {
        currentMaxTimestamp = Math.max(currentMaxTimestamp, eventTimestamp);
    }

    @Override
    public void onPeriodicEmit(WatermarkOutput output) {
        // emit the watermark as current highest timestamp minus the out-of-orderness bound
        output.emitWatermark(new Watermark(currentMaxTimestamp - maxOutOfOrderness - 1));
    }

}

/**
 * This generator generates watermarks that are lagging behind processing time 
 * by a fixed amount. It assumes that elements arrive in Flink after a bounded delay.
 */
public class TimeLagWatermarkGenerator implements WatermarkGenerator<MyEvent> {

    private final long maxTimeLag = 5000; // 5 seconds

    @Override
    public void onEvent(MyEvent event, long eventTimestamp, WatermarkOutput output) {
        // don't need to do anything because we work on processing time
    }

    @Override
    public void onPeriodicEmit(WatermarkOutput output) {
        output.emitWatermark(new Watermark(System.currentTimeMillis() - maxTimeLag));
    }
}
```

>A periodic generator observes stream events and generates watermarks periodically (possibly depending on the stream elements, or purely based on processing time).
>
>The interval (every *n* milliseconds) in which the watermark will be generated is defined via `ExecutionConfig.setAutoWatermarkInterval(...)`. The generators’s `onPeriodicEmit()` method will be called each time, and a new watermark will be emitted if the returned watermark is non-null and larger than the previous watermark.
>
>Here we show two simple examples of watermark generators that use periodic watermark generation. Note that Flink ships with `BoundedOutOfOrdernessWatermarks`, which is a `WatermarkGenerator` that works similarly to the `BoundedOutOfOrdernessGenerator` shown below. You can read about using that [here](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/datastream/event-time/built_in/).

## Punctuated WatermarkGenerator

```java
public class PunctuatedAssigner implements WatermarkGenerator<MyEvent> {

    @Override
    public void onEvent(MyEvent event, long eventTimestamp, WatermarkOutput output) {
        if (event.hasWatermarkMarker()) {
            output.emitWatermark(new Watermark(event.getWatermarkTimestamp()));
        }
    }

    @Override
    public void onPeriodicEmit(WatermarkOutput output) {
        // don't need to do anything because we emit in reaction to events above
    }
}
```

>A punctuated watermark generator will observe the stream of events and emit a watermark whenever it sees a special element that carries watermark information.

**Note**: It is possible to generate a watermark on every single event. However, because each watermark causes some computation downstream, an excessive number of watermarks degrades performance.

# 四、Flink支持的时间属性

事件时间

处理时间

集群时间

# 五、Flink State

State按照是否有key划分为KeyedState和OperatorState两种

## Operator State

*Operator State* (or *non-keyed state*) is state that is is bound to one parallel operator instance. The [Kafka Connector](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/connectors/datastream/kafka/) is a good motivating example for the use of Operator State in Flink. Each parallel instance of the Kafka consumer maintains a map of topic partitions and offsets as its Operator State.

The Operator State interfaces support redistributing state among parallel operator instances when the parallelism is changed. There are different schemes for doing this redistribution.

In a typical stateful Flink Application you don’t need operators state. It is mostly a special type of state that is used in source/sink implementations and scenarios where you don’t have a key by which state can be partitioned.

**Notes:** Operator state is still not supported in Python DataStream API.

通常用在Souce 或Sink 中，在这种场景下，不存在可以使用的key。

## Keyed State

The keyed state interfaces provides access to different types of state that are all scoped to the key of the current input element. This means that this type of state can only be used on a `KeyedStream`, which can be created via `stream.keyBy(…)` in Java/Scala API or `stream.key_by(…)` in Python API.

ValueState<T>

ListState<T>

ReducingState<T>

AggregatingState<IN, OUT>

MapState<UK, UV>

All types of state also have a method `clear()` that clears the state for the currently active key, i.e. the key of the input element.

It is important to keep in mind that these state objects are only used for interfacing with state. The state is not necessarily stored inside but might reside on disk or somewhere else. The second thing to keep in mind is that the value you get from the state depends on the key of the input element. So the value you get in one invocation of your user function can differ from the value in another invocation if the keys involved are different.

>有两种需要注意:
>
>1. state不仅可以存储在内部，也可以存储在磁盘或者其他地方
>2. 从state中获取的值依赖于输入element的key。

To get a state handle, you have to create a `StateDescriptor`. This holds the name of the state (as we will see later, you can create several states, and they have to have unique names so that you can reference them), the type of the values that the state holds, and possibly a user-specified function, such as a `ReduceFunction`. Depending on what type of state you want to retrieve, you create either a `ValueStateDescriptor`, a `ListStateDescriptor`, an `AggregatingStateDescriptor`, a `ReducingStateDescriptor`, or a `MapStateDescriptor`.

State is accessed using the `RuntimeContext`, so it is only possible in *rich functions*.

>StateDescriptor 使用state必须创建的类，包含一下几个地方
>
>1. state的名字
>2. state存储的Value类型
>3. 用户自定义函数
>
>需要通过RuntimeContext获取State，而这个方法需要实现rich functions函数

```java
public class CountWindowAverage extends RichFlatMapFunction<Tuple2<Long, Long>, Tuple2<Long, Long>> {

    /**
     * The ValueState handle. The first field is the count, the second field a running sum.
     */
    private transient ValueState<Tuple2<Long, Long>> sum;

    @Override
    public void flatMap(Tuple2<Long, Long> input, Collector<Tuple2<Long, Long>> out) throws Exception {

        // access the state value
        Tuple2<Long, Long> currentSum = sum.value();

        // update the count
        currentSum.f0 += 1;

        // add the second field of the input value
        currentSum.f1 += input.f1;

        // update the state
        sum.update(currentSum);

        // if the count reaches 2, emit the average and clear the state
        if (currentSum.f0 >= 2) {
            out.collect(new Tuple2<>(input.f0, currentSum.f1 / currentSum.f0));
            sum.clear();
        }
    }

    @Override
    public void open(Configuration config) {
        ValueStateDescriptor<Tuple2<Long, Long>> descriptor =
                new ValueStateDescriptor<>(
                        "average", // the state name
                        TypeInformation.of(new TypeHint<Tuple2<Long, Long>>() {}), // type information
                        Tuple2.of(0L, 0L)); // default value of the state, if nothing was set
        sum = getRuntimeContext().getState(descriptor);
    }
}

// this can be used in a streaming program like this (assuming we have a StreamExecutionEnvironment env)
env.fromElements(Tuple2.of(1L, 3L), Tuple2.of(1L, 5L), Tuple2.of(1L, 7L), Tuple2.of(1L, 4L), Tuple2.of(1L, 2L))
        .keyBy(value -> value.f0)
        .flatMap(new CountWindowAverage())
        .print();

// the printed output will be (1,4) and (1,5)
```

## State Backends

### HashMapStateBackend

The *HashMapStateBackend* holds data internally as objects on the Java heap. Key/value state and window operators hold hash tables that store the values, triggers, etc.

The HashMapStateBackend is encouraged for:

- Jobs with large state, long windows, large key/value states.
- All high-availability setups.

### EmbeddedRocksDBStateBackend

The EmbeddedRocksDBStateBackend holds in-flight data in a [RocksDB](http://rocksdb.org/) database that is (per default) stored in the TaskManager local data directories. Unlike storing java objects in `HashMapStateBackend`, data is stored as serialized byte arrays, which are mainly defined by the type serializer, resulting in key comparisons being byte-wise instead of using Java’s `hashCode()` and `equals()` methods.

The EmbeddedRocksDBStateBackend always performs asynchronous snapshots.

Limitations of the EmbeddedRocksDBStateBackend:

- As RocksDB’s JNI bridge API is based on byte[], the maximum supported size per key and per value is 2^31 bytes each. States that use merge operations in RocksDB (e.g. ListState) can silently accumulate value sizes > 2^31 bytes and will then fail on their next retrieval. This is currently a limitation of RocksDB JNI.

The EmbeddedRocksDBStateBackend is encouraged for:

- Jobs with very large state, long windows, large key/value states.
- All high-availability setups.

### State 存储新旧版本的区别

> Beginning in **Flink 1.13**, the community reworked its public state backend classes to help users better understand the separation of local state storage and checkpoint storage. This change does not affect the runtime implementation or characteristics of Flink’s state backend or checkpointing process; it is simply to communicate intent better. Users can migrate existing applications to use the new API without losing any state or consistency.
>
> 从Flink1.13后，社区重构了state backend 类去帮助用户更好的理解本地状态存储和checkpoint存储的区别。这种改变并不影响实际的运行时实现。只是为了用户更好的理解。



#### MemoryStateBackend

The legacy `MemoryStateBackend` is equivalent to using [`HashMapStateBackend`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/state_backends/#the-hashmapstatebackend) and [`JobManagerCheckpointStorage`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/checkpoints/#the-jobmanagercheckpointstorage).

`flink-conf.yaml` configuration 

```yaml
state.backend: hashmap

# Optional, Flink will automatically default to JobManagerCheckpointStorage
# when no checkpoint directory is specified.
state.checkpoint-storage: jobmanager
```

#### FsStateBackend

The legacy `FsStateBackend` is equivalent to using [`HashMapStateBackend`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/state_backends/#the-hashmapstatebackend) and [`FileSystemCheckpointStorage`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/checkpoints/#the-filesystemcheckpointstorage).

`flink-conf.yaml` configuration [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/state_backends/#flink-confyaml-configuration-1)

```yaml
state.backend: hashmap
state.checkpoints.dir: file:///checkpoint-dir/

# Optional, Flink will automatically default to FileSystemCheckpointStorage
# when a checkpoint directory is specified.
state.checkpoint-storage: filesystem
```

#### RocksDBStateBackend

The legacy `RocksDBStateBackend` is equivalent to using [`EmbeddedRocksDBStateBackend`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/state_backends/#the-embeddedrocksdbstatebackend) and [`FileSystemCheckpointStorage`](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/checkpoints/#the-filesystemcheckpointstorage).

`flink-conf.yaml` configuration [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/state_backends/#flink-confyaml-configuration-2)

```yaml
state.backend: rocksdb
state.checkpoints.dir: file:///checkpoint-dir/

# Optional, Flink will automatically default to FileSystemCheckpointStorage
# when a checkpoint directory is specified.
state.checkpoint-storage: filesystem
```

