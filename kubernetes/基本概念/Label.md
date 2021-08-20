**使用Label可以给对象创建多组标签，Label和Label Selector 共同构成了Kubernates系统中核心的应用模型，使得被管理对象能够被精细地分组管理，同时实现了整个集群的高可用性。**

**一：什么是Label**

Label是Kubernetes系列中另外一个核心概念。是一组绑定到K8s资源对象上的key/value对。同一个对象的labels属性的key必须唯一。label可以附加到各种资源对象上，如Node,Pod,Service,RC等。

通过给指定的资源对象捆绑一个或多个不用的label来实现多维度的资源分组管理功能，以便于灵活，方便地进行资源分配，调度，配置，部署等管理工作。

示例如下：　

- 版本标签："release" : "stable" , "release" : "canary"...
- 环境标签："environment" : "dev" , "environment" : "production"
- 架构标签："tier" : "frontend" , "tier" : "backend" , "tier" : "middleware"
- 分区标签："partition" : "customerA" , "partition" : "customerB"...
- 质量管控标签："track" : "daily" , "track" : "weekly"



**二：什么是Label selector(标签选择器)**


Label selector是Kubernetes核心的分组机制，通过label selector客户端/用户能够识别一组有共同特征或属性的资源对象。



**三:Label selector的查询条件**

基于值相等的查询条件： 类似于ＳＱＬ语句中的＝或！＝；　　例如：select * from pod where name=(或!=)'redis-slave';

基于子集的查询条件: 类似于SQL语句中的in或 not in； 例如：select * from pod where name in(或not in) ('redis-slave','redis-master');

两种查询条件也可以组合在一起使用



**四:Label selector的使用场景
**
1.kube-controller进程通过**资源对象RC**上定义的Label Selector来筛选要监控的Pod副本的数量，从而实现Pod副本的数量始终符合预期设定的全自动控制流程

2.kupe-proxy进程通过**Service**的Label Selector来选择对应的Pod，自动建立器每个Service到对应Pod的请求转发路由表，从而实现Service的智能负载均衡机制

3.通过对某些Node定义特定的Label,并且在**Pod定义文件**中使用NodeSelector这种标签调度策略，Kube-scheduler进程可以实现Pod定向调度的特性