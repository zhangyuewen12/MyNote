# Master

## 简介：

主节点。建议三台。

## 关键进程：

kubernates API server :提供了HTTP Rest接口的关键服务进程，是Kubernetes里所有资源的增删改查等操作的唯一入口，也是集群控制的入口地址。

 所有服务访问统一入口

Kubernetes Controller Mannger： Kubernetes里所有资源对象的自动化控制中心。

Kubernetes Scheduler: 负责资源(Pod)调度的进程。





# Node

## 简介：

工作节点，可以是一台具体的物理机，也可以是虚拟机。Node是kubernetes集群中的工作负载节点，每个Node都会被Master分配一些工作负载(Docker 容器)，当某个Node宕机时，其上的工作负载会被Master自动转移到其他节点上。

## 关键进程：

kubelet : 负责Pod对应容器的创建、启停等任务，同时与master密切协作，实现集群管理的基本功能。(直接跟容器引擎交互实现容器的生命周期管理。)

Kube-proxy:实现Kebernetes Services的通信与负载均衡机制的重要组件。（负责写入规则至 IPTABLES,IPVS 实现服务映射访问。）

Docker Engine：负责docker容器的创建和管理工作。



CoreDNS：可以为集群中SVC 创建一个域名IP的对应关系。

Dashboard  k8s提供一个B/S结构访问体系。





ETCD 键值对数据库，存储K8S集群的所有重要信息(持久化)；

