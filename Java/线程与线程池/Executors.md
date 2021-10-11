# [Java通过Executors提供四种线程池](https://www.cnblogs.com/webglcn/p/5265901.html)



Java通过Executors提供四种线程池，分别为：
newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。
newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行

**(1) newCachedThreadPool**
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。示例代码如下：

```java
package test;  
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  
public class ThreadPoolExecutorTest {  
 public static void main(String[] args) {  
  ExecutorService cachedThreadPool = Executors.newCachedThreadPool();  
  for (int i = 0; i < 10; i++) {  
   final int index = i;  
   try {  
    Thread.sleep(index * 1000);  
   } catch (InterruptedException e) {  
    e.printStackTrace();  
   }  
   cachedThreadPool.execute(new Runnable() {  
    public void run() {  
     System.out.println(index);  
    }  
   });  
  }  
 }  
} 

//线程池为无限大，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。
```





### (3) newScheduledThreadPool

创建一个定长线程池，支持定时及周期性任务执行。延迟执行示例代码如下：

```java
//表示延迟3秒执行。
package test;  
import java.util.concurrent.Executors;  
import java.util.concurrent.ScheduledExecutorService;  
import java.util.concurrent.TimeUnit;  
public class ThreadPoolExecutorTest {  
 public static void main(String[] args) {  
  ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);  
  scheduledThreadPool.schedule(new Runnable() {  
   public void run() {  
    System.out.println("delay 3 seconds");  
   }  
  }, 3, TimeUnit.SECONDS);  
 }  
} 


//表示延迟1秒后每3秒执行一次。
package test;  
import java.util.concurrent.Executors;  
import java.util.concurrent.ScheduledExecutorService;  
import java.util.concurrent.TimeUnit;  
public class ThreadPoolExecutorTest {  
 public static void main(String[] args) {  
  ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);  
  scheduledThreadPool.scheduleAtFixedRate(new Runnable() {  
   public void run() {  
    System.out.println("delay 1 seconds, and excute every 3 seconds");  
   }  
  }, 1, 3, TimeUnit.SECONDS);  
 }  
}
```

