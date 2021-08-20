Pod

**pod是什么**：pod是k8s中基本的构建模块，一个pod可以包含多个和单个容器，包含多个容器时，这些容器总是运行在同一个工作节点上，因为一个pod绝不会跨多个工作节点。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210719150551466.png" alt="image-20210719150551466" style="zoom:50%;" />

**了解pod**：

pod将容器绑定在一起，并将它们作为一个单元进行管理。

在pod中，多个容器可以同时进行一些密切相关的进城，但又保持着一定的隔离。容器组内的容器共享一些资源，不是全部资源。

k8s通过配置docker让一个pod中的容器共享**相同的linux命名空间**，所以一个pod下的所有容器共享相同的主机名和网络接口。这些容器也都在相同的IPC命名空间下运行，能够通过IPC进行通信。

同一pod下的容器需要注意不能绑定到相同的端口号，否则会导致端口冲突。

每个pod有独立的端口空间，不同pod下的容器永远不会遇到端口冲突。一个pod中的所有容器都具有相同的loopback网络接口，容器可以通过localhost与同一pod的其他容器进行通信。

k8s中所有的pod都在同一个共享网络地址空间中，每个pod都有自己的IP，每个pod都可以通过其他pod的IP地址实现相互访问。无论pod是否在同一个工作节点上都能够相互通信。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210719150702439.png" alt="image-20210719150702439" style="zoom:50%;" />

**通过pod管理容器**：将多层应用分散到多个pod中，基于扩缩容考虑分割到多个pod中。

何时在pod中使用多个容器：将多个容器添加到单个pod的主要原因是应用可能由一个主要进程和一个活多个辅助进程组成的

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210719150719576.png" alt="image-20210719150719576" style="zoom:50%;" />

基本上，容器不应该包含多个进程，pod也不应该包含多个并不需要运行在同一台主机上的容器。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210719150740435.png" alt="image-20210719150740435" style="zoom:50%;" />