# Window Aggregation

## 一、Window TVF Aggregation

Window aggregations are defined in the `GROUP BY` clause contains “window_start” and “window_end” columns of the relation applied [Windowing TVF](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-tvf/). Just like queries with regular `GROUP BY` clauses, queries with a group by window aggregation will compute a single result row per group.

```
window 聚合操作被定义在group by 子句中， 该字句包含 表的 “window_start”,"window_end" 属性，这个表示使用window TVF后的表。
对window聚合后的表的查询将为每个组得到一个结果，
```



```sql
SELECT ...
FROM <windowed_table> -- relation applied windowing TVF
GROUP BY window_start, window_end, ...
```

Unlike other aggregations on continuous tables, window aggregation do not emit intermediate results but only a final result, the total aggregation at the end of the window. Moreover, window aggregations purge all intermediate state when no longer needed.

```
与其他的聚合操作，window聚合不会立即生成一个结果，只会产生一个最终结果。在window结束后总的聚合结果。
而且，window聚合操作会释放所有的状态，在不需要后。
```

### 1. Windowing TVFs [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-agg/#windowing-tvfs)

```
-- tables must have time attribute, e.g. `bidtime` in this table
-- 表必须有时间属性
Flink SQL> desc Bid;
+-------------+------------------------+------+-----+--------+---------------------------------+
|        name |                   type | null | key | extras |                       watermark |
+-------------+------------------------+------+-----+--------+---------------------------------+
|     bidtime | TIMESTAMP(3) *ROWTIME* | true |     |        | `bidtime` - INTERVAL '1' SECOND |
|       price |         DECIMAL(10, 2) | true |     |        |                                 |
|        item |                 STRING | true |     |        |                                 |
| supplier_id |                 STRING | true |     |        |                                 |
+-------------+------------------------+------+-----+--------+---------------------------------+

Flink SQL> SELECT * FROM Bid;
+------------------+-------+------+-------------+
|          bidtime | price | item | supplier_id |
+------------------+-------+------+-------------+
| 2020-04-15 08:05 | 4.00  | C    | supplier1   |
| 2020-04-15 08:07 | 2.00  | A    | supplier1   |
| 2020-04-15 08:09 | 5.00  | D    | supplier2   |
| 2020-04-15 08:11 | 3.00  | B    | supplier2   |
| 2020-04-15 08:13 | 1.00  | E    | supplier1   |
| 2020-04-15 08:17 | 6.00  | F    | supplier2   |
+------------------+-------+------+-------------+

-- tumbling window aggregation
-- TUMBLing window 聚合
Flink SQL> SELECT window_start, window_end, SUM(price)
  FROM TABLE(
    TUMBLE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '10' MINUTES))
  GROUP BY window_start, window_end;
+------------------+------------------+-------+
|     window_start |       window_end | price |
+------------------+------------------+-------+
| 2020-04-15 08:00 | 2020-04-15 08:10 | 11.00 |
| 2020-04-15 08:10 | 2020-04-15 08:20 | 10.00 |
+------------------+------------------+-------+

-- hopping window aggregation
-- Hopping window 聚合
Flink SQL> SELECT window_start, window_end, SUM(price)
  FROM TABLE(
    HOP(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '5' MINUTES, INTERVAL '10' MINUTES))
  GROUP BY window_start, window_end;
+------------------+------------------+-------+
|     window_start |       window_end | price |
+------------------+------------------+-------+
| 2020-04-15 08:00 | 2020-04-15 08:10 | 11.00 |
| 2020-04-15 08:05 | 2020-04-15 08:15 | 15.00 |
| 2020-04-15 08:10 | 2020-04-15 08:20 | 10.00 |
| 2020-04-15 08:15 | 2020-04-15 08:25 | 6.00  |
+------------------+------------------+-------+


-- cumulative window aggregation
-- 
Flink SQL> SELECT window_start, window_end, SUM(price)
  FROM TABLE(
    CUMULATE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '2' MINUTES, INTERVAL '10' MINUTES))
  GROUP BY window_start, window_end;
  
+------------------+------------------+-------+
|     window_start |       window_end | price |
+------------------+------------------+-------+
| 2020-04-15 08:00 | 2020-04-15 08:06 | 4.00  |
| 2020-04-15 08:00 | 2020-04-15 08:08 | 6.00  |
| 2020-04-15 08:00 | 2020-04-15 08:10 | 11.00 |
| 2020-04-15 08:10 | 2020-04-15 08:12 | 3.00  |
| 2020-04-15 08:10 | 2020-04-15 08:14 | 4.00  |
| 2020-04-15 08:10 | 2020-04-15 08:16 | 4.00  |
| 2020-04-15 08:10 | 2020-04-15 08:18 | 10.00 |
| 2020-04-15 08:10 | 2020-04-15 08:20 | 10.00 |
+------------------+------------------+-------+
```



