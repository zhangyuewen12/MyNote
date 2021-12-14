##  Uniq 模型

在某些多维分析场景下，用户更关注的是如何保证 Key 的唯一性，即如何获得 Primary Key 唯一性约束。因此，我们引入了 Uniq 的数据模型。该模型本质上是聚合模型的一个特例，也是一种简化的表结构表示方式。我们举例说明。

|               |              |       |              |
| ------------- | ------------ | ----- | ------------ |
| ColumnName    | Type         | IsKey | Comment      |
| user_id       | BIGINT       | Yes   | 用户id       |
| username      | VARCHAR(50)  | Yes   | 用户昵称     |
| city          | VARCHAR(20)  | No    | 用户所在城市 |
| age           | SMALLINT     | No    | 用户年龄     |
| sex           | TINYINT      | No    | 用户性别     |
| phone         | LARGEINT     | No    | 用户电话     |
| address       | VARCHAR(500) | No    | 用户住址     |
| register_time | DATETIME     | No    |              |

这是一个典型的用户基础信息表。这类数据没有聚合需求，只需保证主键唯一性。（这里的主键为 user_id + username）。那么我们的建表语句如下：

```text
CREATE TABLE IF NOT EXISTS example_db.expamle_tbl
(
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `username` VARCHAR(50) NOT NULL COMMENT "用户昵称",
    `city` VARCHAR(20) COMMENT "用户所在城市",
    `age` SMALLINT COMMENT "用户年龄",
    `sex` TINYINT COMMENT "用户性别",
    `phone` LARGEINT COMMENT "用户电话",
    `address` VARCHAR(500) COMMENT "用户地址",
    `register_time` DATETIME COMMENT "用户注册时间"
)
UNIQUE KEY(`user_id`, `username`)
... /* 省略 Partition 和 Distribution 信息 */
；
```

而这个表结构，完全同等于以下使用聚合模型描述的表结构：



| ColumnName    | Type         | AggregationType | Comment      |
| ------------- | ------------ | --------------- | ------------ |
| user_id       | BIGINT       |                 | 用户id       |
| username      | VARCHAR(50)  |                 | 用户昵称     |
| city          | VARCHAR(20)  | REPLACE         | 用户所在城市 |
| age           | SMALLINT     | REPLACE         | 用户年龄     |
| sex           | TINYINT      | REPLACE         | 用户性别     |
| phone         | LARGEINT     | REPLACE         | 用户电话     |
| address       | VARCHAR(500) | REPLACE         | 用户住址     |
| register_time | DATETIME     | REPLACE         |              |

及建表语句：

```text
CREATE TABLE IF NOT EXISTS example_db.expamle_tbl
(
    `user_id` LARGEINT NOT NULL COMMENT "用户id",
    `username` VARCHAR(50) NOT NULL COMMENT "用户昵称",
    `city` VARCHAR(20) REPLACE COMMENT "用户所在城市",
    `age` SMALLINT REPLACE COMMENT "用户年龄",
    `sex` TINYINT REPLACE COMMENT "用户性别",
    `phone` LARGEINT REPLACE COMMENT "用户电话",
    `address` VARCHAR(500) REPLACE COMMENT "用户地址",
    `register_time` DATETIME REPLACE COMMENT "用户注册时间"
)
AGGREGATE KEY(`user_id`, `username`)
... /* 省略 Partition 和 Distribution 信息 */
；
```

即 Uniq 模型完全可以用聚合模型中的 REPLACE 方式替代。其内部的实现方式和数据存储方式也完全一样。这里不再继续举例说明。

