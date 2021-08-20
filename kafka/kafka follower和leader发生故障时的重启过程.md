# kafka发生故障时
1. follower发生故障

follower发生故障时，会被临时提出ISR，待该follower恢复后，会读取本地磁盘记录的上次的HW，并将log文件中高出HW的部分截取掉，从HW向leader进行同步，等该follower的LEO大于或等于该Partition的HW，即follower的LEO达到该partition的HW时，就可以重新加入ISR了。

2. leader发生故障

leader发生故障，会从ISR中重新选择一个新的leader之后，为了多个副本之间的数据一致性，其余follower会将各自的log文件高于HW的部分截掉，然后从新的leader中同步数据。

**注意：这只能保证数据之间的一致性，并不能保证数据不丢失，或者不重复。**