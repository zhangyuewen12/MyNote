```
package java.util.concurrent;

/**
 * A task that returns a result and may throw an exception.
 * Implementors define a single method with no arguments called
 * {@code call}.
 *
 * <p>The {@code Callable} interface is similar to {@link
 * java.lang.Runnable}, in that both are designed for classes whose
 * instances are potentially executed by another thread.  A
 * {@code Runnable}, however, does not return a result and cannot
 * throw a checked exception.
 *
 * <p>The {@link Executors} class contains utility methods to
 * convert from other common forms to {@code Callable} classes.
 *
 * @see Executor
 * @since 1.5
 * @author Doug Lea
 * @param <V> the result type of method {@code call}
 */
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}

```





在Java中，java.util.concurrent.Callable是一个函数式接口，它允许定义一个可以由其他线程执行并返回结果的任务。与Runnable接口不同，Callable接口的call()方法可以返回一个结果，并且可以抛出一个受检异常。

Callable接口定义了一个带有泛型类型参数的call()方法，形如：`V call() throws Exception;`，其中V表示call()方法返回的结果类型。当我们想要执行某些需要返回结果的任务时，就可以实现Callable接口，并在call()方法中编写任务逻辑。

Callable接口通常与ExecutorService一起使用，ExecutorService是用来执行Runnable或Callable任务的线程池。通过将Callable任务提交给ExecutorService，我们可以并行地执行多个任务，并且可以通过Future对象来获取任务的执行结果。

下面是一个简单的示例，演示了如何使用Callable和ExecutorService来执行一个简单的任务并获取结果：

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CallableExample {
    public static void main(String[] args) {
        // 创建一个单线程的线程池
        ExecutorService executor = Executors.newSingleThreadExecutor();

        // 定义一个Callable任务
        Callable<Integer> task = () -> {
            // 模拟耗时操作
            Thread.sleep(2000);
            return 42;
        };

        try {
            // 提交Callable任务并获取Future对象
            Future<Integer> future = executor.submit(task);

            // 阻塞等待任务完成并获取结果
            Integer result = future.get();
            System.out.println("Result: " + result);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 关闭线程池
            executor.shutdown();
        }
    }
}
```

在这个示例中，我们创建了一个Callable任务，该任务会睡眠2秒钟然后返回整数42。然后，我们使用ExecutorService提交了这个任务，并获取了一个Future对象。通过Future对象，我们可以在需要时阻塞等待任务完成，并获取任务的结果。

总的来说，Callable接口提供了一种方便的方式来定义可返回结果并且可能会抛出异常的任务，它与ExecutorService结合使用可以实现并发执行任务并获取结果的功能。