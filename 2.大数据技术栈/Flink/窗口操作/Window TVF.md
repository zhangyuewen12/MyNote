https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-tvf/

# Windowing table-valued functions (Windowing TVFs)

Windows are at the heart of processing infinite streams. Windows split the stream into “buckets” of finite size, over which we can apply computations. This document focuses on how windowing is performed in Flink SQL and how the programmer can benefit to the maximum from its offered functionality.

```
window操作是处理无限流的主要内容，window将流分割成了有限的buckets，我们可以在这些buckets上计算。
这个文档的重点是：如何在Flink SQL中执行window操作，以及程序开发者如何从window操作中得到帮助。
```

Apache Flink provides several window table-valued functions (TVF) to divide the elements of your table into windows, including:

- [Tumble Windows](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-tvf/#tumble)
- [Hop Windows](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-tvf/#hop)
- [Cumulate Windows](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-tvf/#cumulate)
- Session Windows (will be supported soon)

Note that each element can logically belong to more than one window, depending on the windowing table-valued function you use. For example, HOP windowing creates overlapping windows wherein a single element can be assigned to multiple windows.

```
注意，逻辑上，每个element可以属于多个window，取决于你使用的window tvf 函数。
例如，Hop window操作可以创建 “重叠窗口”,其中 一个element可以被分配到多个windows中。
```



Windowing TVFs are Flink defined Polymorphic Table Functions (abbreviated PTF). PTF is part of the SQL 2016 standard, a special table-function, but can have a table as a parameter. PTF is a powerful feature to change the shape of a table. Because PTFs are used semantically like tables, their invocation occurs in a `FROM` clause of a `SELECT` statement.

```
window TVFs 被定义为动态表函数（PTF）。PTF是SQL2016标准的一部分，一种特殊的表函数，但是可以将一个表作为输入参数。PTF是一种非常强大的特性，主要是改变表的结构。由于PTF的使用通过和表类似，对它们的调用通常发生再select语句的from子句后。
```



Windowing TVFs is a replacement of legacy [Grouped Window Functions](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-agg/#group-window-aggregation-deprecated). Windowing TVFs is more SQL standard compliant and more powerful to support complex window-based computations, e.g. Window TopN, Window Join. However, [Grouped Window Functions](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-agg/#group-window-aggregation) can only support Window Aggregation.

See more how to apply further computations based on windowing TVF:

- [Window Aggregation](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-agg/)
- [Window TopN](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-topn/)
- Window Join (will be supported soon)

```
window TVF 是旧的分组窗口函数的替换方案。Window TVF是更符合SQL 准则的，对于基于窗口的复杂计算的支持更好。例如，window TOPN、window Join。而 分组窗口函数只支持窗口聚合。
```

## Window Functions [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-tvf/#window-functions)

Apache Flink provides 3 built-in windowing TVFs: TUMBLE, `HOP` and `CUMULATE`. The return value of windowing TVF is a new relation that includes all columns of original relation as well as additional 3 columns named “window_start”, “window_end”, “window_time” to indicate the assigned window. The “window_time” field is a [time attributes](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/time_attributes/) of the window after windowing TVF which can be used in subsequent time-based operations, e.g. another windowing TVF, or [interval joins](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/joins/#interval-joins), [over aggregations](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/over-agg/). The value of `window_time` always equal to `window_end - 1ms`.

```
Flink提供了三种内置的window TVF:TUMBLE,HOP,CUMULATE.window TVF的返回值是一个新的表，这个表包含了原表的所有的列并且还有额外的三个列，window_start,window_end,window_time。window_time是表示分配的窗口。window_time列是一个窗口的时间属性，该窗口是在window tvf函数后获得的窗口，这个时间属性可以被用在接下来的基于时间的操作中。window_time通常是等于 window_end - 1ms
```



### TUMBLE [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-tvf/#tumble)

The `TUMBLE` function assigns each element to a window of specified window size. Tumbling windows have a fixed size and do not overlap. For example, suppose you specify a tumbling window with a size of 5 minutes. In that case, Flink will evaluate the current window, and a new window started every five minutes, as illustrated by the following figure.

```
TUMBLE函数将每个element分配到一个指定窗口大小的窗口中。
TUMBEL窗口有一个固定的大小，不会重叠。
例如，假如你指定了一个五分钟大小的窗口，Flink将会计算当前的窗口，并且每隔五分钟开创一个新的窗口。如下图
```



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210616102253705.png" alt="image-20210616102253705" style="zoom:50%;" />

The `TUMBLE` function assigns a window for each row of a relation based on a [time attribute](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/time_attributes/) column. The return value of `TUMBLE` is a new relation that includes all columns of original relation as well as additional 3 columns named “window_start”, “window_end”, “window_time” to indicate the assigned window. The original time attribute “timecol” will be a regular timestamp column after window TVF.

`TUMBLE` function takes three required parameters:

```sql
TUMBLE(TABLE data, DESCRIPTOR(timecol), size)
```

- `data`: is a table parameter that can be any relation with a time attribute column.
- `timecol`: is a column descriptor indicating which [time attributes](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/time_attributes/) column of data should be mapped to tumbling windows.
- `size`: is a duration specifying the width of the tumbling windows.

Here is an example invocation on the `Bid` table:

```
-- tables must have time attribute, e.g. `bidtime` in this table
desc Bid;
+-------------+------------------------+------+-----+--------+---------------------------------+
|        name |                   type | null | key | extras |                       watermark |
+-------------+------------------------+------+-----+--------+---------------------------------+
|     bidtime | TIMESTAMP(3) *ROWTIME* | true |     |        | `bidtime` - INTERVAL '1' SECOND |
|       price |         DECIMAL(10, 2) | true |     |        |                                 |
|        item |                 STRING | true |     |        |                                 |
+-------------+------------------------+------+-----+--------+---------------------------------+

SELECT * FROM Bid;
+------------------+-------+------+
|          bidtime | price | item |
+------------------+-------+------+
| 2020-04-15 08:05 |  4.00 | C    |
| 2020-04-15 08:07 |  2.00 | A    |
| 2020-04-15 08:09 |  5.00 | D    |
| 2020-04-15 08:11 |  3.00 | B    |
| 2020-04-15 08:13 |  1.00 | E    |
| 2020-04-15 08:17 |  6.00 | F    |
+------------------+-------+------+


SELECT * FROM TABLE(
   TUMBLE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '10' MINUTES));

Flink SQL> SELECT * FROM TABLE(
   TUMBLE(
     DATA => TABLE Bid,
     TIMECOL => DESCRIPTOR(bidtime),
     SIZE => INTERVAL '10' MINUTES));
+------------------+-------+------+------------------+------------------+-------------------------+
|          bidtime | price | item |     window_start |       window_end |            window_time  |
+------------------+-------+------+------------------+------------------+-------------------------+
| 2020-04-15 08:05 |  4.00 | C    | 2020-04-15 08:00 | 2020-04-15 08:10 | 2020-04-15 08:09:59.999 |
| 2020-04-15 08:07 |  2.00 | A    | 2020-04-15 08:00 | 2020-04-15 08:10 | 2020-04-15 08:09:59.999 |
| 2020-04-15 08:09 |  5.00 | D    | 2020-04-15 08:00 | 2020-04-15 08:10 | 2020-04-15 08:09:59.999 |
| 2020-04-15 08:11 |  3.00 | B    | 2020-04-15 08:10 | 2020-04-15 08:20 | 2020-04-15 08:19:59.999 |
| 2020-04-15 08:13 |  1.00 | E    | 2020-04-15 08:10 | 2020-04-15 08:20 | 2020-04-15 08:19:59.999 |
| 2020-04-15 08:17 |  6.00 | F    | 2020-04-15 08:10 | 2020-04-15 08:20 | 2020-04-15 08:19:59.999 |
+------------------+-------+------+------------------+------------------+-------------------------+


SELECT window_start, window_end, SUM(price)
  FROM TABLE(
    TUMBLE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '10' MINUTES))
  GROUP BY window_start, window_end;
+------------------+------------------+-------+
|     window_start |       window_end | price |
+------------------+------------------+-------+
| 2020-04-15 08:00 | 2020-04-15 08:10 | 11.00 |
| 2020-04-15 08:10 | 2020-04-15 08:20 | 10.00 |
+------------------+------------------+-------+
```



### HOP [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-tvf/#hop)

The `HOP` function assigns elements to windows of fixed length. Like a `TUMBLE` windowing function, the size of the windows is configured by the window size parameter. An additional window slide parameter controls how frequently a hopping window is started. Hence, hopping windows can be overlapping if the slide is smaller than the window size. In this case, elements are assigned to multiple windows. Hopping windows are also known as “sliding windows”.

For example, you could have windows of size 10 minutes that slides by 5 minutes. With this, you get every 5 minutes a window that contains the events that arrived during the last 10 minutes, as depicted by the following figure.

```
Hop 函数将element 分配到指定大小的window中。和TUMBLE window 函数一样，窗口的大小通过window size 参数配置。一个额外的window 滑动参数控制 hop window 滑动的频率。因此，hopwindow可以重叠，如果滑动步长比窗口大小 更小。这种情况下，element被分配到多个窗口，hop window 通常也被称为滑动窗口。 
例如，你可以设置一个10分钟的窗口大小，滑动步长为5分钟。在这种情况下，你每隔五分钟可以得到一个窗口，这个窗口包含的最近十分钟到达的事件。如下图
```

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210616105131304.png" alt="image-20210616105131304" style="zoom:50%;" />

The `HOP` function assigns windows that cover rows within the interval of size and shifting every slide based on a [time attribute](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/time_attributes/) column. The return value of `HOP` is a new relation that includes all columns of original relation as well as additional 3 columns named “window_start”, “window_end”, “window_time” to indicate the assigned window. The original time attribute “timecol” will be a regular timestamp column after windowing TVF.

`HOP` takes three required parameters.

```sql
HOP(TABLE data, DESCRIPTOR(timecol), slide, size [, offset ])
```

- `data`: is a table parameter that can be any relation with an time attribute column.
- `timecol`: is a column descriptor indicating which [time attributes](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/time_attributes/) column of data should be mapped to hopping windows.
- `slide`: is a duration specifying the duration between the start of sequential hopping windows
- `size`: is a duration specifying the width of the hopping windows.

Here is an example invocation on the `Bid` table:

```
SELECT * FROM TABLE(
    HOP(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '5' MINUTES, INTERVAL '10' MINUTES));
    
SELECT * FROM TABLE(
    HOP(
      DATA => TABLE Bid,
      TIMECOL => DESCRIPTOR(bidtime),
      SLIDE => INTERVAL '5' MINUTES,
      SIZE => INTERVAL '10' MINUTES));
+------------------+-------+------+------------------+------------------+-------------------------+
|          bidtime | price | item |     window_start |       window_end |           window_time   |
+------------------+-------+------+------------------+------------------+-------------------------+
| 2020-04-15 08:05 |  4.00 | C    | 2020-04-15 08:00 | 2020-04-15 08:10 | 2020-04-15 08:09:59.999 |
| 2020-04-15 08:05 |  4.00 | C    | 2020-04-15 08:05 | 2020-04-15 08:15 | 2020-04-15 08:14:59.999 |
| 2020-04-15 08:07 |  2.00 | A    | 2020-04-15 08:00 | 2020-04-15 08:10 | 2020-04-15 08:09:59.999 |
| 2020-04-15 08:07 |  2.00 | A    | 2020-04-15 08:05 | 2020-04-15 08:15 | 2020-04-15 08:14:59.999 |
| 2020-04-15 08:09 |  5.00 | D    | 2020-04-15 08:00 | 2020-04-15 08:10 | 2020-04-15 08:09:59.999 |
| 2020-04-15 08:09 |  5.00 | D    | 2020-04-15 08:05 | 2020-04-15 08:15 | 2020-04-15 08:14:59.999 |
| 2020-04-15 08:11 |  3.00 | B    | 2020-04-15 08:05 | 2020-04-15 08:15 | 2020-04-15 08:14:59.999 |
| 2020-04-15 08:11 |  3.00 | B    | 2020-04-15 08:10 | 2020-04-15 08:20 | 2020-04-15 08:19:59.999 |
| 2020-04-15 08:13 |  1.00 | E    | 2020-04-15 08:05 | 2020-04-15 08:15 | 2020-04-15 08:14:59.999 |
| 2020-04-15 08:13 |  1.00 | E    | 2020-04-15 08:10 | 2020-04-15 08:20 | 2020-04-15 08:19:59.999 |
| 2020-04-15 08:17 |  6.00 | F    | 2020-04-15 08:10 | 2020-04-15 08:20 | 2020-04-15 08:19:59.999 |
| 2020-04-15 08:17 |  6.00 | F    | 2020-04-15 08:15 | 2020-04-15 08:25 | 2020-04-15 08:24:59.999 |
+------------------+-------+------+------------------+------------------+-------------------------+
      
      
SELECT window_start, window_end, SUM(price)
  FROM TABLE(
    HOP(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '5' MINUTES, INTERVAL '10' MINUTES))
  GROUP BY window_start, window_end;
+------------------+------------------+-------+
|     window_start |       window_end | price |
+------------------+------------------+-------+
| 2020-04-15 08:00 | 2020-04-15 08:10 | 11.00 |
| 2020-04-15 08:05 | 2020-04-15 08:15 | 15.00 |
| 2020-04-15 08:10 | 2020-04-15 08:20 | 10.00 |
| 2020-04-15 08:15 | 2020-04-15 08:25 |  6.00 |
+------------------+------------------+-------+
```



### CUMULATE [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-tvf/#cumulate)

Cumulating windows are very useful in some scenarios, such as tumbling windows with early firing in a fixed window interval. For example, a daily dashboard draws cumulative UVs from 00:00 to every minute, the UV at 10:00 represents the total number of UV from 00:00 to 10:00. This can be easily and efficiently implemented by CUMULATE windowing.

The `CUMULATE` function assigns elements to windows that cover rows within an initial interval of step size and expand to one more step size (keep window start fixed) every step until the max window size. You can think `CUMULATE` function as applying `TUMBLE` windowing with max window size first, and split each tumbling windows into several windows with same window start and window ends of step-size difference. So cumulating windows do overlap and don’t have a fixed size.

For example, you could have a cumulating window for 1 hour step and 1 day max size, and you will get windows: `[00:00, 01:00)`, `[00:00, 02:00)`, `[00:00, 03:00)`, …, `[00:00, 24:00)` for every day.

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210616110314689.png" alt="image-20210616110314689" style="zoom:50%;" />

The `CUMULATE` functions assigns windows based on a [time attribute](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/time_attributes/) column. The return value of `CUMULATE` is a new relation that includes all columns of original relation as well as additional 3 columns named “window_start”, “window_end”, “window_time” to indicate the assigned window. The original time attribute “timecol” will be a regular timestamp column after window TVF.

`CUMULATE` takes three required parameters.

```sql
CUMULATE(TABLE data, DESCRIPTOR(timecol), step, size)
```

- `data`: is a table parameter that can be any relation with an time attribute column.
- `timecol`: is a column descriptor indicating which [time attributes](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/time_attributes/) column of data should be mapped to tumbling windows.
- `step`: is a duration specifying the increased window size between the end of sequential cumulating windows.
- `size`: is a duration specifying the max width of the cumulating windows. size must be an integral multiple of step .



Here is an example invocation on the Bid table:

```
SELECT * FROM TABLE(
    CUMULATE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '2' MINUTES, INTERVAL '10' MINUTES));
    
SELECT * FROM TABLE(
    CUMULATE(
      DATA => TABLE Bid,
      TIMECOL => DESCRIPTOR(bidtime),
      STEP => INTERVAL '2' MINUTES,
      SIZE => INTERVAL '10' MINUTES));
+------------------+-------+------+------------------+------------------+-------------------------+
|          bidtime | price | item |     window_start |       window_end |            window_time  |
+------------------+-------+------+------------------+------------------+-------------------------+
| 2020-04-15 08:05 |  4.00 | C    | 2020-04-15 08:00 | 2020-04-15 08:06 | 2020-04-15 08:05:59.999 |
| 2020-04-15 08:05 |  4.00 | C    | 2020-04-15 08:00 | 2020-04-15 08:08 | 2020-04-15 08:07:59.999 |
| 2020-04-15 08:05 |  4.00 | C    | 2020-04-15 08:00 | 2020-04-15 08:10 | 2020-04-15 08:09:59.999 |
| 2020-04-15 08:07 |  2.00 | A    | 2020-04-15 08:00 | 2020-04-15 08:08 | 2020-04-15 08:07:59.999 |
| 2020-04-15 08:07 |  2.00 | A    | 2020-04-15 08:00 | 2020-04-15 08:10 | 2020-04-15 08:09:59.999 |
| 2020-04-15 08:09 |  5.00 | D    | 2020-04-15 08:00 | 2020-04-15 08:10 | 2020-04-15 08:09:59.999 |
| 2020-04-15 08:11 |  3.00 | B    | 2020-04-15 08:10 | 2020-04-15 08:12 | 2020-04-15 08:11:59.999 |
| 2020-04-15 08:11 |  3.00 | B    | 2020-04-15 08:10 | 2020-04-15 08:14 | 2020-04-15 08:13:59.999 |
| 2020-04-15 08:11 |  3.00 | B    | 2020-04-15 08:10 | 2020-04-15 08:16 | 2020-04-15 08:15:59.999 |
| 2020-04-15 08:11 |  3.00 | B    | 2020-04-15 08:10 | 2020-04-15 08:18 | 2020-04-15 08:17:59.999 |
| 2020-04-15 08:11 |  3.00 | B    | 2020-04-15 08:10 | 2020-04-15 08:20 | 2020-04-15 08:19:59.999 |
| 2020-04-15 08:13 |  1.00 | E    | 2020-04-15 08:10 | 2020-04-15 08:14 | 2020-04-15 08:13:59.999 |
| 2020-04-15 08:13 |  1.00 | E    | 2020-04-15 08:10 | 2020-04-15 08:16 | 2020-04-15 08:15:59.999 |
| 2020-04-15 08:13 |  1.00 | E    | 2020-04-15 08:10 | 2020-04-15 08:18 | 2020-04-15 08:17:59.999 |
| 2020-04-15 08:13 |  1.00 | E    | 2020-04-15 08:10 | 2020-04-15 08:20 | 2020-04-15 08:19:59.999 |
| 2020-04-15 08:17 |  6.00 | F    | 2020-04-15 08:10 | 2020-04-15 08:18 | 2020-04-15 08:17:59.999 |
| 2020-04-15 08:17 |  6.00 | F    | 2020-04-15 08:10 | 2020-04-15 08:20 | 2020-04-15 08:19:59.999 |
+------------------+-------+------+------------------+------------------+-------------------------+


SELECT window_start, window_end, SUM(price)
  FROM TABLE(
    CUMULATE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '2' MINUTES, INTERVAL '10' MINUTES))
  GROUP BY window_start, window_end;
  
+------------------+------------------+-------+
|     window_start |       window_end | price |
+------------------+------------------+-------+
| 2020-04-15 08:00 | 2020-04-15 08:06 |  4.00 |
| 2020-04-15 08:00 | 2020-04-15 08:08 |  6.00 |
| 2020-04-15 08:00 | 2020-04-15 08:10 | 11.00 |
| 2020-04-15 08:10 | 2020-04-15 08:12 |  3.00 |
| 2020-04-15 08:10 | 2020-04-15 08:14 |  4.00 |
| 2020-04-15 08:10 | 2020-04-15 08:16 |  4.00 |
| 2020-04-15 08:10 | 2020-04-15 08:18 | 10.00 |
| 2020-04-15 08:10 | 2020-04-15 08:20 | 10.00 |
+------------------+------------------+-------+
```

