# spark架构
##部署图
![](https://github.com/JerryLead/SparkInternals/raw/master/markdown/PNGfigures/deploy.png)


接下来分阶段讨论并细化这个图  

下图展示了driver program（假设在 master node 上运行）如何生成 job，并提交到 worker node 上执行
![](https://github.com/JerryLead/SparkInternals/raw/master/markdown/PNGfigures/JobSubmission.png)

Driver 端的逻辑如果用代码表示：    

```
finalRDD.action()
=> sc.runJob()
	sparkContext的runjob方法，接收三个参数，
	一个是目标RDD,需要运行tasks的RDD
	一个是分区，并不是所有的job都是在整个分区上进行的，例如first()操作
	一个是函数，这个函数运行在RDD上的每个分区。
	执行该步骤后，调用DAGScheduler类的runJob方法（）
/**
   * Run a function on a given set of partitions in an RDD and pass the results to the given
   * handler function. This is the main entry point for all actions in Spark.
   *
   * @param rdd target RDD to run tasks on
   * @param func a function to run on each partition of the RDD
   * @param partitions set of partitions to run on; some jobs may not want to compute on all
   * partitions of the target RDD, e.g. for operations like `first()`
   * @param resultHandler callback to pass each result to
   */
   
// generate job, stages and tasks
=> dagScheduler.runJob()
	DAGSchedulaer类的runJob()方法，接收参数
	1. 目标RDD
	2. 目标RDD需要在每个分区上运行的函数
	3. 分区的集合
	4. 。。
	该方法里面主要是调用submitJob()方法
/**
   * Run an action job on the given RDD and pass all the results to the resultHandler function as
   * they arrive.
   *
   * @param rdd target RDD to run tasks on
   * @param func a function to run on each partition of the RDD
   * @param partitions set of partitions to run on; some jobs may not want to compute on all
   *   partitions of the target RDD, e.g. for operations like first()
   * @param callSite where in the user program this job was called
   * @param resultHandler callback to pass each result to
   * @param properties scheduler properties to attach to this job, e.g. fair scheduler pool name
   *
   * @note Throws `Exception` when the job fails
   */
=> dagScheduler.submitJob()
	submit方法主要是在调度器中提交一个job。
	在该方法中，
	首先判断每个Partition都是存在的，并且partitions大于1.
	然后新建一个Jobwaiter对象，
	然后，创建一个JobSubmitted对象。JobSubmitted(
	      jobId, rdd, func2, partitions.toArray, callSite, waiter,
	      SerializationUtils.clone(properties))
	      JobSubmitted类继承了DAGSchedulerEvent
	 并把新建的jobSubmitted对象放到一个数组里，
	 eventProcessLoop.post(JobSubmitted(
	      jobId, rdd, func2, partitions.toArray, callSite, waiter,
	      SerializationUtils.clone(properties)))
	    waiter
	    
	    eventProcessLoop是一个DAGSchedulerEventProcessLoop类的实例对象。
	    在DAGSchedulerEventProcessLoop对象中主要调用doOnReceive方法，
	    在doOnReceive方法里主要
	    		根据提交的事件类型(DAGSchedulerEvent)，调用不同的DAGScheduler类型
	    		的handleJobSubmitted方法。 ----》
	    		
	    		
	最后，返回一个Waiter对象。
	回到DAGScheduler方法的runJob方法。
/**
   * Submit an action job to the scheduler.
   *
   * @param rdd target RDD to run tasks on
   * @param func a function to run on each partition of the RDD
   * @param partitions set of partitions to run on; some jobs may not want to compute on all
   *   partitions of the target RDD, e.g. for operations like first()
   * @param callSite where in the user program this job was called
   * @param resultHandler callback to pass each result to
   * @param properties scheduler properties to attach to this job, e.g. fair scheduler pool name
   *
   * @return a JobWaiter object that can be used to block until the job finishes executing
   *         or can be used to cancel the job.
   *
   * @throws IllegalArgumentException when partitions ids are illegal
   */
=>   dagSchedulerEventProcessActor ! JobSubmitted
=> dagSchedulerEventProcessActor.JobSubmitted()

=> dagScheduler.handleJobSubmitted()

	DAGScheduler类的handleJobSubmitted()方法，
	首先是会创建stage
	createResultStage
	finalStage = createResultStage(finalRDD, func, partitions, jobId, callSite)
	然后，会把创建的stage提交
	submitStage(finalStage)

=> finalStage = newStage()
=>   mapOutputTracker.registerShuffle(shuffleId, rdd.partitions.size)
=> dagScheduler.submitStage()
=>   missingStages = dagScheduler.getMissingParentStages()
=> dagScheduler.subMissingTasks(readyStage)

	在submitStage内部，是提交stage，但是会首先循环提交stage的父stage，
	调用 submitMissingTasks(stage, jobId.get)方法，提交确实的stage。

// add tasks to the taskScheduler
=> taskScheduler.submitTasks(new TaskSet(tasks))
=> fifoSchedulableBuilder.addTaskSetManager(taskSet)

// send tasks
=> sparkDeploySchedulerBackend.reviveOffers()
=> driverActor ! ReviveOffers
=> sparkDeploySchedulerBackend.makeOffers()
=> sparkDeploySchedulerBackend.launchTasks()
=> foreach task
      CoarseGrainedExecutorBackend(executorId) ! LaunchTask(serializedTask)
```

代码的文字描述：

当用户的 program 调用 val sc = new SparkContext(sparkConf) 时，这**个语句会帮助 program 启动诸多有关 driver 通信、job 执行的对象、线程、actor等，该语句确立了 program 的 driver 地位**。

## 生成 Job 逻辑执行图
Driver program 中的 transformation() 建立 computing chain（一系列的 RDD），每个 RDD 的 compute() 定义数据来了怎么计算得到该 RDD 中 partition 的结果，getDependencies() 定义 RDD 之间 partition 的数据依赖。

## 生成 Job 物理执行图
每个 action() 触发生成一个 job，在 dagScheduler.runJob() 的时候进行 stage 划分，在 submitStage() 的时候生成该 stage 包含的具体的 ShuffleMapTasks 或者 ResultTasks，然后将 tasks 打包成 TaskSet 交给 taskScheduler，如果 taskSet 可以运行就将 tasks 交给 sparkDeploySchedulerBackend 去分配执行。

## 分配 Task
sparkDeploySchedulerBackend 接收到 taskSet 后，会通过自带的 DriverActor 将 serialized tasks 发送到调度器指定的 worker node 上的 CoarseGrainedExecutorBackend Actor上。

# Job 接收
Worker 端接收到 tasks 后，执行如下操作  

```
coarseGrainedExecutorBackend ! LaunchTask(serializedTask)
=> executor.launchTask()
=> executor.threadPool.execute(new TaskRunner(taskId, serializedTask))
```
executor 将 task 包装成 taskRunner，并从线程池中抽取出一个空闲线程运行 task。一个 CoarseGrainedExecutorBackend 进程有且仅有一个 executor 对象。
![](https://github.com/JerryLead/SparkInternals/raw/master/markdown/PNGfigures/taskexecution.png)

Executor 收到 serialized 的 task 后，先 deserialize 出正常的 task，然后运行 task 得到其执行结果 directResult，这个结果要送回到 driver 那里。但是通过 Actor 发送的数据包不宜过大，如果 result 比较大（比如 groupByKey 的 result）先把 result 存放到本地的“内存＋磁盘”上，由 blockManager 来管理，只把存储位置信息（indirectResult）发送给 driver，driver 需要实际的 result 的时候，会通过 HTTP 去 fetch。如果 result 不大（小于spark.akka.frameSize = 10MB），那么直接发送给 driver。

上面的描述还有一些细节：如果 task 运行结束生成的 directResult > akka.frameSize，directResult 会被存放到由 blockManager 管理的本地“内存＋磁盘”上。**BlockManager 中的 memoryStore 开辟了一个 LinkedHashMap 来存储要存放到本地内存的数据。**LinkedHashMap 存储的数据总大小不超过 Runtime.getRuntime.maxMemory * spark.storage.memoryFraction(default 0.6) 。如果 LinkedHashMap 剩余空间不足以存放新来的数据，就将数据交给 diskStore 存放到磁盘上，但前提是该数据的 storageLevel 中包含“磁盘”。****