### 2. GROUPING SETS [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-agg/#grouping-sets)

Window aggregations also support `GROUPING SETS` syntax. Grouping sets allow for more complex grouping operations than those describable by a standard `GROUP BY`. Rows are grouped separately by each specified grouping set and aggregates are computed for each group just as for simple `GROUP BY` clauses.

Window aggregations with `GROUPING SETS` require both the `window_start` and `window_end` columns have to be in the `GROUP BY` clause, but not in the `GROUPING SETS` clause.

```
window 聚合还支持 grouping sets 语法，grouping sets 可以支持更复杂的聚合操作比标准的group by 语法。
行按照指定的分组集被分组，每个组进行计算，就像单独的group by 语义一样。

使用 grouping sets语句，需要window_start,window_end列已经在group by 语义中，但不在grouping sets语义。
```



```
-- ()表示 空 ，按照空分组 所有行都会输入到这个组。

Flink SQL> SELECT window_start, window_end, supplier_id, SUM(price) as price
  FROM TABLE(
    TUMBLE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '10' MINUTES))
  GROUP BY window_start, window_end, GROUPING SETS ((supplier_id), ());
  
+------------------+------------------+-------------+-------+
|     window_start |       window_end | supplier_id | price |
+------------------+------------------+-------------+-------+
| 2020-04-15 08:00 | 2020-04-15 08:10 |      (NULL) | 11.00 |
| 2020-04-15 08:00 | 2020-04-15 08:10 |   supplier2 |  5.00 |
| 2020-04-15 08:00 | 2020-04-15 08:10 |   supplier1 |  6.00 |
| 2020-04-15 08:10 | 2020-04-15 08:20 |      (NULL) | 10.00 |
| 2020-04-15 08:10 | 2020-04-15 08:20 |   supplier2 |  9.00 |
| 2020-04-15 08:10 | 2020-04-15 08:20 |   supplier1 |  1.00 |
+------------------+------------------+-------------+-------+
```



Each sublist of `GROUPING SETS` may specify zero or more columns or expressions and is interpreted the same way as though used directly in the `GROUP BY` clause. An empty grouping set means that all rows are aggregated down to a single group, which is output even if no input rows were present.

References to the grouping columns or expressions are replaced by null values in result rows for grouping sets in which those columns do not appear.



```
每个grouping sets 子集可以指定0个或多个列 或表达式 并 可以 以同样的样被解析，就像直接在group by 语义中使用一样。
一个空的grouping set 意味中，所有的行被聚集到单个group，即使不存在任何输入行，也会输出该组。

在结果集中，对于grouping columns或者表达式的引用 被替换成 null值，当这些列不出现的时候。
```

https://help.aliyun.com/document_detail/167961.html

如果您经常需要对数据进行多维度聚合分析（例如既需要按照a列聚合，也要按照b列聚合，同时要按照a和b两列聚合），您可以使用GROUPING SETS语句进行多维度聚合分析，避免多次使用UNION ALL影响性能。

### 语法格式

