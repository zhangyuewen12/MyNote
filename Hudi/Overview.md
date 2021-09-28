# Overview

Apache Hudi (pronounced “hoodie”) provides streaming primitives over hadoop compatible storages

- Update/Delete Records (how do I change records in a table?)
- Change Streams (how do I fetch records that changed?)

In this section, we will discuss key concepts & terminologies that are important to understand, to be able to effectively use these primitives.



## Timeline[#](https://hudi.apache.org/docs/overview#timeline)

At its core, Hudi maintains a `timeline` of all actions performed on the table at different `instants` of time that helps provide instantaneous views of the table, while also efficiently supporting retrieval of data in the order of arrival. A Hudi instant consists of the following components

- `Instant action` : Type of action performed on the table
- `Instant time` : Instant time is typically a timestamp (e.g: 20190117010349), which monotonically increases in the order of action's begin time.
- `state` : current state of the instant

Hudi guarantees that the actions performed on the timeline are atomic & timeline consistent based on the instant time.

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210918095431707.png" alt="image-20210918095431707" style="zoom:50%;" />

Example above shows upserts happenings between 10:00 and 10:20 on a Hudi table, roughly every 5 mins, leaving commit metadata on the Hudi timeline, along with other background cleaning/compactions. One key observation to make is that the commit time indicates the `arrival time` of the data (10:20AM), while the actual data organization reflects the actual time or `event time`, the data was intended for (hourly buckets from 07:00). These are two key concepts when reasoning about tradeoffs between latency and completeness of data.

When there is late arriving data (data intended for 9:00 arriving >1 hr late at 10:20), we can see the upsert producing new data into even older time buckets/folders. With the help of the timeline, an incremental query attempting to get all new data that was committed successfully since 10:00 hours, is able to very efficiently consume only the changed files without say scanning all the time buckets > 07:00.

