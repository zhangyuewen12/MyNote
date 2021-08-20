# Savepoints [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/savepoints/#savepoints)



# What is a Savepoint?

A Savepoint is a consistent image of the execution state of a streaming job, created via Flink’s [checkpointing mechanism](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/learn-flink/fault_tolerance/). You can use Savepoints to stop-and-resume, fork, or update your Flink jobs. Savepoints consist of two parts: a directory with (typically large) binary files on stable storage (e.g. HDFS, S3, …) and a (relatively small) meta data file. The files on stable storage represent the net data of the job’s execution state image. The meta data file of a Savepoint contains (primarily) pointers to all files on stable storage that are part of the Savepoint, in form of relative paths.

```
savepoints 是一个job执行状态的连续image，通过checkpoints机制实现，你可以使用savepoints去重启，复制，更新你的job。
savepoints由两个部分组成：一个包含二进制文件的目录和一个元数据文件。
这些在分布式存储上的文件表示job执行状态的网络数据，元数据文件主要包含一些指针，指向存储在分布式存储系统的文件，这些文件也是savepoint的一部分，以一种相对路径的形式。
```



> In order to allow upgrades between programs and Flink versions, it is important to check out the following section about [assigning IDs to your operators](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/ops/state/savepoints/#assigning-operator-ids).

#  How is a Savepoint different from a Checkpoint?

Conceptually, Flink’s Savepoints are different from Checkpoints in a similar way that backups are different from recovery logs in traditional database systems. The primary purpose of Checkpoints is to provide a recovery mechanism in case of unexpected job failures. A Checkpoint’s lifecycle is managed by Flink, i.e. a Checkpoint is created, owned, and released by Flink - without user interaction. As a method of recovery and being periodically triggered, two main design goals for the Checkpoint implementation are i) being as lightweight to create and ii) being as fast to restore from as possible. Optimizations towards those goals can exploit certain properties, e.g. that the job code doesn’t change between the execution attempts. Checkpoints are usually dropped after the job was terminated by the user (except if explicitly configured as retained Checkpoints).

In contrast to all this, Savepoints are created, owned, and deleted by the user. Their use-case is for planned, manual backup and resume. For example, this could be an update of your Flink version, changing your job graph, changing parallelism, forking a second job like for a red/blue deployment, and so on. Of course, Savepoints must survive job termination. Conceptually, Savepoints can be a bit more expensive to produce and restore and focus more on portability and support for the previously mentioned changes to the job.

Flink’s savepoint binary format is unified across all state backends. That means you can take a savepoint with one state backend and then restore it using another.

> State backends did not start producing a common format until version 1.13. Therefore, if you want to switch the state backend you should first upgrade your Flink version then take a savepoint with the new version, and only after that, you can restore it with a different state backend.

