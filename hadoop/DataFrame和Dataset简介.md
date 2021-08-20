# 一、Spark SQL简介
Spark SQL 是 Spark 中的一个子模块，主要用于操作结构化数据。它具有以下特点：

* 能够将 SQL 查询与 Spark 程序无缝混合，允许您使用 SQL 或 DataFrame API 对结构化数据进行查询；
* 支持多种开发语言；
* 支持多达上百种的外部数据源，包括 Hive，Avro，Parquet，ORC，JSON 和 JDBC 等；
* 支持 HiveQL 语法以及 Hive SerDes 和 UDF，允许你访问现有的 Hive 仓库；
* 支持标准的 JDBC 和 ODBC 连接；
* 支持优化器，列式存储和代码生成等特性；
* 支持扩展并能保证容错。

![](https://camo.githubusercontent.com/ce2cf78097f3d01ad04da9b2725e2ddc48ad0f361d3773c9adfc1e78143ac09e/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f73716c2d686976652d617263682e706e67)

# 二、DataFrame & DataSet
## 2.1 DataFrame
为了支持结构化数据的处理，Spark SQL 提供了新的数据结构 DataFrame。DataFrame 是一个由具名列组成的数据集。它在概念上等同于关系数据库中的表或 R/Python 语言中的 data frame。 由于 Spark SQL 支持多种语言的开发，所以每种语言都定义了 DataFrame 的抽象.
## 2.2 DataFrame 对比 RDDs
![](https://camo.githubusercontent.com/fd5e0bdcc317cc5f7326add814cc9e47a563c16f9d28e0f24a8f0a1b8104239d/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f737061726b2d646174614672616d652b524444732e706e67)
DataFrame 内部的有明确 Scheme 结构，即列名、列字段类型都是已知的，这带来的好处是可以减少数据读取以及更好地优化执行计划，从而保证查询效率。

DataFrame 和 RDDs 应该如何选择？

* 如果你想使用函数式编程而不是 DataFrame API，则使用 RDDs；
* 如果你的数据是非结构化的 (比如流媒体或者字符流)，则使用 RDDs，
* 如果你的数据是结构化的 (如 RDBMS 中的数据) 或者半结构化的 (如日志)，出于性能上的考虑，应优先使用 DataFrame。

## 2.3 DataSet
Dataset 也是分布式的数据集合，在 Spark 1.6 版本被引入，它集成了 RDD 和 DataFrame 的优点，具备强类型的特点，同时支持 Lambda 函数，但只能在 Scala 和 Java 语言中使用。在 Spark 2.0 后，为了方便开发者，Spark 将 DataFrame 和 Dataset 的 API 融合到一起，提供了结构化的 API(Structured API)，即用户可以通过一套标准的 API 就能完成对两者的操作。

_*这里注意一下：DataFrame 被标记为 Untyped API，而 DataSet 被标记为 Typed API，后文会对两者做出解释。*_
![](https://camo.githubusercontent.com/9491f8ec3aeeac301f6d3019e1717670e795adff3a768075d7ed31173b5e298c/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f737061726b2d756e696665642e706e67)

下面的给出一个 IDEA 中代码编译的示例：
![](https://camo.githubusercontent.com/b3dd7579837de2088c2e4f94dc4f5a2c3a44ccf91d23d9a9840b92e32e519b8b/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f737061726b2de8bf90e8a18ce697b6e7b1bbe59e8be5ae89e585a82e706e67)
这里一个可能的疑惑是 DataFrame 明明是有确定的 Scheme 结构 (即列名、列字段类型都是已知的)，但是为什么还是无法对列名进行推断和错误判断，这是因为 DataFrame 是 Untyped 的。

## 2.5 Untyped & Typed
在上面我们介绍过 DataFrame API 被标记为 Untyped API，而 DataSet API 被标记为 Typed API。DataFrame 的 Untyped 是相对于语言或 API 层面而言，它确实有明确的 Scheme 结构，即列名，列类型都是确定的，但这些信息完全由 Spark 来维护，Spark 只会在运行时检查这些类型和指定类型是否一致。这也就是为什么在 Spark 2.0 之后，官方推荐把 DataFrame 看做是 DatSet[Row]，Row 是 Spark 中定义的一个 trait，其子类中封装了列字段的信息。

相对而言，DataSet 是 Typed 的，即强类型。如下面代码，DataSet 的类型由 Case Class(Scala) 或者 Java Bean(Java) 来明确指定的，在这里即每一行数据代表一个 Person，这些信息由 JVM 来保证正确性，所以字段名错误和类型错误在编译的时候就会被 IDE 所发现。

```
case class Person(name: String, age: Long)
val dataSet: Dataset[Person] = spark.read.json("people.json").as[Person]
```

## 三、DataFrame & DataSet & RDDs 总结
这里对三者做一下简单的总结：

* RDDs 适合非结构化数据的处理，而 DataFrame & DataSet 更适合结构化数据和半结构化的处理；
* DataFrame & DataSet 可以通过统一的 Structured API 进行访问，而 RDDs 则更适合函数式编程的场景；
* 相比于 DataFrame 而言，DataSet 是强类型的 (Typed)，有着更为严格的静态类型检查；
* DataSets、DataFrames、SQL 的底层都依赖了 RDDs API，并对外提供结构化的访问接口。

![](https://camo.githubusercontent.com/a6b74c74abae230af70feda1495e3ac1065328b7c34c2f9fb2918a17000fd28b/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f737061726b2d7374727563747572652d6170692e706e67)
