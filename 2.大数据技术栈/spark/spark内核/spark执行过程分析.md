# spark执行过程分析

## 1.spark的action算子触发job执行。

开始调用sparkContext的runJob方法

```
def runJob[T, U: ClassTag]
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
   
在RDD中给定的一组分区上运行一个函数，并将结果传递给给定的处理函数。 
这是Spark中所有动作的主要切入点。
```
## 2.runJob方法调用dagScheduler.runjob方法
dagScheduler.runJob(rdd, cleanedFunc, partitions, callSite, resultHandler, localProperties.get)

```
dagScheduler.runjob方法作用

Run an action job on the given RDD and pass all the results to the resultHandler function as
   * they arrive.
   *
 运行一个给定RDD的action job 并 传递所有的结果给 resultHandler 方法作用输入。
 
  def runJob[T, U](
      rdd: RDD[T],
      func: (TaskContext, Iterator[T]) => U,
      partitions: Seq[Int],
      callSite: CallSite,
      resultHandler: (Int, U) => Unit,
      properties: Properties): Unit = {
    val start = System.nanoTime
    val waiter = submitJob(rdd, func, partitions, callSite, resultHandler, properties)
    ThreadUtils.awaitReady(waiter.completionFuture, Duration.Inf)
    waiter.completionFuture.value.get match {
      case scala.util.Success(_) =>
        logInfo("Job %d finished: %s, took %f s".format
          (waiter.jobId, callSite.shortForm, (System.nanoTime - start) / 1e9))
      case scala.util.Failure(exception) =>
        logInfo("Job %d failed: %s, took %f s".format
          (waiter.jobId, callSite.shortForm, (System.nanoTime - start) / 1e9))
        // SPARK-8644: Include user stack trace in exceptions coming from DAGScheduler.
        val callerStackTrace = Thread.currentThread().getStackTrace.tail
        exception.setStackTrace(exception.getStackTrace ++ callerStackTrace)
        throw exception
    }
  }
```

## 3.dagScheduler.runjob方法 调用 -》》val waiter = submitJob(rdd, func, partitions, callSite, resultHandler, properties)
```
submitJob 方法
作用：Submit an action job to the scheduler
内部调用：
val waiter = new JobWaiter(this, jobId, partitions.size, resultHandler)

eventProcessLoop.post(JobSubmitted(
      jobId, rdd, func2, partitions.toArray, callSite, waiter,
      SerializationUtils.clone(properties)))
      
waiter
```
submitJoob方法是将waiter放入到eventProcessLoop队列里。

将该获取到的Job(已有JobId），插入到**LinkedBlockingDeque结构**的事件处理队列中eventProcessLoop


private[scheduler] val eventProcessLoop = new DAGSchedulerEventProcessLoop(this)

// 在把
eventProcessLoop（LinkedBlockingDeque类型）放入新事件后，调起底层的DAGSchedulerEventProcessLoop.onReceive（），执行doOnReceive（）

## 4.DAGSchedulerEventProcessLoop.onReceive方法

```
/**
   * The main event loop of the DAG scheduler.
   */
  override def onReceive(event: DAGSchedulerEvent): Unit = {
    val timerContext = timer.time()
    try {
      doOnReceive(event)
    } finally {
      timerContext.stop()
    }
  }
```
onReceive方法调用doOnReceive方法

doOnReceive方法  
根据DAGSchedulerEvent的具体类型如JobSubmitted事件或者MapStageSubmitted事件，调取具体的Submitted handle函数提交具体的Job

例如  
    case JobSubmitted(jobId, rdd, func, partitions, callSite, listener, properties) =>
      dagScheduler.handleJobSubmitted(jobId, rdd, func, partitions, callSite, listener, properties)
## handleJobSubmitted 内部

```
finalStage = createResultStage(finalRDD, func, partitions, jobId, callSite)

建立stage  

val job = new ActiveJob(jobId, finalStage, callSite, listener, properties)  
同时开始逆向构建缺失的stage，getMissingParentStages(finalStage)

DAG构建完毕，提交stage，  
submitStage(finalStage)

```
## submitStage
```
if (missing.isEmpty) {
          logInfo("Submitting " + stage + " (" + stage.rdd + "), which has no missing parents")
          submitMissingTasks(stage, jobId.get)
        } else {
          for (parent <- missing) {
            submitStage(parent)
          }
          waitingStages += stage
        }
```
## 如果该stage是缺失的 ，那么运行方法：submitMissingTasks，该方法根据stage是shuffleMapStage或者ResultStage执行不同的stageStart()方法
```
stage match {
      case s: ShuffleMapStage =>
        outputCommitCoordinator.stageStart(stage = s.id, maxPartitionId = s.numPartitions - 1)
      case s: ResultStage =>
        outputCommitCoordinator.stageStart(
          stage = s.id, maxPartitionId = s.rdd.partitions.length - 1)
    }
```


