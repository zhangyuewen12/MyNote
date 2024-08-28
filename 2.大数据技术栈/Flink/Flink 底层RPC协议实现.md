# [【Flink】Flink 底层RPC框架分析](https://www.cnblogs.com/leesf456/p/11120045.html)

# 1. 前言

> 对于Flink中各个组件（JobMaster、TaskManager、Dispatcher等），其底层RPC框架基于Akka实现，本文着重分析Flink中的Rpc框架实现机制及梳理其通信流程



# 2. Akka介绍

在Actor模型中，所有的实体被认为是独立的actors。actors和其他actors通过发送异步消息通信。

每个actor有一个邮箱(mailbox)，它收到的消息存储在里面。另外，每一个actor维护自身单独的状态。一个Actors网络如下所示：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210706102608401.png" alt="image-20210706102608401" style="zoom:50%;" />

每个actor是一个单一的线程，它不断地从其邮箱中poll(拉取)消息，并且连续不断地处理。对于已经处理过的消息的结果，actor可以改变它自身的内部状态或者发送一个新消息或者孵化一个新的actor。**尽管单个的actor是自然有序的，但一个包含若干个actor的系统却是高度并发的并且极具扩展性的。因为那些处理线程是所有actor之间共享的。**这也是我们为什么不该在actor线程里调用可能导致阻塞的“调用”。因为这样的调用可能会阻塞该线程使得他们无法替其他actor处理消息。

## 2.1. 创建Akka系统

Akka系统的核心ActorSystem和Actor，若需构建一个Akka系统，首先需要创建ActorSystem，创建完ActorSystem后，可通过其创建Actor（注意：Akka不允许直接new一个Actor，只能通过 Akka 提供的某些 API 才能创建或查找 Actor，一般会通过 ActorSystem#actorOf和ActorContext#actorOf来创建 Actor），另外，我们只能通过ActorRef（Actor的引用， 其对原生的 Actor 实例做了良好的封装，外界不能随意修改其内部状态）来与Actor进行通信。如下代码展示了如何配置一个Akka系统。

```java
// 1. 构建ActorSystem
// 使用缺省配置
ActorSystem system = ActorSystem.create("sys");
// 也可显示指定appsys配置
// ActorSystem system1 = ActorSystem.create("helloakka", ConfigFactory.load("appsys"));

// 2. 构建Actor,获取该Actor的引用，即ActorRef
ActorRef helloActor = system.actorOf(Props.create(HelloActor.class), "helloActor");

// 3. 给helloActor发送消息
helloActor.tell("hello helloActor", ActorRef.noSender());

// 4. 关闭ActorSystem
system.terminate();
```

在Akka中，创建的每个Actor都有自己的路径，该路径遵循 ActorSystem 的层级结构，大致如下：

```
本地：akka://sys/user/helloActor
远程：akka.tcp://sys@l27.0.0.1:2020/user/remoteActor 
```

其中本地路径含义如下：

- sys，创建的ActorSystem的名字；
- user，通过ActorSystem#actorOf和ActorContext#actorOf 方法创建的 Actor 都属于/user下，与/user对应的是/system， 其是系统层面创建的，与系统整体行为有关，在开发阶段并不需要对其过多关注；
- helloActor，我们创建的HelloActor

其中远程部分路径含义如下：

- akka.tcp，远程通信方式为tcp；
- sys@127.0.0.1:2020，ActorSystem名字及远程主机ip和端口号。

## 2.2. 根据path获取Actor

若提供了Actor的路径，可以通过路径获取到ActorRef，然后与之通信，代码如下所示：

```java
ActorSystem system = ActorSystem.create("sys")；
ActorSelection as = system.actorSelection("/path/to/actor");

Timeout timeout = new Timeout(Duration.create(2, "seconds"));
Future<ActorRef> fu = as.resolveOne(timeout);

fu.onSuccess(new OnSuccess<ActorRef>() {
    @Override
    public void onSuccess(ActorRef actor) {
        System.out.println("actor:" + actor);
        actor.tell("hello actor", ActorRef.noSender());
    }
}, system.dispatcher());

fu.onFailure(new OnFailure() {
    @Override
    public void onFailure(Throwable failure) {
        System.out.println("failure:" + failure);
    }
}, system.dispatcher());
```

由上面可知，若需要与远端Actor通信，路径中必须提供ip:port。

## 2.3. 与Actor通信

### 2.3.1. tell方式

当使用tell方式时，表示仅仅使用异步方式给某个Actor发送消息，无需等待Actor的响应结果，并且也不会阻塞后续代码的运行，如：

```java
helloActor.tell("hello helloActor", ActorRef.noSender());
```

其中：第一个参数为消息，它可以是任何可序列化的数据或对象，第二个参数表示发送者，通常来讲是另外一个 Actor 的引用， ActorRef.noSender()表示无发送者（（实际上是一个 叫做deadLetters的Actor）。

### 2.3.2. ask方式

