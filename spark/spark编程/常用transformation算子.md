## RDD编程模型常用transformation算子

transformation包括

（1）map 

```
map函数方法参数：

/**
   * Return a new RDD by applying a function to all elements of this RDD.
   */
  def map[U: ClassTag](f: T => U): RDD[U]
//使用示例

scala> val rdd1=sc.parallelize(Array(1,2,3,4)).map(x=>2*x).collect
rdd1: Array[Int] = Array(2, 4, 6, 8)
```

（2）filter 

```
方法参数：

/**
   * Return a new RDD containing only the elements that satisfy a predicate.
   */
  def filter(f: T => Boolean): RDD[T]
  
使用示例

scala> val rdd1=sc.parallelize(Array(1,2,3,4)).filter(x=>x>1).collect
rdd1: Array[Int] = Array(2, 3, 4)
```

（3）flatMap 

```
方法参数：

/**
   *  Return a new RDD by first applying a function to all elements of this
   *  RDD, and then flattening the results.
   */
  def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U] 


使用示例：

scala>  val data =Array(Array(1, 2, 3, 4, 5),Array(4,5,6))
data: Array[Array[Int]] = Array(Array(1, 2, 3, 4, 5), Array(4, 5, 6))

scala> val rdd1=sc.parallelize(data)
rdd1: org.apache.spark.rdd.RDD[Array[Int]] = ParallelCollectionRDD[2] at parallelize at <console>:23

scala> val rdd2=rdd1.flatMap(x=>x.map(y=>y))
rdd2: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[3] at flatMap at <console>:25

scala> rdd2.collect
res0: Array[Int] = Array(1, 2, 3, 4, 5, 4, 5, 6)

```

(4)	mapPartitions
mapPartitions是map的一个变种。map的输入函数是应用于RDD中每个元素，而mapPartitions的输入函数是应用于每个分区，也就是把每个分区中的内容作为整体来处理的。 
它的函数定义为：

```
def mapPartitions[U: ClassTag](f: Iterator[T] => Iterator[U], preservesPartitioning: Boolean = false): RDD[U]
```

f即为输入函数，它处理每个分区里面的内容。每个分区中的内容将以Iterator[T]传递给输入函数f，f的输出结果是Iterator[U]。最终的RDD由所有分区经过输入函数处理后的结果合并起来的。

举例：

```
scala> val a = sc.parallelize(1 to 9, 3)
scala> def myfunc[T](iter: Iterator[T]) : Iterator[(T, T)] = {
    var res = List[(T, T)]() 
    var pre = iter.next while (iter.hasNext) {
        val cur = iter.next; 
        res .::= (pre, cur) pre = cur;
    } 
    res.iterator
}
scala> a.mapPartitions(myfunc).collect
res0: Array[(Int, Int)] = Array((2,3), (1,2), (5,6), (4,5), (8,9), (7,8))
```

上述例子中的函数myfunc是把分区中一个元素和它的下一个元素组成一个Tuple。因为分区中最后一个元素没有下一个元素了，所以(3,4)和(6,7)不在结果中。 
mapPartitions还有些变种，比如mapPartitionsWithContext，它能把处理过程中的一些状态信息传递给用户指定的输入函数。还有mapPartitionsWithIndex，它能把分区的index传递给用户指定的输入函数。

（5）union 
union将两个RDD数据集元素合并，类似两个集合的并集 
union函数参数：

```
/**
   * Return the union of this RDD and another one. Any identical elements will appear multiple
   * times (use `.distinct()` to eliminate them).
   */
  def union(other: RDD[T]): RDD[T] 
```

RDD与另外一个RDD进行Union操作之后，两个数据集中的存在的重复元素
 
```
scala> val rdd1=sc.parallelize(1 to 5)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[15] at parallelize at <console>:21

scala> val rdd2=sc.parallelize(4 to 8)
rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[16] at parallelize at <console>:21
//存在重复元素
scala> rdd1.union(rdd2).collect
res13: Array[Int] = Array(1, 2, 3, 4, 5, 4, 5, 6, 7, 8)
```

（6）intersection 

