# Spark代码中设置appName在client模式和cluster模式中不一样问题
## 问题描述
![](https://support.huaweicloud.com/devg-mrs/zh-cn_image_0250982788.jpg)
**Spark应用名在使用yarn-cluster模式提交时不生效，在使用yarn-client模式提交时生效，**如图1所示，第一个应用是使用yarn-client模式提交的，正确显示我们代码里设置的应用名Spark Pi，第二个应用是使用yarn-cluster模式提交的，设置的应用名没有生效。

In general, configuration values explicitly set on a SparkConf take the highest precedence, then flags passed to spark-submit, then values in the defaults file.
通常情况下，SparkConf的配置拥有最高的优先级，然后是spark-submit中的参数，最后是默认配置文件的参数

## 回答
导致这个问题的主要原因是，yarn-client和yarn-cluster模式在提交任务时setAppName的执行顺序不同导致，yarn-client中setAppName是在向yarn注册Application之前读取，yarn-cluser模式则是在向yarn注册Application之后读取，这就导致yarn-cluster模式设置的应用名不生效。

## 解决措施：

**在spark-submit脚本提交任务时用--name设置应用名和sparkconf.setAppName(appname)里面的应用名一样。**

比如我们代码里设置的应用名为Spark Pi，用yarn-cluster模式提交应用时可以这样设置，在--name后面添加应用名，执行的命令如下：

./spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode cluster --name SparkPi lib/spark-examples*.jar 10