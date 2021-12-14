## Deployment Modes [#](https://nightlies.apache.org/flink/flink-docs-release-1.14/docs/deployment/overview/#deployment-modes)

Flink can execute applications in one of three ways:

- in Application Mode,
- in a Per-Job Mode,
- in Session Mode.

The above modes differ in:

- the cluster lifecycle and resource isolation guarantees
- whether the application’s `main()` method is executed on the client or on the cluster.



![image-20211104090956169](/Users/zyw/Library/Application Support/typora-user-images/image-20211104090956169.png)

#### Application Mode [#](https://nightlies.apache.org/flink/flink-docs-release-1.14/docs/deployment/overview/#application-mode)

In all the other modes, the application’s `main()` method is executed on the client side. This process includes downloading the application’s dependencies locally, executing the `main()` to extract a representation of the application that Flink’s runtime can understand (i.e. the `JobGraph`) and ship the dependencies and the `JobGraph(s)` to the cluster. This makes the Client a heavy resource consumer as it may need substantial network bandwidth to download dependencies and ship binaries to the cluster, and CPU cycles to execute the `main()`. This problem can be more pronounced when the Client is shared across users.

Building on this observation, the *Application Mode* creates a cluster per submitted application, but this time, the `main()` method of the application is executed on the JobManager. Creating a cluster per application can be seen as creating a session cluster shared only among the jobs of a particular application, and torn down when the application finishes. With this architecture, the *Application Mode* provides the same resource isolation and load balancing guarantees as the *Per-Job* mode, but at the granularity of a whole application. Executing the `main()` on the JobManager allows for saving the CPU cycles required, but also save the bandwidth required for downloading the dependencies locally. Furthermore, it allows for more even spread of the network load for downloading the dependencies of the applications in the cluster, as there is one JobManager per application.

> In the Application Mode, the `main()` is executed on the cluster and not on the client, as in the other modes. This may have implications for your code as, for example, any paths you register in your environment using the `registerCachedFile()` must be accessible by the JobManager of your application.

Compared to the *Per-Job* mode, the *Application Mode* allows the submission of applications consisting of multiple jobs. The order of job execution is not affected by the deployment mode but by the call used to launch the job. Using `execute()`, which is blocking, establishes an order and it will lead to the execution of the “next” job being postponed until “this” job finishes. Using `executeAsync()`, which is non-blocking, will lead to the “next” job starting before “this” job finishes.

> The Application Mode allows for multi-`execute()` applications but High-Availability is not supported in these cases. High-Availability in Application Mode is only supported for single-`execute()` applications.
>
> Additionally, when any of multiple running jobs in Application Mode (submitted for example using `executeAsync()`) gets cancelled, all jobs will be stopped and the JobManager will shut down. Regular job completions (by the sources shutting down) are supported.



#### Summary [#](https://nightlies.apache.org/flink/flink-docs-release-1.14/docs/deployment/overview/#summary)

In *Session Mode*, the cluster lifecycle is independent of that of any job running on the cluster and the resources are shared across all jobs. The *Per-Job* mode pays the price of spinning up a cluster for every submitted job, but this comes with better isolation guarantees as the resources are not shared across jobs. In this case, the lifecycle of the cluster is bound to that of the job. Finally, the *Application Mode* creates a session cluster per application and executes the application’s `main()` method on the cluster.