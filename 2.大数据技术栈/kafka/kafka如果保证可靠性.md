对于 kafka 来说，以下几个方面来保障消息分发的可靠性: 

1.消息发送的可靠性保障(producer)

2.消息消费的可靠性保障(consumer)

3.Kafka 集群的可靠性保障(Broker)



从生产者角度

```
消息的发送有3中模式，即发后即忘，同步，异步。
对于发后即忘的模式不适合高可靠性要求的场景，如果要提高可靠性的话，那么生产者可以采用同步或异步的模式，在出现异常情况时，可以及时获得通知。
生产者的参数acks的控制：
acks=0
producer 不等待 broker 的 acks。发送的消息可能丢失，但永远不会重发。
acks=1
leader 不等待其他 follower 同步完毕，leader 直接写 log，然后发送 acks 给 producer。这种 情况下会有数据重发现象，可靠性比 only once 好点，但是仍然会丢消息。例如 leader 挂了， 但是其他 replication 还没完成同步。
acks=all
leader 等待所有 follower 同步完成才返回 acks。消息可靠不丢失(丢了会重发)，没收到 ack 会重发
```

从消费者角度

```
消费者
如果将 consumer 设置为 autocommit，consumer 一旦读到数据立即自动 commit。如果只讨论这一读取消息的过程，那 Kafka 确保了 Exactly once。

但实际使用中应用程序并非在 consumer 读取完数据就结束了，而是要进行进一步处理，而 数据处理与 commit 的顺序在很大程度上决定了 consumer delivery guarantee:

At most once:读完消息先 commit，再处理消息。

如果 consumer 在 commit 后还没来得及处理消息就 crash 了，下次重新开始工作后就无 法读到刚刚已提交而未处理的消息，因为此时 offset 已经改变了。

At least once(默认):读完消息先处理再 commit。

如果在处理完消息之后 ,commit 之前， consumer crash 了，下次重新开始工作时还会
处理刚刚未 commit 的消息，实际上该消息已经被处理过。但即使数据可能重复，这个在 应用上需要可以容忍的。

Exactly once(避免重复消费):保存offset和处理消息这两个环节采用two-phase commit(2PC)。

传统方式:把消息处理和提交 offset 做成原子操作。offset 提交失败，消息处理也要回滚。
kafka 推荐方式:消息处理结果和 offset 的存储放在一起存储。
比如将 offset 和处理结果一 起写到 HDFS，那就可以保证数据的输出和 offset 的更新要么都完成，要么都不完成，间接 实现 Exactly once。
```

从集群角度出发

```
对于 broker，落盘的数据，有 replica 机制保证数据不丢失。
就kafka而言，越多的副本数越能够保证数据的可靠性，但同时副本数越多，也会引起磁盘、网络带宽的浪费，同时会引起性能的下降。
```

