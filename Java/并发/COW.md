# [写入时复制（CopyOnWrite）与 读写锁](https://www.cnblogs.com/snm511/p/14084649.html)

## 一、CopyOnWrite 思想

写入时复制（CopyOnWrite，简称COW）思想是计算机程序设计领域中的一种通用优化策略。其核心思想是，如果有多个调用者（Callers）同时访问相同的资源（如内存或者是磁盘上的数据存储），他们会共同获取相同的指针指向相同的资源，直到某个调用者修改资源内容时，系统才会真正复制一份专用副本（private copy）给该调用者，而其他调用者所见到的最初的资源仍然保持不变。这过程对其他的调用者都是透明的（transparently）。此做法主要的优点是如果调用者没有修改资源，就不会有副本（private copy）被创建，因此多个调用者只是读取操作时可以共享同一份资源。

通俗易懂的讲，写入时复制技术就是不同进程在访问同一资源的时候，只有更新操作，才会去复制一份新的数据并更新替换，否则都是访问同一个资源。

JDK 的 CopyOnWriteArrayList/CopyOnWriteArraySet 容器正是采用了 COW 思想，它是如何工作的呢？简单来说，就是平时查询的时候，都不需要加锁，随便访问，只有在更新的时候，才会从原来的数据复制一个副本出来，然后修改这个副本，最后把原数据替换成当前的副本。修改操作的同时，读操作不会被阻塞，而是继续读取旧的数据。这点要跟读写锁区分一下。

## 二、源码分析

我们先来看看 CopyOnWriteArrayList 的 add() 方法，其实也非常简单，就是在访问的时候加锁，拷贝出来一个副本，先操作这个副本，再把现有的数据替换为这个副本。

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
private E get(Object[] a, int index) {
    return (E) a[index];
}

/**
 * {@inheritDoc}
 *
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E get(int index) {
    return get(getArray(), index);
}
```

## 三、优点和缺点

### 1.优点

对于一些读多写少的数据，写入时复制的做法就很不错，例如配置、黑名单、物流地址等变化非常少的数据，这是一种无锁的实现。可以帮我们实现程序更高的并发。

CopyOnWriteArrayList 并发安全且性能比 Vector 好。Vector 是增删改查方法都加了synchronized 来保证同步，但是每个方法执行的时候都要去获得锁，性能就会大大下降，而 CopyOnWriteArrayList 只是在增删改上加锁，但是读不加锁，在读方面的性能就好于 Vector。

### 2.缺点

数据一致性问题。这种实现只是保证数据的**最终一致性**，不能保证数据的实时一致性；在添加到拷贝数据而还没进行替换的时候，读到的仍然是旧数据。

内存占用问题。在进行写操作的时候，内存里会同时驻扎两个对象的内存，如果对象比较大，频繁地进行替换会消耗内存，从而引发 Java 的 GC 问题，这个时候，我们应该考虑其他的容器，例如 ConcurrentHashMap。

## 四、和读写锁比较

读写锁：分为读锁和写锁，多个读锁不互斥，读锁与写锁互斥。

总之，读的时候上读锁，写的时候上写锁！ Java里面的实现：`ReentrantReadWriteLock`

```java
ReadWriteLock mylock = new ReentrantReadWriteLock(false);
```

- 读锁lock、unlock：

```java
myLock.readLock().lock();
myLock.readLock().unlock();
```

- 写锁lock、unlock:

```java
myLock.writeLock().lock();
myLock.writeLock().unlock();
```

**总结：**读写锁是遵循写写互斥、读写互斥、读读不互斥的原则，而copyOnWrite则是写写互斥、读写不互斥、读读不互斥的原则。