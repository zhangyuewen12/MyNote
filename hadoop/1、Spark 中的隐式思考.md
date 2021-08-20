#scala中的implicit

# 1、Spark 中的隐式思考
隐式转换是Scala的一大特性, 如果对其不是很了解, 在阅读Spark代码时候就会很迷糊,有人这样问过我？

```
RDD这个类没有reduceByKey,groupByKey等函数啊,并且RDD的子类也没有这些函数,但是好像PairRDDFunctions这个类里面好像有这些函数 为什么我可以在RDD调用这些函数呢?
```