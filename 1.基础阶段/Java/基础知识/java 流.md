

# 一、输入输出流

## java IO流的实现机制

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220103518146.png" alt="image-20221220103518146" style="zoom:50%;" />

## 序列化

Serialization(序列化)：将java对象以一连串的字节保存在磁盘文件中的过程，也可以说是保存java对象状态的过程。序列化可以将数据永久保存在磁盘上(通常保存在文件中)。

deserialization(反序列化)：将保存在磁盘文件中的java字节码重新转换成java对象称为反序列化。
```
JDK类库中的序列化步骤：

第一步：创建一个输出流对象，它可以包装一个输出流对象，如：文件输出流。

ObjectOutputStream out = new ObjectOutputStream(new fileOutputStream("E:\\JavaXuLiehua\\Student\\Student1.txt"));

 第二步：通过输出流对象的writeObject()方法写对象

out.writeObject("hollo word");

out.writeObject("happy")


JDK中反序列化操作：

第一步：创建文件输入流对象

 ObjectInputStream in = new ObjectInputStream(new fileInputStream("E:\\JavaXuLiehua\\Student\\Student1.txt"));

 第二步：调用readObject()方法

 String obj1 = (String)in.readObject();

 String obj2 = (String)in.readObject();

```

### demo

把Student类的对象序列化到txt文件（E:\\JavaXuLiehua\\Student\\Student1.txt）中，并对文件进行反序列化：

```java
import java.io.*;
import java.io.Externalizable;
/*
把student类对象序列化到文件E:\\JavaXuLiehua\\Student\\Student1.txt
 */
public class UserStudent {
    public static void main(String[] args) throws IOException {
        Student st = new Student("Tom",'M',20,3.6);         //实例化student类
        //判断Student1.txt是否创建成功
        File file = new File("E:\\JavaXuLiehua\\Student\\Student1.txt");
        if(file.exists()) {
            System.out.println("文件存在");
        }else{
            //否则创建新文件
            file.createNewFile();
        }
        try {
            //Student对象序列化过程
            FileOutputStream fos = new FileOutputStream(file);
            ObjectOutputStream oos = new ObjectOutputStream(fos);
            //调用 ObjectOutputStream 中的 writeObject() 方法 写对象
            oos.writeObject(st);
            oos.flush();        //flush方法刷新缓冲区，写字符时会用，因为字符会先进入缓冲区，将内存中的数据立刻写出
            fos.close();
            oos.close();
 
            //Student对象反序列化过程
            FileInputStream fis = new FileInputStream(file);
            //创建对象输入流
            ObjectInputStream ois = new ObjectInputStream(fis);
            //读取对象
            Student st1 = (Student) ois.readObject();           //会抛出异常（类找不到异常）
            System.out.println("name = " + st1.getName());
            System.out.println("sex = " + st1.getSex());
            System.out.println("year = " + st1.getYear());
            System.out.println("gpa = " + st1.getGpa());
            ois.close();
            fis.close();
        }catch (ClassNotFoundException e){
            e.printStackTrace();
        }
    }
}
```

> name = Tom
> sex = M
> year = 20
> gpa = 3.6

transient关键字
transient关键字表示被修饰的数据不能进行序列化

```
private transient char sex;         //被transient关键字修饰，不参与序列化 运行结果如下：
```

文件存在

> name = Tom
> sex = 
> year = 20
> gpa = 3.6

此时可以看见，被transient关键字修饰的变量sex并没有被序列化，返回了空值。

### Externalizable接口实现序列化与反序列化

Externalizable接口继承Serializable接口，实现Externalizable接口需要实现readExternal()方法和writeExternal()方法，这两个方法是抽象方法，对应的是serializable接口的readObject()方法和writeObject()方法，可以理解为把serializable的两个方法抽象出来。Externalizable没有serializable的限制，static和transient关键字修饰的属性也能进行序列化。

### 注意事项：

1.如果一个类能够被序列化，那么它的子类也能后被序列化。

2.由static代表类的成员变量，transient代表对象的临时数据，因此被声明为这两种类型的变量不能够被序列化。



# 二、同步与异步、阻塞和非阻塞

