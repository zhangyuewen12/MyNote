

```
checkpoints
保存在的状态存在什么地方，有checkpoint storage 决定。
checkpoint storage options 主要分为:
- *JobManagerCheckpointStorage* 将检查点快照存在jobmanage的java堆中
- *FileSystemCheckpointStorage* 将检查点快照保存在文件系统中

state backends
state在内部的表示方式，以及在检查点上的持久化方式和位置取决于所选的**状态后端**。
state backends 主要分为:
- *HashMapStateBackend*    	 

The HashMapStateBackend holds data internally as objects on the Java heap. Key/value state and window operators hold hash tables that store the values, triggers, etc.

- *EmbeddedRocksDBStateBackend*
The EmbeddedRocksDBStateBackend holds in-flight data in a [RocksDB](http://rocksdb.org/) database that is (per default) stored in the TaskManager local data directories. 
```



# Checkpoints [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/checkpoints/#checkpoints)



## Overview [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/checkpoints/#overview)

Checkpoints make state in Flink fault tolerant by allowing state and the corresponding stream positions to be recovered, thereby giving the application the same semantics as a failure-free execution.

See [Checkpointing](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/datastream/fault-tolerance/checkpointing/) for how to enable and configure checkpoints for your program.

## Checkpoint Storage [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/checkpoints/#checkpoint-storage)

When checkpointing is enabled, managed state is persisted to ensure consistent recovery in case of failures. Where the state is persisted during checkpointing depends on the chosen **Checkpoint Storage**.

## Available Checkpoint Storage Options [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/checkpoints/#available-checkpoint-storage-options)

Out of the box, Flink bundles these checkpoint storage types:

- *JobManagerCheckpointStorage*
- *FileSystemCheckpointStorage*

> If a checkpoint directory is configured `FileSystemCheckpointStorage` will be used, otherwise the system will use the `JobManagerCheckpointStorage`.

### The JobManagerCheckpointStorage [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/checkpoints/#the-jobmanagercheckpointstorage)

The *JobManagerCheckpointStorage* stores checkpoint snapshots in the JobManager’s heap.

It can be configured to fail the checkpoint if it goes over a certain size to avoid `OutOfMemoryError`’s on the JobManager. To set this feature, users can instantiate a `JobManagerCheckpointStorage` with the corresponding max size:

```java
new JobManagerCheckpointStorage(MAX_MEM_STATE_SIZE);
```

Limitations of the `JobManagerCheckpointStorage`:

- The size of each individual state is by default limited to 5 MB. This value can be increased in the constructor of the `JobManagerCheckpointStorage`.
- Irrespective of the configured maximal state size, the state cannot be larger than the Akka frame size (see [Configuration](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/)).
- The aggregate state must fit into the JobManager memory.

The JobManagerCheckpointStorage is encouraged for:

- Local development and debugging
- Jobs that use very little state, such as jobs that consist only of record-at-a-time functions (Map, FlatMap, Filter, …). The Kafka Consumer requires very little state.

### The FileSystemCheckpointStorage [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/checkpoints/#the-filesystemcheckpointstorage)

The *FileSystemCheckpointStorage* is configured with a file system URL (type, address, path), such as “hdfs://namenode:40010/flink/checkpoints” or “file:///data/flink/checkpoints”.

Upon checkpointing, it writes state snapshots into files in the configured file system and directory. Minimal metadata is stored in the JobManager’s memory (or, in high-availability mode, in the metadata checkpoint).

If a checkpoint directory is specified, `FileSystemCheckpointStorage` will be used to persist checkpoint snapshots.

The `FileSystemCheckpointStorage` is encouraged for:

- All high-availability setups.

It is also recommended to set [managed memory](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_setup_tm/#managed-memory) to zero. This will ensure that the maximum amount of memory is allocated for user code on the JVM.





# State Backends [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/state_backends/#state-backends)

Programs written in the [Data Stream API](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/datastream/overview/) often hold state in various forms:

- Windows gather elements or aggregates until they are triggered
- Transformation functions may use the key/value state interface to store values
- Transformation functions may implement the `CheckpointedFunction` interface to make their local variables fault tolerant

See also [state section](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/datastream/fault-tolerance/state/) in the streaming API guide.

When checkpointing is activated, such state is persisted upon checkpoints to guard against data loss and recover consistently. How the state is represented internally, and how and where it is persisted upon checkpoints depends on the chosen **State Backend**.

## Available State Backends [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/state_backends/#available-state-backends)

Out of the box, Flink bundles these state backends:

- *HashMapStateBackend*
- *EmbeddedRocksDBStateBackend*

If nothing else is configured, the system will use the HashMapStateBackend.

### The HashMapStateBackend [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/state_backends/#the-hashmapstatebackend)

The *HashMapStateBackend* holds data internally as objects on the Java heap. Key/value state and window operators hold hash tables that store the values, triggers, etc.

The HashMapStateBackend is encouraged for:

- Jobs with large state, long windows, large key/value states.
- All high-availability setups.

It is also recommended to set [managed memory](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_setup_tm/#managed-memory) to zero. This will ensure that the maximum amount of memory is allocated for user code on the JVM.

### The EmbeddedRocksDBStateBackend [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/state_backends/#the-embeddedrocksdbstatebackend)

The EmbeddedRocksDBStateBackend holds in-flight data in a [RocksDB](http://rocksdb.org/) database that is (per default) stored in the TaskManager local data directories. Unlike storing java objects in `HashMapStateBackend`, data is stored as serialized byte arrays, which are mainly defined by the type serializer, resulting in key comparisons being byte-wise instead of using Java’s `hashCode()` and `equals()` methods.

The EmbeddedRocksDBStateBackend always performs asynchronous snapshots.

Limitations of the EmbeddedRocksDBStateBackend:

- As RocksDB’s JNI bridge API is based on byte[], the maximum supported size per key and per value is 2^31 bytes each. States that use merge operations in RocksDB (e.g. ListState) can silently accumulate value sizes > 2^31 bytes and will then fail on their next retrieval. This is currently a limitation of RocksDB JNI.

The EmbeddedRocksDBStateBackend is encouraged for:

- Jobs with very large state, long windows, large key/value states.
- All high-availability setups.

Note that the amount of state that you can keep is only limited by the amount of disk space available. This allows keeping very large state, compared to the HashMapStateBackend that keeps state in memory. This also means, however, that the maximum throughput that can be achieved will be lower with this state backend. All reads/writes from/to this backend have to go through de-/serialization to retrieve/store the state objects, which is also more expensive than always working with the on-heap representation as the heap-based backends are doing.

Check also recommendations about the [task executor memory configuration](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/memory/mem_tuning/#rocksdb-state-backend) for the EmbeddedRocksDBStateBackend.

EmbeddedRocksDBStateBackend is currently the only backend that offers incremental checkpoints (see [here](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/large_state_tuning/)).

Certain RocksDB native metrics are available but disabled by default, you can find full documentation [here](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#rocksdb-native-metrics)

The total memory amount of RocksDB instance(s) per slot can also be bounded, please refer to documentation [here](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/large_state_tuning/#bounding-rocksdb-memory-usage) for details.