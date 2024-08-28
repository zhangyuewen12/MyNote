# SparkContext的parallelize的参数

1.解释

- 并行集合的创建（RDD）  
使用已经存在的迭代器或者集合通过调用spark驱动程序提供的parallelize函数来创建并行集合  
- 并行集合被创建用来在分布式集群上并行计算的。

2.例子  
data = [1, 2, 3, 4, 5]  
distData = sc.parallelize(data)   

一旦创建RDD，RDD，就可以在集群上并行的去被操作。我们可以调用distData.reduce(lambda a, b:a + b)添加元素到list。之后在RDD上进行一些操作或者行动.


3.parallelize的一个重要的参数  
就是分区数量。  
就是将RDD切分多少个分区。  
这个分区数目每个CPU一般是2-4个在你的集群上。通常，spark会自动设置这个数量在你的集群上。你也可以手动去传参，这个函数的第二个参数，比如`sc.parallelize(data, 5)