## kafka follower 如何与 leader 同步数据?

Kafka 的复制机制既不是完全的同步复制，也不是单纯的异步复制。

完全同步复制要求 All Alive Follower 都复制完，这条消息才会被认为 commit，这种复制方式极大的影响了吞吐率。

 而异步复制方式下，Follower 异步的从 Leader 复制数据，数据只要被 Leader 写入 log 就被认 为已经 commit，这种情况下，如果 leader 挂掉，会丢失数据。

kafka 使用 ISR 的方式很好的 均衡了确保数据不丢失以及吞吐率。Follower 可以批量的从 Leader 复制数据，而且 Leader 充分利用磁盘顺序读以及 send file(zero copy)机制，这样极大的提高复制性能，内部批量写 磁盘，大幅减少了 Follower 与 Leader 的消息量差

## 说一说消费者负载均衡策略?

一个 kafka 分区只能被一个消费者消费，一个消费者可以消费多个 kafka 分区。
 当 kafka 分区数跟消费者组中的消费者数量一致，一个消费者对应一个分区。
 当 kafka 分区数大于消费者组中的消费者数量，一个消费者对应多个分区。
当 kafka 分区数小于消费者组中的消费者数量，一个消费者对应一个分区，此时消费者会有 空闲。

