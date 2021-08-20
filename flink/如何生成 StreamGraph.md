# 如何生成 StreamGraph

资料链接 https://www.jianshu.com/p/413b8c96ccb4



生成StreamGraph的入口，StreamGraph()方法

```
	protected final List<StreamTransformation<?>> transformations = new ArrayList<>();
	
	/**
	 * Getter of the {@link org.apache.flink.streaming.api.graph.StreamGraph} of the streaming job.
	 *
	 * @return The streamgraph representing the transformations
	 */
	@Internal
	public StreamGraph getStreamGraph() {
		if (transformations.size() <= 0) {
			throw new IllegalStateException("No operators defined in streaming topology. Cannot execute.");
		}
		return StreamGraphGenerator.generate(this, transformations);
	}
```





## Transformation

`StreamGraphGenerator.generate` 的一个关键的参数是 `List<StreamTransformation<?>>`。

**StreamTransformation**`代表了从一个或多个DataStream生成新DataStream的操作。`



DataStream`的底层其实就是一个 `StreamTransformation`，描述了这个`DataStream是怎么来的。



DataStream 上常见的 transformation 有 map、flatmap、filter等）。这些transformation会构造出一棵 StreamTransformation 树，通过这棵树转换成 StreamGraph。

比如 `DataStream.map`源码如下，其中`SingleOutputStreamOperator`为DataStream的子类：

#### 首先自定义的MapFuction函数转换成StreamMap

```
public <R> SingleOutputStreamOperator<R> map(MapFunction<T, R> mapper) {
  // 通过java reflection抽出mapper的返回值类型
  TypeInformation<R> outType = TypeExtractor.getMapReturnTypes(clean(mapper), getType(),
      Utils.getCallLocationName(), true);

  // 返回一个新的DataStream，SteramMap 为 StreamOperator 的实现类
  return transform("Map", outType, new StreamMap<>(clean(mapper)));
}
```

#### 将`StreamMap`包装到`OneInputTransformation`，然后将`OneInputTransformation`封装成SingleOutputStreamOperator，最后该Operator存到env中

```
public <R> SingleOutputStreamOperator<R> transform(String operatorName, TypeInformation<R> outTypeInfo, OneInputStreamOperator<T, R> operator) {
  // read the output type of the input Transform to coax out errors about MissingTypeInfo
  transformation.getOutputType();

  // 新的transformation会连接上当前DataStream中的transformation，从而构建成一棵树
  OneInputTransformation<T, R> resultTransform = new OneInputTransformation<>(
      this.transformation,
      operatorName,
      operator,
      outTypeInfo,
      environment.getParallelism());

  @SuppressWarnings({ "unchecked", "rawtypes" })
  SingleOutputStreamOperator<R> returnStream = new SingleOutputStreamOperator(environment, resultTransform);

  // 所有的transformation都会存到 env 中，调用execute时遍历该list生成StreamGraph
  getExecutionEnvironment().addOperator(resultTransform);

  return returnStream;
}
```

## 生成 StreamGraph 的源码分析

我们通过在DataStream上做了一系列的转换（map、filter等）得到了`StreamTransformation`集合，然后通过`StreamGraphGenerator.generate`获得`StreamGraph`，该方法的源码如下：

