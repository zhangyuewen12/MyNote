# RC（Replication Controller）

RC定义了一个期望的场景，即声明某种Pod的副本数量在任意时刻都符合某个预期值，RC的定义包括如下：

1）Pod期望的副本数

2）用于筛选目标Pod的Label Selector

3）当Pod的副本数量小于预期的时候，用于创建新的Pod的Pod模板



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210719160932647.png" alt="image-20210719160932647" style="zoom:50%;" />

当我们定义了一个RC并提交到Kubernetes集群中以后，Master节点上的Controller Manager组件就得到通知，定期巡检系统中当前存活的目标Pod，并确保目标Pod实例的数量刚好等于此RC的期望值，如果有过多的Pod副本在运行，系统就会停掉一些Pod，否则系统就会再自动创建一些Pod。通过RC，Kubernetes实现了用户应用集群的高可用性，并大大减少了系统管理员在传统IT环境中需要完成的手工运维工作（如主机监控脚本，应用监控脚本，故障恢复脚本）。



# RC的特性与作用

1）在大多数情况下，我们通过定义一个RC实现Pod的创建过程以及副本数量的自动控制。

2）RC里包括完整的Pod定义模板

3）RC里包括完整的Label Selector机制实现对Pod副本的自动控制

4）通过改变RC里的Pod副本数量，可以实现Pod的扩容和缩容功能

5）通过改变RC里的Pod模板中的镜像版本，可以实现Pod的滚动升级功能 