当我们需要从Actor获取响应结果时，可使用ask方法，ask方法会将返回结果包装在scala.concurrent.Future中，然后通过异步回调获取返回结果。 如调用方：

```java
// 异步发送消息给Actor，并获取响应结果
Future<Object> fu = Patterns.ask(printerActor, "hello helloActor", timeout);
fu.onComplete(new OnComplete<Object>() {
    @Override
    public void onComplete(Throwable failure, String success) throws Throwable {
         if (failure != null) {
             System.out.println("failure is " + failure);
         } else {
             System.out.println("success is " + success);
         }
    }
}, system.dispatcher());
```

HelloActor处理消息方法的代码大致如下：

```java
private void handleMessage(Object object) {
    if (object instanceof String) {
      String str = (String) object;
      log.info("[HelloActor] message is {}, sender is {}", str, 	getSender().path().toString());
      // 给发送者发送消息
      getSender().tell(str, getSelf());
    }
  }
```

上面主要介绍了Akka中的ActorSystem、Actor，及与Actor的通信；Flink借此构建了其底层通信系统

# 3. RPC类图结构

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210706103300521.png" alt="image-20210706103300521" style="zoom:50%;" />

## 3.1. RpcGateway

Flink的RPC协议通过RpcGateway来定义；由前面可知，若想与远端`Actor`通信，则必须提供地址（ip和port），如在Flink-on-Yarn模式下，JobMaster会先启动ActorSystem，此时TaskExecutor的Container还未分配，后面与TaskExecutor通信时，必须让其提供对应地址，从类继承图可以看到基本上所有组件都实现了RpcGateway接口，其代码如下：

```java
public interface RpcGateway {

	/**
	 * Returns the fully qualified address under which the associated rpc endpoint is reachable.
	 *
	 * @return Fully qualified (RPC) address under which the associated rpc endpoint is reachable
	 */
	String getAddress();

	/**
	 * Returns the fully qualified hostname under which the associated rpc endpoint is reachable.
	 *
	 * @return Fully qualified hostname under which the associated rpc endpoint is reachable
	 */
	String getHostname();
}
```

## 3.2. RpcEndpoint

每个RpcEndpoint对应了一个路径（endpointId和actorSystem共同确定），每个路径对应一个Actor，其实现了RpcGateway接口，其构造函数如下：

```java
protected RpcEndpoint(final RpcService rpcService, final String endpointId) {
	// 保存rpcService和endpointId
	this.rpcService = checkNotNull(rpcService, "rpcService");
	this.endpointId = checkNotNull(endpointId, "endpointId");
	// 通过RpcService启动RpcServer
	this.rpcServer = rpcService.startServer(this);
	// 主线程执行器，所有调用在主线程中串行执行
	this.mainThreadExecutor = new MainThreadExecutor(rpcServer, 	this::validateRunsInMainThread);
}
```

在RpcEndpoint中还定义了一些方法如`runAsync(Runnable)`、`callAsync(Callable, Time)`方法来执行Rpc调用，值得注意的是在Flink的设计中，对于同一个Endpoint，所有的调用都运行在主线程，因此不会有并发问题，当启动RpcEndpoint/进行Rpc调用时，其会委托RcpServer进行处理.

## 3.3. RpcService

Rpc服务的接口，其主要作用如下：

- 根据提供的RpcEndpoint来启动RpcServer（Actor）；
- 根据提供的地址连接到RpcServer，并返回一个RpcGateway；
- 延迟/立刻调度Runnable、Callable；
- 停止RpcServer（Actor）或自身服务；

在Flink中其实现类为AkkaRpcService。

### 3.3.1. AkkaRpcService

AkkaRpcService中封装了ActorSystem，并保存了ActorRef到RpcEndpoint的映射关系，在构造RpcEndpoint时会启动指定rpcEndpoint上的RpcServer，其会根据Endpoint类型（FencedRpcEndpoint或其他）来创建不同的Actor（FencedAkkaRpcActor或AkkaRpcActor），并将RpcEndpoint和Actor对应的ActorRef保存起来，然后使用动态代理创建RpcServer，具体代码如下：

