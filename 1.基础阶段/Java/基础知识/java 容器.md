

# 一、容器框架

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220165157775.png" alt="image-20221220165157775" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220165317915.png" alt="image-20221220165317915" style="zoom:50%;" />

Java的容器主要分为2个大类，即Collection和Map。Collection代表着集合，类似数组，只保存一个数字。而Map则是映射，保留键值对两个值。
根据图，首先提一下List Queue Set Map 这四个的区别。

- List(对付顺序的好帮手): 存储的元素是有序的、可重复的。
- Set (注重独一无二的性质)：存储的元素是无序的、不可重复的。
- Queue (实现排队功能的叫号机):按特定的排队规则来确定先后顺序，存储的元素是有序的、可重复的。
- Map (用 key 来搜索的专家) :使用键值对（key-value）存储，类似于数学上的函数 y=f(x)，“x” 代表 key，“y” 代表 value，key 是无序的、不可重复的，value 是无序的、可重复的，每个键最多映射到一个值。

# 二、ArrayList、Vector、LinkedList

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220165719138.png" alt="image-20221220165719138" style="zoom:50%;" />

# 三、Map

## HashMap

### HashMap实现原理

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220171824635.png" alt="image-20221220171824635" style="zoom:50%;" />

HashMap主要是以数组和链表实现的。每个列表被称为桶。要想査找表中对象的位置， 就要先计算它的散列码， 然后与桶的总数取余， 所得到的结果就是保存这个元素的桶的索引。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220171844452.png" alt="image-20221220171844452" style="zoom:50%;" />

在 JDK 1.7 的时候，采用的是头插法

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220172012147.png" alt="image-20221220172012147" style="zoom:50%;" />

 JDK 1.8 改成了尾插法

扩容 resize 分为两步：

1）扩容：创建一个新的 Entry/Node 空数组，长度是原数组的 2 倍

2）ReHash：遍历原 Entry/Node 数组，把所有的 Entry/Node 节点重新 Hash 到新数组

为什么要 ReHash 呢？直接复制到新数组不行吗？

显然是不行的，因为数组的长度改变以后，Hash 的规则也随之改变。index 的计算公式是这样的：

index = HashCode(key) & (Length - 1)
比如说数组原来的长度（Length）是 4，Hash 出来的值是 2 ，然后数组长度翻倍了变成 16，显然 Hash 出来的值也就会变了。画个图解释下：
<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220172143063.png" alt="image-20221220172143063" style="zoom:50%;" />

### 为啥 JDK 1.7 使用头插法，JDK 1.8 之后改成尾插法了呢？

我们来看 1.7 的 resize 方法：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220174057642.png" alt="image-20221220174057642" style="zoom:50%;" />

![image-20221220174105545](/Users/zyw/Library/Application Support/typora-user-images/image-20221220174105545.png)

newTable 就是扩容后的新数组，transfer 方法是 resize 的核心，它的的功能就是 ReHash，然后将原数组中的数据迁移到新数据。我们先来把 transfer 代码简化一下，方便下文的理解：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220174129349.png" alt="image-20221220174129349" style="zoom:50%;" />

先来看看单线程情况下，正常的 resize 的过程。假设我们原来的数组容量为 2，记录数为 3，分别为：[3,A]、[7,B]、[5,C]，并且这三个 Entry 节点都落到了第二个桶里面，新数组容量会被扩容到 4。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220172548021.png" alt="image-20221220172548021" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220172558674.png" alt="image-20221220172558674" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220172611012.png" alt="image-20221220172611012" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220172639157.png" alt="image-20221220172639157" style="zoom:50%;" />

OK，那现在如果我们有两个线程 Thread1 和 Thread2，假设线程 Thread1 执行到了 transfer 方法的 Entry next = e.next 这一句，然后时间片用完了被挂起了。随后线程 Thread2 顺利执行并完成 resize 方法。于是我们有下面这个样子：
<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220172748849.png" alt="image-20221220172748849" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220172757050.png" alt="image-20221220172757050" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220172808187.png" alt="image-20221220172808187" style="zoom:50%;" />

注意，Thread1 的 e 指向了 [3,A]，next 指向了 [7,B]，而在线程 Thread2 进行 ReHash后，e 和 next 指向了线程 Thread2 重组后的链表。我们可以看到链表的顺序被反转了。

OK，这个时候线程 Thread1 被重新调度执行，先是执行 e.next = newTable[i],newTalbe[i] = e，i 就是 ReHash 后的 index 值：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220172953763.png" alt="image-20221220172953763" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220173003656.png" alt="image-20221220173003656" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220173021430.png" alt="image-20221220173021430" style="zoom:50%;" />

然后执行 `e = next`，此时 e 指向了 [7,B]

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220173203340.png" alt="image-20221220173203340" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220173221802.png" alt="image-20221220173221802" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220173231255.png" alt="image-20221220173231255" style="zoom:50%;" />

然后，线程 Thread1 继续执行。

> next  = e.next  => [3,A]
>
> e.next = newTable[i] 
>
> newTable[i] = next
>
> e = next



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220173441819.png" alt="image-20221220173441819" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220173449195.png" alt="image-20221220173449195" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220173458040.png" alt="image-20221220173458040" style="zoom:50%;" />

