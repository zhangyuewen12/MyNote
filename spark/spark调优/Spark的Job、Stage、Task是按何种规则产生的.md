# Spark的Job、Stage、Task是按何种规则产生的

![](https://img-blog.csdn.net/20180124230232622?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ2FvcHUxMjM0NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
**上面这张图就可以很清晰的说明这个问题。（图中最小的方块代表一个partition，包裹partition的方块是RDD，忽略颜色）**

## Job
Spark的Job来源于用户执行action操作，就是从RDD中获取结果的操作，而不是将一个RDD转换成另一个RDD的transformation操作。
## Stage
Spark的Stage是分割RDD执行的各种transformation而来。如上图，将这些转化步骤分为了3个Stage，分别为Stage1，Stage2和Stage3。这里最重要的是搞清楚分割Stage的规则，其实只有一个：从宽依赖处分割。
  
知道了这个分割规则，其实还是有一点疑惑，为什么这么分？  
其实道理蛮明显的，子RDD的partition会依赖父RDD中多个partition，这样就可能会有一些partition没有准备好，导致计算不能继续，所以就分开了，直到准备好了父RDD中所有partition，再继续进行将父RDD转换为子RDD的计算。而窄依赖完全不会有这个顾虑，窄依赖是父RDD一个partition对应子RDD一个partition，那么直接计算就可以了。

## Task
一个Stage内，最终的RDD有多少个partition，就会产生多少个task。看一看上图就明白了。
task是相同的代码块，只是操作不同partition中的数据。