1.在多线程语义下

在多线程语义下，用于描述任务的线程访问机制，同步和异步关注的是否是否可以同时被调用，阻塞和非阻塞关注的线程的状态。

同步：指代码的同步执行，一个代码块同一时间只能被一个线程访问。

异步：指代码的异步执行，多个代码块同一时间可以被多个线程访问。

阻塞：线程阻塞状态，表示线程没有挂起。

非阻塞：线程不处于阻塞状态，表示线程没有挂起。

2.在IO语境下的概念

1.同步：表示一个IO操作，在没有得到结果之前，该操作会一直等待结果。

2.异步：表示一个IO操作，不用得到返回。结果由发起者自己轮训或者IO操作的执行发起回调函数。

3.阻塞：是指发起者发起IO操作，等待时的状态。

3.非阻塞：是指发起者不会等到IO操作完成。

# 三、BIO

Java BIO 基本介绍
I/O 模型简单的理解：就是用什么样的通道进行数据的发送和接收，很大程度上决定了程序通信的性能。
Java 共支持 3 种网络编程模型 I/O 模式：BIO、NIO、AIO。
Java BIO：同步并阻塞（传统阻塞型），服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销。【简单示意图】
<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220111347504.png" alt="image-20221220111347504" style="zoom:50%;" />



# 四、NIO

Java NIO 全称 Java non-blocking IO，是指 JDK 提供的新 API。从 JDK1.4 开始，Java 提供了一系列改进的输入/输出的新特性，被统称为 NIO（即 NewIO），是同步非阻塞的。

NIO 相关类都被放在 java.nio 包及子包下，并且对原 java.io 包中的很多类进行改写。【基本案例】

NIO 有三大核心部分：Channel（通道）、Buffer（缓冲区）、Selector（选择器） 。

NIO 是面向缓冲区，或者面向块编程的。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动，这就增加了处理过程中的灵活性，使用它可以提供非阻塞式的高伸缩性网络。

Java NIO 的非阻塞模式，使一个线程从某通道发送请求或者读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。非阻塞写也是如此，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。【后面有案例说明】

通俗理解：NIO 是可以做到用一个线程来处理多个操作的。假设有 10000 个请求过来,根据实际情况，可以分配 50 或者 100 个线程来处理。不像之前的阻塞 IO 那样，非得分配 10000 个。

HTTP 2.0 使用了多路复用的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比 HTTP 1.1 大了好几个数量级。


<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220111956428.png" alt="image-20221220111956428" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221220112013492.png" alt="image-20221220112013492" style="zoom:50%;" />

- 每个 Channel 都会对应一个 Buffer。
- Selector 对应一个线程，一个线程对应多个 Channel（连接）。
- 该图反应了有三个 Channel 注册到该 Selector //程序
- 程序切换到哪个 Channel 是由事件决定的，Event 就是一个重要的概念。
- Selector 会根据不同的事件，在各个通道上切换。
- Buffer 就是一个内存块，底层是有一个数组。
- 数据的读取写入是通过 Buffer，这个和 BIO是不同的，BIO 中要么是输入流，或者是输出流，不能双向，但是 NIO 的 Buffer 是可以读也可以写，需要 flip 方法切换 Channel 是双向的，可以返回底层操作系统的情况，比如 Linux，底层的操作系统通道就是双向的

## BIO、NIO、AIO 对比表

| -        | BIO      | NIO                    | AIO        |
| -------- | -------- | ---------------------- | ---------- |
| IO模型   | 同步阻塞 | 同步非阻塞（多路复用） | 异步非阻塞 |
| 编程难度 | 简单     | 复杂                   | 复杂       |
| 可靠性   | 差       | 好                     | 好         |
| 吞吐量   | 低       | 高                     | 高         |

与NIO不同，当进行读写操作时，只须直接调用API的read或write方法即可, 这两种方法均为异步的，对于读操作而言，当有流可读取时，操作系统会将可读的流传入read方法的缓冲区,对于写操作而言，当操作系统将write方法传递的流写入完毕时，操作系统主动通知应用程序即可以理解为，read/write方法都是异步的，完成后会主动调用回调函数