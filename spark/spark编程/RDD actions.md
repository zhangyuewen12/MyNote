# RDD actions

本小节将介绍常用的action操作，前面使用的collect方法便是一种action，它返回RDD中所有的数据元素，方法定义如下：

```
/** 
* Return an array that contains all of the elements in this RDD. 
*/ 
def collect(): Array[T]
```
(1) reduce(func) 

reduce采样累加或关联操作减少RDD中元素的数量，其方法定义如下： 
/** 
* Reduces the elements of this RDD using the specified commutative and 
* associative binary operator. 
*/ 

```
def reduce(f: (T, T) => T): T 
使用示例：
scala> val data=sc.parallelize(1 to 9)
data: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[6] at parallelize at <console>:22

scala> data.reduce((x,y)=>x+y)
res12: Int = 45

scala> data.reduce(_+_)
res13: Int = 45
```

（2）count()

```
/** 
* Return the number of elements in the RDD. 
*/ 
def count(): Long
使用示例：

scala> val data=sc.parallelize(1 to 9)
data: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[6] at parallelize at <console>:22
scala> data.count
res14: Long = 9
```

（3）first() 

```
/** 
* Return the first element in this RDD. 
*/ 
def first()

scala> val data=sc.parallelize(1 to 9)
data: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[6] at parallelize at <console>:22
scala> data.first
res15: Int = 1
```


（4）take(n)

```


/** 
* Take the first num elements of the RDD. It works by first scanning one partition, and use the 
* results from that partition to estimate the number of additional partitions needed to satisfy 
* the limit. 
* 
* @note due to complications in the internal implementation, this method will raise 
* an exception if called on an RDD of Nothing or Null. 
*/ 
def take(num: Int): Array[T]

scala> val data=sc.parallelize(1 to 9)
data: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[6] at parallelize at <console>:22
scala> data.take(2)
res16: Array[Int] = Array(1, 2)
```