方法返回两个RDD数据集的交集 

```
函数参数： 
/** 
* Return the intersection of this RDD and another one. The output will not contain any duplicate 
* elements, even if the input RDDs did. 
* 
* Note that this method performs a shuffle internally. 
*/ 
def intersection(other: RDD[T]): RDD[T]

使用示例：

scala> rdd1.intersection(rdd2).collect
res14: Array[Int] = Array(4, 5)
```

（7）distinct 

```
distinct函数参数：

/** 
* Return a new RDD containing the distinct elements in this RDD. 
*/ 
def distinct(): RDD[T]

scala> val rdd1=sc.parallelize(1 to 5)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:21

scala> val rdd2=sc.parallelize(4 to 8)
rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[1] at parallelize at <console>:21

scala> rdd1.union(rdd2).distinct.collect
res0: Array[Int] = Array(6, 1, 7, 8, 2, 3, 4, 5)
```

（8）groupByKey([numTasks]) 

输入数据为(K, V) 对, 返回的是 (K, Iterable) ，numTasks指定task数量，该参数是可选的，下面给出的是无参数的groupByKey方法 ,相当于sql中group by

```
/** 
* Group the values for each key in the RDD into a single sequence. Hash-partitions the 
* resulting RDD with the existing partitioner/parallelism level. The ordering of elements 
* within each group is not guaranteed, and may even differ each time the resulting RDD is 
* evaluated. 
* 
* Note: This operation may be very expensive. If you are grouping in order to perform an 
* aggregation (such as a sum or average) over each key, using [[PairRDDFunctions.aggregateByKey]] 
* or [[PairRDDFunctions.reduceByKey]] will provide much better performance. 
*/ 
def groupByKey(): RDD[(K, Iterable[V])]


scala> val rdd1=sc.parallelize(1 to 5)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:21

scala> val rdd2=sc.parallelize(4 to 8)
rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[1] at parallelize at <console>:21

scala> rdd1.union(rdd2).map((_,1)).groupByKey.collect
res2: Array[(Int, Iterable[Int])] = Array((6,CompactBuffer(1)), (1,CompactBuffer(1)), (7,CompactBuffer(1)), (8,CompactBuffer(1)), (2,CompactBuffer(1)), (3,CompactBuffer(1)), (4,CompactBuffer(1, 1)), (5,CompactBuffer(1, 1)))
```

1. reduceByKey(func, [numTasks]) 

```
reduceByKey函数输入数据为(K, V)对，返回的数据集结果也是（K,V）对，只不过V为经过聚合操作后的值 
/** 
* Merge the values for each key using an associative reduce function. This will also perform 
* the merging locally on each mapper before sending results to a reducer, similarly to a 
* “combiner” in MapReduce. Output will be hash-partitioned with numPartitions partitions. 
*/ 
def reduceByKey(func: (V, V) => V, numPartitions: Int): RDD[(K, V)]

使用示例：

scala> rdd1.union(rdd2).map((_,1)).reduceByKey(_+_).collect
res4: Array[(Int, Int)] = Array((6,1), (1,1), (7,1), (8,1), (2,1), (3,1), (4,2), (5,2))
```

2. sortByKey([ascending], [numTasks]) 
3. 
对输入的数据集按key排序 
sortByKey方法定义

```
/**
   * Sort the RDD by key, so that each partition contains a sorted range of the elements. Calling
   * `collect` or `save` on the resulting RDD will return or output an ordered list of records
   * (in the `save` case, they will be written to multiple `part-X` files in the filesystem, in
   * order of the keys).
   */
  // TODO: this currently doesn't work on P other than Tuple2!
  def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.length)
      : RDD[(K, V)]
      

使用示例：

scala> var data = sc.parallelize(List((1,3),(1,2),(1, 4),(2,3),(7,9),(2,4)))
data: org.apache.spark.rdd.RDD[(Int, Int)] = ParallelCollectionRDD[20] at parallelize at <console>:21

scala> data.sortByKey(true).collect
res10: Array[(Int, Int)] = Array((1,3), (1,2), (1,4), (2,3), (2,4), (7,9))
```
