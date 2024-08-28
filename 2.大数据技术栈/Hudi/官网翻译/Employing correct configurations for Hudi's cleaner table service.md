https://hudi.apache.org/blog/2021/06/10/employing-right-configurations-for-hudi-cleaner/#cleaning-policies





# Employing correct configurations for Hudi's cleaner table service



Apache Hudi provides snapshot isolation between writers and readers. This is made possible by Hudi’s MVCC concurrency model. In this blog, we will explain how to employ the right configurations to manage multiple file versions. Furthermore, we will discuss mechanisms available to users on how to maintain just the required number of old file versions so that long running readers do not fail.

> ApacheHudi在读取和写入之间提供快照隔离。这是通过Hudi的MVCC并发模型实现的。在这个博客中，我们将解释如何使用正确的配置来管理多个文件版本。此外，我们将讨论用户可以使用的机制，如何维护所需数量的旧文件版本，以便长时间运行的阅读器不会出现故障。

### Reclaiming space and keeping your data lake storage costs in check

Hudi provides different table management services to be able to manage your tables on the data lake. One of these services is called the **Cleaner**. As you write more data to your table, for every batch of updates received, Hudi can either generate a new version of the data file with updates applied to records (COPY_ON_WRITE) or write these delta updates to a log file, avoiding rewriting newer version of an existing file (MERGE_ON_READ). In such situations, depending on the frequency of your updates, the number of file versions of log files can grow indefinitely. If your use-cases do not require keeping an infinite history of these versions, it is imperative to have a process that reclaims older versions of the data. This is Hudi’s cleaner service.

> Hudi提供不同的表管理服务，以便能够管理数据湖上的表。其中一项服务被称为**清洁工**。当您向表中写入更多数据时，对于接收到的每一批更新，Hudi可以使用应用于记录的更新生成数据文件的新版本（COPY_ON_write），也可以将这些增量更新写入日志文件，避免重写现有文件的更新版本（MERGE_ON_READ）。在这种情况下，根据更新的频率，日志文件的文件版本数可能无限增长。如果您的用例不需要保留这些版本的无限历史，那么必须有一个回收旧版本数据的过程。这是胡迪的清洁服务。



### Problem Statement

In a data lake architecture, it is a very common scenario to have readers and writers concurrently accessing the same table. As the Hudi cleaner service periodically reclaims older file versions, scenarios arise where a long running query might be accessing a file version that is deemed to be reclaimed by the cleaner. Here, we need to employ the correct configs to ensure readers (aka queries) don’t fail.

> 在数据湖架构中，写入器和读取器同时访问同一个表是非常常见的场景。当Hudi回收期服务定期回收较旧的文件版本时，会出现长时间运行的查询可能正在访问被回收期视为回收的文件版本的情况。这里，我们需要使用正确的配置来确保读者（即查询）不会失败。



### Deeper dive into Hudi Cleaner

To deal with the mentioned scenario, lets understand the different cleaning policies that Hudi offers and the corresponding properties that need to be configured. Options are available to schedule cleaning asynchronously or synchronously. Before going into more details, we would like to explain a few underlying concepts:

- **Hudi base file**: Columnar file which consists of final data after compaction. A base file’s name follows the following naming convention: `<fileId>_<writeToken>_<instantTime>.parquet`. In subsequent writes of this file, file id remains the same and commit time gets updated to show the latest version. This also implies any particular version of a record, given its partition path, can be uniquely located using the file id and instant time.
- **File slice**: A file slice consists of the base file and any log files consisting of the delta, in case of MERGE_ON_READ table type.
- **Hudi File Group**: Any file group in Hudi is uniquely identified by the partition path and the file id that the files in this group have as part of their name. A file group consists of all the file slices in a particular partition path. Also any partition path can have multiple file groups.

> 为了处理上述场景，让我们了解Hudi提供的不同清理策略以及需要配置的相应属性。可以选择异步或同步安排清洁。在详细讨论之前，我们想解释几个基本概念：
> Hudi基本文件：由压缩后的最终数据组成的列文件。基本文件的名称遵循以下命名约定：＜fileId＞_＜writeToken＞_＜instantTime＞.parquet。在该文件的后续写入中，文件id保持不变，提交时间更新为显示最新版本。这也意味着记录的任何特定版本（给定其分区路径）都可以使用文件id和即时时间唯一定位。
> 文件切片：如果是MERGE_ON_READ表类型，文件切片由基本文件和由增量组成的任何日志文件组成。
> Hudi文件组：Hudi中的任何文件组都由分区路径和文件id唯一标识。该组中的文件将文件id作为其名称的一部分。文件组由特定分区路径中的所有文件切片组成。此外，任何分区路径都可以有多个文件组。

### Cleaning Policies

Hudi cleaner currently supports below cleaning policies:

- **KEEP_LATEST_COMMITS**: This is the default policy. This is a temporal cleaning policy that ensures the effect of having lookback into all the changes that happened in the last X commits. Suppose a writer is ingesting data into a Hudi dataset every 30 minutes and the longest running query can take 5 hours to finish, then the user should retain atleast the last 10 commits. With such a configuration, we ensure that the oldest version of a file is kept on disk for at least 5 hours, thereby preventing the longest running query from failing at any point in time. Incremental cleaning is also possible using this policy.
- **KEEP_LATEST_FILE_VERSIONS**: This policy has the effect of keeping N number of file versions irrespective of time. This policy is useful when it is known how many MAX versions of the file does one want to keep at any given time. To achieve the same behaviour as before of preventing long running queries from failing, one should do their calculations based on data patterns. Alternatively, this policy is also useful if a user just wants to maintain 1 latest version of the file.

> -**KEEP_LATEST_COMMITS**：这是默认策略。这是一种临时清理策略，可确保对上次X次提交中发生的所有更改进行回顾。假设一个编写者每30分钟将数据输入一个Hudi数据集，而最长时间的查询可能需要5个小时才能完成，那么用户应该至少保留最后10次提交。通过这样的配置，我们可以确保文件的最旧版本在磁盘上保存至少5个小时，从而防止运行时间最长的查询在任何时间点失败。使用此策略还可以进行增量清理。
> -**KEEP_LATEST_FILE_VERSIONS**：此策略具有保留N个文件版本的效果，而不考虑时间。当知道在任何给定时间要保留多少文件的MAX版本时，此策略非常有用。为了实现与以前相同的行为，防止长时间运行的查询失败，应该根据数据模式进行计算。或者，如果用户只想维护文件的最新版本，此策略也很有用