# Spark中job、stage、task的划分+源码执行过程分析
![](https://img-blog.csdn.net/20170911221635721?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGp3MTk5MDg5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

job、stage、task  
* Worker Node：物理节点，上面执行executor进程  
* Executor：Worker Node为某应用启动的一个进程，执行多个tasks  
* Jobs:action 的触发会生成一个job, Job会提交给DAGScheduler,分解成Stage,  
* Stage:DAGScheduler 根据shuffle将job划分为不同的stage，同一个stage中包含多个task，这些tasks有相同的 shuffle dependencies。  
* Task:被送到executor上的工作单元，task简单的说就是在一个数据partition上的单个数据处理流程。