```php
// 构造 StreamGraph 入口函数
public static StreamGraph generate(StreamExecutionEnvironment env, List<StreamTransformation<?>> transformations) {
    return new StreamGraphGenerator(env).generateInternal(transformations);
}

// 自底向上（sink->source）对转换树的每个transformation进行转换。
private StreamGraph generateInternal(List<StreamTransformation<?>> transformations) {
  for (StreamTransformation<?> transformation: transformations) {
    transform(transformation);
  }
  return streamGraph;
}

// 对具体的一个transformation进行转换，转换成 StreamGraph 中的 StreamNode 和 StreamEdge
// 返回值为该transform的id集合，通常大小为1个（除FeedbackTransformation）
private Collection<Integer> transform(StreamTransformation<?> transform) {  
  // 跳过已经转换过的transformation
  if (alreadyTransformed.containsKey(transform)) {
    return alreadyTransformed.get(transform);
  }

  LOG.debug("Transforming " + transform);

  // 为了触发 MissingTypeInfo 的异常
  transform.getOutputType();

  Collection<Integer> transformedIds;
  if (transform instanceof OneInputTransformation<?, ?>) {
    transformedIds = transformOnInputTransform((OneInputTransformation<?, ?>) transform);
  } else if (transform instanceof TwoInputTransformation<?, ?, ?>) {
    transformedIds = transformTwoInputTransform((TwoInputTransformation<?, ?, ?>) transform);
  } else if (transform instanceof SourceTransformation<?>) {
    transformedIds = transformSource((SourceTransformation<?>) transform);
  } else if (transform instanceof SinkTransformation<?>) {
    transformedIds = transformSink((SinkTransformation<?>) transform);
  } else if (transform instanceof UnionTransformation<?>) {
    transformedIds = transformUnion((UnionTransformation<?>) transform);
  } else if (transform instanceof SplitTransformation<?>) {
    transformedIds = transformSplit((SplitTransformation<?>) transform);
  } else if (transform instanceof SelectTransformation<?>) {
    transformedIds = transformSelect((SelectTransformation<?>) transform);
  } else if (transform instanceof FeedbackTransformation<?>) {
    transformedIds = transformFeedback((FeedbackTransformation<?>) transform);
  } else if (transform instanceof CoFeedbackTransformation<?>) {
    transformedIds = transformCoFeedback((CoFeedbackTransformation<?>) transform);
  } else if (transform instanceof PartitionTransformation<?>) {
    transformedIds = transformPartition((PartitionTransformation<?>) transform);
  } else {
    throw new IllegalStateException("Unknown transformation: " + transform);
  }

  // need this check because the iterate transformation adds itself before
  // transforming the feedback edges
  if (!alreadyTransformed.containsKey(transform)) {
    alreadyTransformed.put(transform, transformedIds);
  }

  if (transform.getBufferTimeout() > 0) {
    streamGraph.setBufferTimeout(transform.getId(), transform.getBufferTimeout());
  }
  if (transform.getUid() != null) {
    streamGraph.setTransformationId(transform.getId(), transform.getUid());
  }

  return transformedIds;
}
```

最终都会调用 transformXXX 来对具体的`StreamTransformation`进行转换。我们可以看下`transformOnInputTransform(transform)`的实现：

```kotlin
private <IN, OUT> Collection<Integer> transformOnInputTransform(OneInputTransformation<IN, OUT> transform) {
// 递归对该transform的直接上游transform进行转换，获得直接上游id集合
Collection<Integer> inputIds = transform(transform.getInput());

// 递归调用可能已经处理过该transform了
if (alreadyTransformed.containsKey(transform)) {
  return alreadyTransformed.get(transform);
}

String slotSharingGroup = determineSlotSharingGroup(transform.getSlotSharingGroup(), inputIds);

// 添加 StreamNode
streamGraph.addOperator(transform.getId(),
    slotSharingGroup,
    transform.getOperator(),
    transform.getInputType(),
    transform.getOutputType(),
    transform.getName());

if (transform.getStateKeySelector() != null) {
  TypeSerializer<?> keySerializer = transform.getStateKeyType().createSerializer(env.getConfig());
  streamGraph.setOneInputStateKey(transform.getId(), transform.getStateKeySelector(), keySerializer);
}

streamGraph.setParallelism(transform.getId(), transform.getParallelism());

// 添加 StreamEdge
for (Integer inputId: inputIds) {
  streamGraph.addEdge(inputId, transform.getId(), 0);
}

return Collections.singleton(transform.getId());
}
```

该函数首先会对该transform的上游transform进行递归转换，确保上游的都已经完成了转化。然后通过transform构造出`StreamNode`，最后与上游的`transform`进行连接，构造出`StreamNode`。