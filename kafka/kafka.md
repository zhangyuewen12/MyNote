# kafkad 的ISR 机制
kafka的ISR机制

kafka的ISR机制被成为“不丢消息”机制。在说ISR机制前，先讲一下kafka的副本（replica）。

kafka的Replica

1.kafka的topic可以设置有N个副本（replica），副本数最好要小于broker的数量，也就是要保证一个broker上的replica最多有一个，所以可以用broker id指定Partition replica。

2.创建副本的单位是topic的分区，每个分区有1个leader和0到多个follower，我们把多个replica分为Lerder replica和follower replica。

3.当producer在向partition中写数据时，根据ack机制，默认ack=1，只会向leader中写入数据，然后leader中的数据会复制到其他的replica中，follower会周期性的从leader中pull数据，但是对于数据的读写操作都在leader replica中，follower副本只是当leader副本挂了后才重新选取leader，follower并不向外提供服务。

kafka的“同步”

kafka不是完全同步，也不是完全异步，是一种特殊的ISR（In Sync Replica）

1.leader会维持一个与其保持同步的replica集合，该集合就是ISR，每一个partition都有一个ISR，它时有leader动态维护。

2.我们要保证kafka不丢失message，就要保证ISR这组集合存活（至少有一个存活），并且消息commit成功。

 

所以我们判定存活的概念时什么呢？分布式消息系统对一个节点是否存活有这样两个条件判断：第一个，节点必须维护和zookeeper的连接，zookeeper通过心跳机制检查每个节点的连接；第二个，如果节点时follower，它必要能及时同步与leader的写操作，不是延时太久。

如果满足上面2个条件，就可以说节点时“in-sync“（同步中的）。leader会追踪”同步中的“节点，如果有节点挂了，卡了，或延时太久，那么leader会它移除，延时的时间由参数replica.log.max.messages决定，判断是不是卡住了，由参数replica.log.time.max.ms决定。