```java
	public <C extends RpcEndpoint & RpcGateway> RpcServer startServer(C rpcEndpoint) {
		checkNotNull(rpcEndpoint, "rpc endpoint");

		CompletableFuture<Void> terminationFuture = new CompletableFuture<>();
		final Props akkaRpcActorProps;
		// 根据RpcEndpoint类型创建不同类型的Props
		if (rpcEndpoint instanceof FencedRpcEndpoint) {
			akkaRpcActorProps = Props.create(
				FencedAkkaRpcActor.class,
				rpcEndpoint,
				terminationFuture,
				getVersion(),
				configuration.getMaximumFramesize());
		} else {
			akkaRpcActorProps = Props.create(
				AkkaRpcActor.class,
				rpcEndpoint,
				terminationFuture,
				getVersion(),
				configuration.getMaximumFramesize());
		}

		ActorRef actorRef;
		// 同步块，创建Actor，并获取对应的ActorRef
		synchronized (lock) {
			checkState(!stopped, "RpcService is stopped");
			actorRef = actorSystem.actorOf(akkaRpcActorProps, rpcEndpoint.getEndpointId());
			actors.put(actorRef, rpcEndpoint);
		}

		LOG.info("Starting RPC endpoint for {} at {} .", rpcEndpoint.getClass().getName(), actorRef.path());

		// 获取Actor的路径
		final String akkaAddress = AkkaUtils.getAkkaURL(actorSystem, actorRef);
		final String hostname;
		Option<String> host = actorRef.path().address().host();
		if (host.isEmpty()) {
			hostname = "localhost";
		} else {
			hostname = host.get();
		}
		// 解析该RpcEndpoint实现的所有RpcGateway接口
		Set<Class<?>> implementedRpcGateways = new HashSet<>(RpcUtils.extractImplementedRpcGateways(rpcEndpoint.getClass()));
		
		// 额外添加RpcServer和AkkaBasedEnpoint类
		implementedRpcGateways.add(RpcServer.class);
		implementedRpcGateways.add(AkkaBasedEndpoint.class);

		final InvocationHandler akkaInvocationHandler;

		// 根据不同类型动态创建代理对象
		if (rpcEndpoint instanceof FencedRpcEndpoint) {
			// a FencedRpcEndpoint needs a FencedAkkaInvocationHandler
			akkaInvocationHandler = new FencedAkkaInvocationHandler<>(
				akkaAddress,
				hostname,
				actorRef,
				configuration.getTimeout(),
				configuration.getMaximumFramesize(),
				terminationFuture,
				((FencedRpcEndpoint<?>) rpcEndpoint)::getFencingToken);

			implementedRpcGateways.add(FencedMainThreadExecutable.class);
		} else {
			akkaInvocationHandler = new AkkaInvocationHandler(
				akkaAddress,
				hostname,
				actorRef,
				configuration.getTimeout(),
				configuration.getMaximumFramesize(),
				terminationFuture);
		}

		// Rather than using the System ClassLoader directly, we derive the ClassLoader
		// from this class . That works better in cases where Flink runs embedded and all Flink
		// code is loaded dynamically (for example from an OSGI bundle) through a custom ClassLoader
		ClassLoader classLoader = getClass().getClassLoader();

		// 生成RpcServer对象，而后对该server的调用都会进入Handler的invoke方法处理，handler实现了多个接口的方法
		@SuppressWarnings("unchecked")
		RpcServer server = (RpcServer) Proxy.newProxyInstance(
			classLoader,
			implementedRpcGateways.toArray(new Class<?>[implementedRpcGateways.size()]),
			akkaInvocationHandler);

		return server;
	}
```

当启动RpcServer后，即创建了相应的Actor（注意此时Actor的处于停止状态）和动态代理对象，需要调用RpcEndpoint#start启动启动Actor，此时启动RpcEndpoint流程如下（以非FencedRpcEndpoint为例）：

- 调用RpcEndpoint#start；

- 委托给RpcServer#start；

- 调用动态代理的AkkaInvocationHandler#invoke；发现调用的是StartStoppable#start方法，则直接进行本地方法调用；invoke方法的代码如下：

  ```java
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  		Class<?> declaringClass = method.getDeclaringClass();
  
  		Object result;
  		// 先匹配指定类型（handler已实现接口的方法），若匹配成功则直接进行本地方法调用；若匹配为FencedRpcGateway类型，则抛出异常（应该在FencedAkkaInvocationHandler中处理）；其他则进行Rpc调用
  		if (declaringClass.equals(AkkaBasedEndpoint.class) ||
  			declaringClass.equals(Object.class) ||
  			declaringClass.equals(RpcGateway.class) ||
  			declaringClass.equals(StartStoppable.class) ||
  			declaringClass.equals(MainThreadExecutable.class) ||
  			declaringClass.equals(RpcServer.class)) {
  			result = method.invoke(this, args);
  		} else if (declaringClass.equals(FencedRpcGateway.class)) {
  			throw new UnsupportedOperationException("AkkaInvocationHandler does not support the call FencedRpcGateway#" +
  				method.getName() + ". This indicates that you retrieved a FencedRpcGateway without specifying a " +
  				"fencing token. Please use RpcService#connect(RpcService, F, Time) with F being the fencing token to " +
  				"retrieve a properly FencedRpcGateway.");
  		} else {
  			result = invokeRpc(method, args);
  		}
  
  		return result;
  	}
  ```

- 调用AkkaInvocationHandler#start；

- 通过ActorRef#tell给对应的Actor发送消息`rpcEndpoint.tell(ControlMessages.START, ActorRef.noSender());`；

- 调用AkkaRpcActor#handleControlMessage处理控制类型消息；

- 在主线程中将自身状态变更为Started状态；

经过上述步骤就完成了Actor的启动过程，Actor启动后便可与Acto通信让其执行代码（如runSync/callSync等）和处理Rpc请求了。下面分别介绍处理执行代码和处理Rpc请求；

