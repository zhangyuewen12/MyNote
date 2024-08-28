# 一、实现多线程的方式

**1.继承Thread来实现多线程**

Java提供了一个超类Thread给我们来extends，一旦继承了它，就可以通过override 其中的run方法，来实现多线程，具体代码如下：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221221110552080.png" alt="image-20221221110552080" style="zoom:50%;" />

**2.通过实现Runnable接口来实现**

因为对于一些类来说，他们不能继承Thread来实现多线程，因为Java规定同时只能继承一个超类，但是却可以同时实现多个接口，因此Runnable就更格外受欢迎。具体代码如下

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221221110613350.png" alt="image-20221221110613350" style="zoom:50%;" />

上面这种是直接定义了类来实现了Runnable方法，其实还可以变种为匿名内部类的方法来创建出一个Thread，具体如下：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221221110644778.png" alt="image-20221221110644778" style="zoom:50%;" />

**3.通过Callable来实现一个Thread**

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221221110712927.png" alt="image-20221221110712927" style="zoom:50%;" />



# 二、多线程同步

当我们有多个线程要同时访问一个变量或对象时，如果这些线程中既有读又有写操作时，就会导致变量值或对象的状态出现混乱，从而导致程序异常。Java提供三种同步机制的方式

## 1、synchronized锁住方法

```
public class Bank {
    private int count =0;//账户余额

    //存钱
    public synchronized void addMoney(int money){
        
        count +=money;
        System.out.println(System.currentTimeMillis()+"存进："+money);
    }

    //取钱
    public synchronized void subMoney(int money){
        if(count-money < 0){
            System.out.println("余额不足");
            return;
        }
        count -=money;
        System.out.println(+System.currentTimeMillis()+"取出："+money);
    }

    //查询
    public void lookMoney(){
        System.out.println("账户余额："+count);
    }
}
```



## 2、wait 与notify 

当使用synchoronized来修饰某个共享资源的时候，如果线程A1在执行synchoronized代码，另外一个线程A2也要执行同一代码，线程A2将要等待A1执行完成。

在synchoronized代码被执行期间，线程可以通过调用对象的wait方法，释放对象锁。进入等待状态，并且可以通过notify方法通知等待线程去重新获取锁。wait和notify都是native方法，具体实现由虚拟机本地的C代码执行。

```java
package com.example.demo;

/**
 * TestThread
 *
 * @author zhangyuewen
 * @since 2022/12/21
 **/
public class TestThread {
    private Object lock = new Object();
    private boolean envReady = false;
    private  class WorkerThead extends Thread{
        @Override
        public void run() {
            System.out.println("线程Work等待拿到锁");
            synchronized (lock){
                System.out.println("线程Work拿到锁");
                if (!envReady){
                    System.out.println("准备工作未完成，线程Work释放锁");
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("线程Work 拿到锁");
                    System.out.println("线程Work 完成");

                }
            }

        }
    }

    private class PrepareEnvThread extends Thread{
        @Override
        public void run() {
            System.out.println("PrepareEnvThread 等到拿锁！");
            synchronized (lock){
                System.out.println("PrepareEnvThread 拿到锁！");
                envReady = true;
               
                lock.notify();
                System.out.println("准备工作完成，通知Work");
            }
        }
    }

    public void prepareEnv(){
        new PrepareEnvThread().start();
    }
    public void work(){
        new WorkerThead().start();
    }
    public static void main(String[] args) {
        TestThread testThread = new TestThread();
        testThread.work();
        try {
            Thread.sleep(1000);
        }catch (Exception e){
            e.printStackTrace();
        }
        testThread.prepareEnv();
    }
}

```

> 线程Work等待拿到锁
> 线程Work拿到锁
> 准备工作未完成，线程Work释放锁
> PrepareEnvThread 等到拿锁！
> PrepareEnvThread 拿到锁！
> 准备工作完成，通知Work
> 线程Work 拿到锁
> 线程Work 完成

注意事项：

	1. wait和notify方法在synchonized方法的调用必须处在该对象的锁中。用这些方法的时候，首先要获取锁。
	1. 线程调用notify后，只有等待代码退出synchronized后，被唤醒的进程才能去获取锁。
	1. 被notify的进程，不会被立刻执行。只是重新进入获取锁的阻塞队列里。

## 3、使用重入锁实现线程同步

在JavaSE5.0中新增了一个java.util.concurrent包来支持同步。

ReentrantLock类是可重入、互斥、实现了Lock接口的锁。

ReentrantLock锁的实现原理虽然和synchronized不用，但是它和synchronized一样都是通过保证线程间的互斥访问临界区，来保证线程安全，实现线程间的通信。相比于**synchronized使用Object类的三个方法（wait、notify、notifyAll）来实现线程的阻塞和运行两个状态的切换**，**ReentrantLock使用Condition阻塞队列的await()、signal()、signalAll()三个方法来实现线程阻塞和运行两个状态的切换，进而实现线程间的通信**,而且相比前者使用起来更清晰也更简单。前者是java底层级别的，后者是语言级别的，后者可控制性和扩展性更好。

