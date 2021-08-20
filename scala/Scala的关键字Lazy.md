# Scala的关键字Lazy
Scala中使用关键字lazy来定义惰性变量，实现延迟加载(懒加载)。 
惰性变量只能是**不可变变量**，并且只有在调用惰性变量时，才会去实例化这个变量。  

在Java中，要实现延迟加载(懒加载)，需要自己手动实现。一般的做法是这样的

```
public class JavaLazyDemo {
    private String name;

    //初始化姓名为huangbo
    private String initName(){
        return "huangbo";
    }

    public String getName(){
        //如果name为空，进行初始化
        if(name == null){
            name = initName();
        }
        return  name;
    }

}
```

在Scala中对延迟加载这一特性提供了语法级别的支持:

```
lazy val name = initName()
```
使用lazy关键字修饰变量后，**只有在使用该变量时**，才会调用其实例化方法。也就是说在定义property=initProperty()时并不会调用initProperty()方法，**只有在后面的代码中使用变量property时才会调用initProperty()方法。**

如果不使用lazy关键字对变量修饰，那么变量property是立即实例化的:  

```
object ScalaLazyDemo {
  def init():String = {
    println("huangbo 666")
    return "huangbo"
  }

  def main(args: Array[String]): Unit = {
    val name = init();
    println("666")
    println(name)
  }
}
```
上面的property没有使用lazy关键字进行修饰，所以property是立即实例化的，调用了initName()方法进行实例化。
![](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180417200825446-205644124.png)


## 使用Lazy进行修饰

```
object ScalaLazyDemo {
  def init():String = {
    println("huangbo 666")
    return "huangbo"
  }

  def main(args: Array[String]): Unit = {
    lazy val name = init();
    println("666")
    println(name)
  }
}
```
![](https://images2018.cnblogs.com/blog/1228818/201804/1228818-20180417201020721-1174508623.png)
在声明name时，并没有立即调用实例化方法initName(),而是在使用name时，才会调用实例化方法,并且无论缩少次调用，实例化方法只会执行一次。

## 懒加载的优点
1. lazy的一个作用，是将推迟复杂的计算，直到需要计算的时候才计算，而如果不使用，则完全不会进行计算。这无疑会提高效率。
2. 除了延迟计算，懒加载也可以用于构建相互依赖或循环的数据结构。我这边再举个从stackOverFlow看到的例子：
这种情况会出现栈溢出，因为无限递归，最终会导致堆栈溢出。  

```
trait Foo { val foo: Foo }
case class Fee extends Foo { val foo = Faa() }
case class Faa extends Foo { val foo = Fee() }

println(Fee().foo)
//StackOverflowException
```

而使用了lazy关键字就不会了，**因为经过lazy关键字修饰，变量里面的内容压根就不会去调用**。

```
trait Foo { val foo: Foo }
case class Fee extends Foo { lazy val foo = Faa() }
case class Faa extends Foo { lazy val foo = Fee() }

println(Fee().foo)
//Faa()
```