```
SELECT [ ALL | DISTINCT ]
{ * | projectItem [, projectItem ]* }
FROM tableExpression
GROUP BY 
[GROUPING SETS { groupItem [, groupItem ]* } ];
```



### 示例

- 测试数据

  | username | month | day  |
  | :------- | :---- | :--- |
  | Lily     | 10    | 1    |
  | Lucy     | 11    | 21   |
  | Lily     | 11    | 21   |



- 测试案例

  ```
  SELECT  
      `month`,
      `day`,
      count(distinct `username`) as uv
  FROM tmall_item
  group by 
  grouping sets((`month`),(`month`,`day`));
  ```

- 测试结果 “这个结果显示的应该是包含过程过程结果”

  | month | day  | uv   |
  | :---- | :--- | :--- |
  | 10    | 1    | 1    |
  | 10    | null | 1    |
  | 11    | 21   | 1    |
  | 11    | null | 1    |
  | 11    | 21   | 2    |
  | 11    | null | 2    |

### ROLLUP [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-agg/#rollup)

`ROLLUP` is a shorthand notation for specifying a common type of grouping set. It represents the given list of expressions and all prefixes of the list, including the empty list.

Window aggregations with `ROLLUP` requires both the `window_start` and `window_end` columns have to be in the `GROUP BY` clause, but not in the `ROLLUP` clause.

For example, the following query is equivalent to the one above.

```
ROLLUP 是一种简写的语法，用于指定分组集的公共类型。它表示 给定的表达式和列表中的所有前缀，包括空列表。
```



```sql
SELECT window_start, window_end, supplier_id, SUM(price) as price
FROM TABLE(
    TUMBLE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '10' MINUTES))
GROUP BY window_start, window_end, ROLLUP (supplier_id);
```

#### CUBE [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-agg/#cube)

`CUBE` is a shorthand notation for specifying a common type of grouping set. It represents the given list and all of its possible subsets - the power set.

Window aggregations with `CUBE` requires both the `window_start` and `window_end` columns have to be in the `GROUP BY` clause, but not in the `CUBE` clause.

For example, the following two queries are equivalent.

```sql
SELECT window_start, window_end, item, supplier_id, SUM(price) as price
  FROM TABLE(
    TUMBLE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '10' MINUTES))
  GROUP BY window_start, window_end, CUBE (supplier_id, item);

SELECT window_start, window_end, item, supplier_id, SUM(price) as price
  FROM TABLE(
    TUMBLE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '10' MINUTES))
  GROUP BY window_start, window_end, GROUPING SETS (
      (supplier_id, item),
      (supplier_id      ),
      (             item),
      (                 )
)
```

### Cascading Window Aggregation [#](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-agg/#cascading-window-aggregation)

The `window_start` and `window_end` columns are regular timestamp columns, not time attributes. Thus they can’t be used as time attributes in subsequent time-based operations. In order to propagate time attributes, you need to additionally add `window_time` column into `GROUP BY` clause. The `window_time` is the third column produced by [Windowing TVFs](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-tvf/#window-functions) which is a time attribute of the assigned window. Adding `window_time` into `GROUP BY` clause makes `window_time` also to be group key that can be selected. Then following queries can use this column for subsequent time-based operations, such as cascading window aggregations and [Window TopN](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-topn/).

The following shows a cascading window aggregation where the first window aggregation propagates the time attribute for the second window aggregation.

```sql
-- tumbling 5 minutes for each supplier_id
CREATE VIEW window1 AS
SELECT window_start, window_end, window_time as rowtime, SUM(price) as partial_price
  FROM TABLE(
    TUMBLE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '5' MINUTES))
  GROUP BY supplier_id, window_start, window_end, window_time;

-- tumbling 10 minutes on the first window
SELECT window_start, window_end, SUM(partial_price) as total_price
  FROM TABLE(
      TUMBLE(TABLE window1, DESCRIPTOR(rowtime), INTERVAL '10' MINUTES))
  GROUP BY window_start, window_end;
```



## 二、Group Window Aggregation

