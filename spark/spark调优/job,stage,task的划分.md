Job的划分

1、Application :

　　应用，创建一个SparkContext可以认为创建了一个Application

2、Job

　　在一个app中每执行一次行动算子 就会创建一个Job,一个application会有多个job

3、stage

　　阶段，每碰到一个shuffle算子，会产生一个新的stage，一个Job中可以包含多个stage

4、task

　　任务，表示阶段执行的时候的并行度，一个stage会有多个task