```
   //只给出要修改的代码，其余代码与上同
    class Bank {
        
        private int account = 100;
        //需要声明这个锁
        private Lock lock = new ReentrantLock();
        public int getAccount() {
            return account;
        }
        //这里不再需要synchronized 
        public void save(int money) {
            lock.lock();
            try{
                account += money;
            }finally{
                lock.unlock();
            }
            
        }
    ｝
```

**关于Lock对象和synchronized关键字的选择**：

```handlebars
> a.最好两个都不用，使用一种java.util.concurrent包提供的机制，能够帮助用户处理所有与锁相关的代码。 
> b.如果synchronized关键字能满足用户的需求，就用synchronized，因为它能简化代码 
> c.如果需要更高级的功能，就用ReentrantLock类，此时要注意及时释放锁，否则会出现死锁，通常在finally代码释放锁
```

# 三 、Lock的分类

## 3.1 ReentrantLock可重入锁

可重入锁又名递归锁，直指同一个线程在外层方法获得锁之后，在进入内层方法时，会自动获得锁。**ReentrantLock**和**Synchronized**都是可重入锁。可重入锁的好处之一就是在一定程度上避免死锁。

```
package com.example.demo;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * ReentrantLockTest
 *
 * @author zhangyuewen
 * @since 2022/12/21
 **/
public class ReentrantLockTest {
    private Lock lock = new ReentrantLock();

    public void method1(){
        lock.lock();
        System.out.println("method1 is called");
        method2();
        lock.unlock();
    }
    public void method2(){
        lock.lock();
        System.out.println("method2 is called");
        lock.unlock();
    }
    public static void main(String[] args) {
        new ReentrantLockTest().method1();
    }
}

```

在method1 中已经通过lock方法获取到了锁，然后调用method2的时候通过lock仍然呢个获取到锁，这就体现了重入锁的特性。

ReentrantLock锁的实现原理虽然和synchronized不用，但是它和synchronized一样都是通过保证线程间的互斥访问临界区，来保证线程安全，实现线程间的通信。相比于**synchronized使用Object类的三个方法（wait、notify、notifyAll）来实现线程的阻塞和运行两个状态的切换**，**ReentrantLock使用Condition阻塞队列的await()、signal()、signalAll()三个方法来实现线程阻塞和运行两个状态的切换，进而实现线程间的通信**,而且相比前者使用起来更清晰也更简单。前者是java底层级别的，后者是语言级别的，后者可控制性和扩展性更好。

> 

## 3.2 ReentrantReadWriteLock

ReentrantLock可重入锁，但其存在明显的弊端。对于读场景而言，实际完全可以允许多个线程同时访问，而不必使用独占锁来进行并发保护。故ReentrantReadWriteLock读写锁应运而生。其内部维护了两个锁——读锁、写锁。前者为共享锁，后者则为互斥锁。



# 四、synchronized和Lock的异同

