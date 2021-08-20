LEO&HW基本概念

Base Offset：是起始位移，该副本中第一条消息的offset，如下图，这里的起始位移是0，如果一个日志文件写满1G后（默认1G后会log rolling），这个起始位移就不是0开始了。  

HW（high watermark）：副本的高水印值，replica中leader副本和follower副本都会有这个值，通过它可以得知副本中已提交或已备份消息的范围，leader副本中的HW，决定了消费者能消费的最新消息能到哪个offset。如下图所示，HW值为8，代表offset为[0,8]的9条消息都可以被消费到，它们是对消费者可见的，而[9,12]这4条消息由于未提交，对消费者是不可见的。注意HW最多达到LEO值时，这时可见范围不会包含HW值对应的那条消息了，如下图如果HW也是13，则消费的消息范围就是[0,12]。  


LEO（log end offset）：日志末端位移，代表日志文件中下一条待写入消息的offset，这个offset上实际是没有消息的。不管是leader副本还是follower副本，都有这个值。当leader副本收到生产者的一条消息，LEO通常会自增1，而follower副本需要从leader副本fetch到数据后，才会增加它的LEO，最后leader副本会比较自己的LEO以及满足条件的follower副本上的LEO，选取两者中较小值作为新的HW，来更新自己的HW值。  
 
![](https://img2020.cnblogs.com/blog/1486105/202004/1486105-20200406121247606-1193043003.png)