消息在真正发往 Kafka 之前，有可能需要经历[拦截器](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU3OTc1MDM1Mg%3D%3D%26mid%3D2247498434%26idx%3D2%26sn%3D89d9a7e60ca7c34af3794af5b5007b19%26chksm%3Dfd63ea7dca14636b1401eb9177be0297388e192f9cc7f7e5f3fc6101571a7da23382496fbbe1%26scene%3D21%23wechat_redirect&source=article&objectId=1787257)、[序列化器](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU3OTc1MDM1Mg%3D%3D%26mid%3D2247497846%26idx%3D1%26sn%3D39fefa0e590ed2bee8d01dcc086d4f9b%26chksm%3Dfd63e8c9ca1461df033b5078c93638692be3acb7533dedae16276c451415a4c17b5b59ec5ae5%26scene%3D21%23wechat_redirect&source=article&objectId=1787257)和[分区器](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU3OTc1MDM1Mg%3D%3D%26mid%3D2247498001%26idx%3D2%26sn%3Dc4c7abcb03424a42c80e5b53390951cd%26chksm%3Dfd63e9aeca1460b8c83823c9d3324a3df14df1b6a6d045acbce414fb59fd0597133dcc01ebd3%26scene%3D21%23wechat_redirect&source=article&objectId=1787257)等一系列的作用，前面已经做了一系列分析。那么在此之后又会发生什么呢？先看一下生产者客户端的整体架构，如下图所示。

![image-20240606103008062](/Users/zyw/Library/Application Support/typora-user-images/image-20240606103008062.png)

整个生产者客户端由两个线程协调运行，这两个线程分别为主线程和发送线程。在主线程中由 KafkaProducer 创建消息，然后通过可能的拦截器、序列化器和分区器的作用之后缓存到消息收集器（RecordAccumulator，也称为消息累加器）中。发送线程负责从消息收集器中获取消息并将其发送到 Kafka 中。

RecordAccumulator主要用来缓存消息以便发送线程可以批量发送，进而减少网络传输的资源消耗以提升性能。消息收集器缓存的大小可以通过生产者客户端参数 buffer.memory 配置，默认值为 33554432B，即32MB。如果生产者发送消息的速度超过发送到[服务器](https://cloud.tencent.com/act/pro/promotion-cvm?from_column=20065&from=20065)的速度，则会导致生产者空间不足，这个时候 KafkaProducer 的 send() 方法调用要么被阻塞，要么抛出异常，这个取决于参数 max.block.ms 的配置，此参数的默认值为60000，即60秒。

主线程中发送过来的消息都会被追加到消息收集器（RecordAccumulator）的某个双端队列（Deque）中，在RecordAccumulator的内部为每个分区都维护了一个双端队列，队列中的内容就是ProducerBatch，即 Deque。消息写入缓存时，追加到双端队列的尾部；Sender 读取消息时，从双端队列的头部读取。注意 ProducerBatch 不是 ProducerRecord，ProducerBatch 中可以包含一至多个 ProducerRecord。

通俗地说，ProducerRecord 是生产者中创建的消息，而 ProducerBatch 是指一个消息批次，ProducerRecord 会被包含在 ProducerBatch 中，这样可以使字节的使用更加紧凑。与此同时，将较小的 ProducerRecord 拼凑成一个较大的 ProducerBatch，也可以减少网络请求的次数以提升整体的吞吐量。

如果生产者客户端需要向很多分区发送消息，则可以将 buffer.memory 参数适当调大以增加整体的吞吐量。





消息在网络上都是以字节的形式传输的，在发送之前需要创建一块内存区域来保存对应的消息。在 Kafka 生产者客户端中，通过 java.io.ByteBuffer实现消息内存的创建和释放。不过频繁的创建和释放是比较耗费资源的，在 消息收集器的内部还有一个 BufferPool，它主要用来实现 ByteBuffer 的复用，以实现缓存的高效利用。不过 BufferPool 只针对特定大小的 ByteBuffer 进行管理，而其他大小的 ByteBuffer 不会缓存进 BufferPool 中，这个特定的大小由 batch.size 参数来指定，默认值为16384B，即16KB。我们可以适当地调大 batch.size 参数以便多缓存一些消息。

ProducerBatch 的大小和 batch.size 参数也有着密切的关系。当一条消息流入消息收集器时，会先寻找与消息分区所对应的双端队列（如果没有则新建），再从这个双端队列的尾部获取一个 ProducerBatch（如果没有则新建），查看 ProducerBatch 中是否还可以写入这个 ProducerRecord，如果可以则写入，如果不可以则需要创建一个新的 ProducerBatch。

在新建 ProducerBatch 时评估这条消息的大小是否超过 batch.size 参数的大小，如果不超过，那么就以 batch.size 参数的大小来创建 ProducerBatch，这样在使用完这段内存区域之后，可以通过 BufferPool 的管理来进行复用；如果超过，那么就以评估的大小来创建 ProducerBatch，这段内存区域不会被复用。

Sender 从 RecordAccumulator 中获取缓存的消息之后，会进一步将原本<分区, Deque< ProducerBatch>> 的保存形式转变成 <Node, List< ProducerBatch> 的形式，其中 Node 表示 Kafka 集群的 broker 节点。

对于网络连接来说，生产者客户端是与具体的 broker 节点建立的连接，也就是向具体的 broker 节点发送消息，而并不关心消息属于哪一个分区；而对于 KafkaProducer 的应用逻辑而言，我们只关注向哪个分区中发送哪些消息，所以在这里需要做一个应用逻辑层面到网络I/O层面的转换。

在转换成 <Node, List> 的形式之后，Sender 还会进一步封装成 <Node, Request> 的形式，这样就可以将 Request 请求发往各个 Node 了，这里的 Request 是指 Kafka 的各种协议请求，对于消息发送而言就是指具体的 ProduceReques

请求在从 Sender 线程发往 Kafka 之前还会保存到 InFlightRequests 中，保存对象的具体形式为 Map<NodeId, Deque>，它的主要作用是缓存了已经发出去但还没有收到响应的请求（NodeId 是一个 String 类型，表示节点的 id 编号）。

与此同时，InFlightRequests 还提供了许多管理类的方法，并且通过配置参数还可以限制每个连接（也就是客户端与 Node 之间的连接）最多缓存的请求数。这个配置参数为 max.in.flight.requests.per.connection，默认值为5，即每个连接最多只能缓存5个未响应的请求，超过该数值之后就不能再向这个连接发送更多的请求了，除非有缓存的请求收到了响应（Response）。通过比较 Deque 的 size 与这个参数的大小来判断对应的 Node 中是否已经堆积了很多未响应的消息，如果真是如此，那么说明这个 Node 节点负载较大或网络连接有问题，再继续向其发送请求会增大请求超时的可能。