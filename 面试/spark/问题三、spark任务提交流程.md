# 介绍一下Spark的任务提交流程

#### Yarn 的client运行模式

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230822105845234.png" alt="image-20230822105845234" style="zoom:50%;" />

> 解析：
> 在 YARN Client 模式下，Driver 在任务提交的本地机器上运行，Driver 启动后会ResourceManager 通讯申请启动 ApplicationMaster，随后 ResourceManager分 配 container ， 在 合 适 的NodeManager 上 启 动 ApplicationMaster ， 此 时 的ApplicationMaster 的功能相当于一个 ExecutorLaucher，只负责向 ResourceManager申请 Executor 内存。ResourceManager 接到 ApplicationMaster 的资源申请后会分配 container，然后ApplicationMaster 在资源分配指定的 NodeManager 上启动 Executor 进程，Executor进程启动后会向 Driver 反向注册，Executor 全部注册完成后 Driver 开始执行 main函数，之后执行到 Action 算子时，触发一个 job，并根据宽依赖开始划分 stage，每个 stage 生成对应的 taskSet，之后将 task 分发到各个 Executor 上执行。



#### Yarn cluster运行模式

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230822110127777.png" alt="image-20230822110127777" style="zoom:50%;" />

> 解析：
> 在 YARN Cluster 模式下，任务提交后会和 ResourceManager 通讯申请启动ApplicationMaster，随后 ResourceManager 分配 container，在合适的 NodeManager上启动 ApplicationMaster，此时的 ApplicationMaster 就是 Driver。Driver 启动后向 ResourceManager 申请 Executor 内存，ResourceManager 接到ApplicationMaster 的资源申请后会分配 container，然后在合适的 NodeManager 上启动 Executor 进程，Executor 进程启动后会向 Driver 反向注册，Executor 全部注册完成后 Driver 开始执行 main 函数，之后执行到 Action 算子时，触发一个 job，并根据宽依赖开始划分 stage，每个 stage 生成对应的 taskSet，之后将 task 分发到各个Executor 上执行。

