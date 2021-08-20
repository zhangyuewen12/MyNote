**Transformation 是 Flink操作的底层实现，无论是map还是Flatmap。**

DataStream类中包含两个变量：

- StreamExecutionEnvironment
- Transformation
  或者说DataStream类的所有操作都是围绕着两个变量进行。

每一次操作（map、flatmap等）都是在新建一个Transformation并将当前Transformation与下一个建立链接的关系。

Transformation中重要的变量

- id
- name
- parallelism
- outputType

重点说说PhysicalTransformation，它包含四个子类

- OneInputTransformation
- TwoInputTransformation
- SourceTransformation
- SinkTransformation

其中都包含变量StreamOperatorFactory用于记录操作的用户方法。除开SourceTransformation之外，其他三个中都包含Transformation input，表示上一级的Transformation。