区别：1、lock是一个接口，而synchronized是java的一个关键字。2、synchronized在发生异常时会自动释放占有的锁，因此不会出现[死锁](https://so.csdn.net/so/search?q=死锁&spm=1001.2101.3001.7020)；而lock发生异常时，不会主动释放占有的锁，必须手动来释放锁，可能引起死锁的发生。



# 五、sleep和wait的区别

sleep和wait的区别在于这两个方法来自不同的类分别是Thread和Object，sleep方法没有释放锁，而wait方法释放了锁，使得其他线程可以使用同步控制块或者方法。wait只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用(使用范围)。

sleep的作用

sleep的作用是让线程休眠制定的时间，在时间到达时恢复，也就是说sleep将在接到时间到达事件事恢复线程执行。

wait的作用

调用wait方法将会将调用者的线程挂起，直到其他线程调用同一个对象的notify方法才会重新激活调用者。

sleep与wait差异总结

1、来自不同的类：sleep是Thread的静态类方法，谁调用的谁去睡觉，即使在a线程里调用了b的sleep方法，实际上还是a去睡觉，要让b线程睡觉要在b的代码中调用sleep。

2、有没有释放锁(释放资源)：sleep不出让系统资源;wait是进入线程等待池等待，出让系统资源，其他线程可以占用CPU。

3、一般wait不会加时间限制，因为如果wait线程的运行资源不够，再出来也没用，要等待其他线程调用notify/notifyAll唤醒等待池中的所有线程，才会进入就绪队列等待OS分配系统资源。sleep(milliseconds)可以用时间指定使它自动唤醒过来，如果时间不到只能调用interrupt()强行打断。

4、sleep必须捕获异常，而wait，notify和notifyAll不需要捕获异常。

# 六、死锁产生的4个必要条件

1 . 产生死锁的必要条件：
（1）互斥条件：进程要求对所分配的资源进行排它性控制，即在一段时间内某资源仅为一进程所占用。
（2）请求和保持条件：当进程因请求资源而阻塞时，对已获得的资源保持不放。
（3）不剥夺条件：进程已获得的资源在未使用完之前，不能剥夺，只能在使用完时由自己释放。
（4）环路等待条件：在发生死锁时，必然存在一个进程–资源的环形链。

# 七、join方法的作用

Join方法的作用是让调用该方法的线程在执行玩run()方法后，再继续执行join方法后面的代码。当main主线程调用g的join()方法时，main线程会获得线程对象g的锁，调用该对象的wait（等待时间）方法，直到该对象唤醒main线程。

# 八、ExecutorService的submit和execute分别有什么区别？

1.execute没有返回值，如果不需要知道线程的结果就使用execute方法，性能会好很多。

2.submit返回一个Future对象，如果想知道线程结果就需要使用submit提交，而且它能在主线程中通过Future的get方法捕获线程中的异常。

# 九、yield方法的作用

yield方法的作用: 释放我的CPU时间片. (比如某个时间段,虽然我获得了cpu的执行权,但是并不满足执行的条件, 把cpu的执行权让给了其他的线程. ) . 注意的是, 即使释放了cpu的时间片, 但是线程的状态, 依然还是runnable. 即使刚刚放弃了执行的权利, 也可能下一次就被调度回来了.

定位: jvm不保证一定会把cpu的执行权让给其他线程. 因此生产环境中, 一般不会使用yield, 但是在一些并发包的源码中, 会运用到yield .

yield和sleep的区别: 是否随时可能再次被调度. sleep期间, 它是已经被阻塞了, 不会把它再调度起来. 但是yield是暂时把调度权让给其他线程, 下次也可能会被调度到.

# 十、线程池

## 1.线程池的核心参数

```
public ThreadPoolExecutor(int corePoolSize,//核心线程数
                          int maximumPoolSize,//最大线程数
                          long keepAliveTime,//线程空闲时间
                          TimeUnit unit,//时间单位
                          BlockingQueue<Runnable> workQueue,//任务队列
                          ThreadFactory threadFactory,//线程工厂
                          RejectedExecutionHandler handler//拒绝策略) 
{
    ...
}
```

## 2.线程池的执行顺序

线程池按以下行为执行任务

- 当线程数小于核心线程数时，创建线程。
- 当线程数大于等于核心线程数，且任务队列未满时，将任务放入任务队列。
- 当线程数大于等于核心线程数，且任务队列已满，若线程数小于最大线程数，创建线程。
- 若线程数等于最大线程数，则执行拒绝策略



## **1. Executor**

Executor提供了execute()接口来执行已提交的Runnable执行目标实例,它只有1个方法：

```
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
```



## **2. ExecutorService**

继承于Executor,Java异步目标任务的“执行者服务接”口，对外提供异步任务的接收服务

```java
public interface ExecutorService extends Executor {
    void shutdown();
    boolean isShutdown();
    <T> Future<T> submit(Runnable task, T result);
    ....
}
```

## **3. AbstractExecutorService**

抽象类，实现了ExecutorService

```
Provides default implementations of ExecutorService execution methods. This class implements the submit, invokeAny and invokeAll methods using a RunnableFuture returned by newTaskFor, which defaults to the FutureTask class provided in this package. For example, the implementation of submit(Runnable) creates an associated RunnableFuture that is executed and returned. Subclasses may override the newTaskFor methods to return RunnableFuture implementations other than FutureTask.

public abstract class AbstractExecutorService implements ExecutorService {

    /**
     * Returns a {@code RunnableFuture} for the given runnable and default
     * value.
     *
     * @param runnable the runnable task being wrapped
     * @param value the default value for the returned future
     * @param <T> the type of the given value
     * @return a {@code RunnableFuture} which, when run, will run the
     * underlying runnable and which, as a {@code Future}, will yield
     * the given value as its result and provide for cancellation of
     * the underlying task
     * @since 1.6
     */
    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }
}
```



## **4. ThreadPoolExecutor**

线程池实现类，继承于AbstractExecutorService，JUC线程池的**核心实现类**

```
public class ThreadPoolExecutor extends AbstractExecutorService {
	/**
	* Factory for new threads. All threads are created using this factory (via method addWorker)
	**/
	
	private volatile ThreadFactory threadFactory;
	
	Handler called when saturated or shutdown in execute.
	private volatile RejectedExecutionHandler handler;
	
	private volatile int corePoolSize;
	
	private volatile int maximumPoolSize;
	
	private int largestPoolSize;
	
	The queue used for holding tasks and handing off to worker threads.
	private final BlockingQueue<Runnable> workQueue;
	
	
	private final ReentrantLock mainLock = new ReentrantLock();
	
	
	private final HashSet<Worker> workers = new HashSet<Worker>();
	
  public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
   }	
   
   public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }
}
```



## **5. ScheduledExecutorService**

继承于ExecutorService。它是一个可以完成“延时”和“周期性”任务的调度线程池接口

## **6. ScheduledThreadPoolExecutor**

继承于ThreadPoolExecutor，实现了ExecutorService中延时执行和周期执行等抽象方法

## **7. Executors**

静态工厂类，它通过静态工厂方法返回***\*ExecutorService\****、**ScheduledExecutorService**等线程池示例对象

**1. newSingleThreadExetor创建“单线程化线程池”**

**2. newFixedThreadPool创建“固定数量的线程池**

**3. newCachedThreadPool创建“可缓存线程池”**

**4. newScheduledThreadPool创建“可调度线程池”**

