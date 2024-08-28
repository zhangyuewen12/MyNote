https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSCommands.html#fsck

hadoop fsck详解
我们知道fsck是用来检测hdfs上文件、block信息的，但是fsck输出的结果我们是否能看明白呢？

```
下面我们来看一个fsck输出的结果
hadoop fsck /
########################## 情况一 ####################
Status: HEALTHY
Total size: 3107919020687 B
Total dirs: 142
Total files: 321
Total symlinks: 0
Total blocks (validated): 11738 (avg. block size 264774154 B)
Minimally replicated blocks: 11738 (100.0 %)
Over-replicated blocks: 3605 (30.712217 %)
Under-replicated blocks: 0 (0.0 %)
Mis-replicated blocks: 8011 (68.24842 %)
Default replication factor: 3
Average block replication: 3.3083148
Corrupt blocks: 0
Missing replicas: 0 (0.0 %)
Number of data-nodes: 11
Number of racks: 2
FSCK ended at Fri Nov 10 15:11:47 CST 2017 in 418 milliseconds
 
 
The filesystem under path '/' is HEALTHY
 
注：
这种情况是我们在原来的机架上扩增了一个不同机房的机架
由于hadoop对扩增机房的数据平衡策略是：
1）先拷贝一份数据到新增机架的机器上，然后再在原来机架上删除一份数据
2）所以这里的Over-replicated blocks会显示，是3605 。代表的是集群新增了3605 副本数，超过了默认的副本数
 
########################## 情况二 ####################
.....................Status: HEALTHY
Total size: 3130802412834 B
Total dirs: 143
Total files: 321
Total symlinks: 0 (Files currently being written: 1)
Total blocks (validated): 11824 (avg. block size 264783695 B)
Minimally replicated blocks: 11824 (100.0 %)
Over-replicated blocks: 0 (0.0 %)
Under-replicated blocks: 755 (6.385318 %)
Mis-replicated blocks: 0 (0.0 %)
Default replication factor: 3
Average block replication: 2.937331
Corrupt blocks: 0
Missing replicas: 755 (2.1275997 %)
Number of data-nodes: 10
Number of racks: 1
FSCK ended at Mon Nov 13 16:59:13 CST 2017 in 69 milliseconds
 
注：
当新增机房的slave节点和之前的slave节点数据重新平衡后，我把新增机房的节点网络中断，
然后后就出现了 Under-replicated blocks，意思就是集群中有这么多副本数是小于集群指定的副本数。
 
 
 
########################## 情况三 ####################
Status: HEALTHY
Total size: 3130802412834 B
Total dirs: 143
Total files: 322
Total symlinks: 0
Total blocks (validated): 11824 (avg. block size 264783695 B)
Minimally replicated blocks: 11824 (100.0 %)
Over-replicated blocks: 0 (0.0 %)
Under-replicated blocks: 0 (0.0 %)
Mis-replicated blocks: 0 (0.0 %)
Default replication factor: 3
Average block replication: 3.001184
Corrupt blocks: 0
Missing replicas: 0 (0.0 %)
Number of data-nodes: 14
Number of racks: 2
FSCK ended at Mon Nov 13 11:00:37 CST 2017 in 642 milliseconds
 
注：
这里的是最终达到平衡后的检测结果

```


status：代表这次hdfs上block检测的结果
Total size: 代表/目录下文件总大小
Total dirs：代表检测的目录下总共有多少个目录
Total files：代表检测的目录下总共有多少文件
Total symlinks：代表检测的目录下有多少个符号连接
Total blocks(validated)：代表检测的目录下有多少个block块是有效的
Minimally replicated blocks：代表拷贝的最小block块数
Over-replicated blocks：指的是副本数大于指定副本数的block数量
Under-replicated blocks：指的是副本数小于指定副本数的block数量
Mis-replicated blocks：指丢失的block块数量
Default replication factor: 3 指默认的副本数是3份（自身一份，需要拷贝两份）
Missing replicas：丢失的副本数
Number of data-nodes：有多少个节点
Number of racks：有多少个机架