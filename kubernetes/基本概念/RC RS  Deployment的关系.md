**Docker可以看作一个进程。**

**Pod可以看成是一个计算机，上面运行着很多进程。**

**Replica Sets/replicationController 就是很多计算机组成的计算机集群。**

**Deployment就是控制一个集群整体升级，监控状态等，可以认为Deployment就是控制一个集群的变化状态，也就是部署。**

ReplicationController 用来确保容器应用的副本数始终保持在用户定义的副本数(当集群资源足够的情况下)，即如果容器异常退出，会重新自动创建新的Pod替代，同理，当有多余的Pod时，会自动清除。

ReplicaSet和ReplicationController没有本质的不同，只是名字不一样，并且ReplicaSet支持集合式的selector。

虽然ReplicaSet可以独立使用，但一般还是建议使用Deloyment来自动管理ReplicaSet，这样就无需担心跟其他机制的不兼容问题。（比如ReplicaSet不支持rolling-update，但Deployment支持。）同时，Deployment不能独立的创建pod，所以需要通过管理replicaSet来创建Pod。





`Deployment`继承了rc的全部功能外，还可以查看升级详细进度和状态，当升级出现问题的时候，可以使用回滚操作回滚到指定的版本，每一次对Deployment的操作，都会保存下来，变能方便的进行回滚操作了，另外对于每一次升级都可以随时暂停和启动，拥有多种升级方案：`Recreate`删除现在的`Pod`，重新创建；`RollingUpdate`滚动升级，逐步替换现有`Pod`，对于生产环境的服务升级，显然这是一种最好的方式。



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210720150146477.png" alt="image-20210720150146477" style="zoom:50%;" />





可以看出一个Deployment拥有多个Replica Set，而一个Replica Set拥有一个或多个Pod。

一个Deployment控制多个rs主要是为了支持回滚机制，每当Deployment操作时，Kubernetes会重新生成一个Replica Set并保留，以后有需要的话就可以回滚至之前的状态。 

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210720150352815.png" alt="image-20210720150352815" style="zoom:50%;" />