此时，e指向 [3,A] .OK，Thread1 再进入下一步循环，

> next = e.next  
>
> e.next = newTable[i]
>
> newTable[i] = e
>
> e =next;

执行到 `e.next = newTable[i] `，导致 [3,A].next 指向了 [7,B]，循环链表出现！！！

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220173521028.png" alt="image-20221220173521028" style="zoom:50%;" />

由于 JDK 1.7 中 HashMap 使用头插会改变链表上元素的的顺序，在旧数组向新数组转移元素的过程中修改了链表中节点的引用关系，因此 JDK 1.8 改成了尾插法，在扩容时会保持链表元素原本的顺序，避免了链表成环的问题。

## TreeMap

- 会对集合内的元素排序，可以在 new 的时候添加比较器
- 默认按照 key 进行升序排序，也可以自定义排序规则
- 底层仍是红黑树

```
TreeMap map = new TreeMap<>(new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return o2 - o1;
            }
        });
```

## HashTable

线程安全的HashMap

## LinkedHashMap

## ConcurrentHashMap

### 为什么引入ConcurrentHashMap？

1.线程不安全的HashMap
在多线程环境下，使用HashMap的put操作会引起死循环，原因是多线程会导致HashMap的Entry链表形成环形数据结构，导致Entry的next节点永远不为空，就会产生死循环获取Entry。

2.效率低下的HashTable
HashTable容器使用sychronized来保证线程安全，采取锁住整个表结构来达到同步目的，在线程竞争激烈的情况下，当一个线程访问HashTable的同步方法，其他线程也访问同步方法时，会进入阻塞或轮询状态；如线程1使用put方法时，其他线程既不能使用put方法，也不能使用get方法，效率非常低下。

3.ConcurrentHashMap的锁分段技术可提升并发访问效率
首先将数据分成一段一段地存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220183000957.png" alt="image-20221220183000957" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220183114056.png" alt="image-20221220183114056" style="zoom:50%;" />



DK8中的ConcurrentHashMap选择了与HashMap相同的Node数组+链表+红黑树结构

在锁的实现上，抛弃了原有的Segment分段锁，采用CAS+synchronized实现更加细粒度的锁。将锁的级别控制在更细粒度的哈希桶数组元素级别，只需要锁住这个链表头节点（红黑树的根节点），就不会影响其他的哈希桶数组元素的读写，大大提供了并发度。

# 四、Set

# 五、BlockingQueue

# 六、Collection和Collections的区别

1.Collection 表示一组对象，这些对象也称为 collection 的元素。一些 collection 允许有重复的元素，而另一些则不允许。一些 collection 是有序的，而另一些则是无序的。JDK 不提供此接口的任何直接 实现：它提供更具体的子接口（如 Set 和 List）实现。此接口通常用来传递 collection，并在需要最大普遍性的地方操作这些 collection。

2.Collections是Java集合框架提供的一个工具类，它的里面包含了大量的用于操作或者返回集合的静态方法，并且此类不能被实例化.

# 七、迭代器

Java Iterator（迭代器）不是一个集合，它是一种用于访问集合的方法，可用于迭代 [ArrayList](https://www.runoob.com/java/java-arraylist.html) 和 [HashSet](https://www.runoob.com/java/java-hashset.html) 等集合。

Iterator 是 Java 迭代器最简单的实现，ListIterator 是 Collection API 中的接口， 它扩展了 Iterator 接口。

迭代器 it 的两个基本操作是 next 、hasNext 和 remove。

调用 it.next() 会返回迭代器的下一个元素，并且更新迭代器的状态。

调用 it.hasNext() 用于检测集合中是否还有元素。

调用 it.remove() 将迭代器返回的元素删除。

Iterator 类位于 java.util 包中，使用前需要引入它，语法格式如下：

```
// 引入 ArrayList 和 Iterator 类
import java.util.ArrayList;
import java.util.Iterator;

public class RunoobTest {
    public static void main(String[] args) {
        ArrayList<Integer> numbers = new ArrayList<Integer>();
        numbers.add(12);
        numbers.add(8);
        numbers.add(2);
        numbers.add(23);
        Iterator<Integer> it = numbers.iterator();
        while(it.hasNext()) {
            Integer i = it.next();
            if(i < 10) {  
                it.remove();  // 删除小于 10 的元素
            }
        }
        System.out.println(numbers);
    }
}
执行以上代码，输出结果如下：

[12, 23]
```



## ConCurrentModifictionException异常

如果你使用的遍历方法为for (L i:list)或使用迭代器遍历，且在循环体中使用了List的remove操作，就会出现ConcurrentModificationException异常。

```
public class Test {
    public static void main(String[] args)  {
        ArrayList<Integer> list = new ArrayList<Integer>();
        list.add(2);
        Iterator<Integer> iterator = list.iterator();
        while(iterator.hasNext()){
            Integer integer = iterator.next();
            if(integer==2)
                list.remove(integer);
        }
    }
}

```

