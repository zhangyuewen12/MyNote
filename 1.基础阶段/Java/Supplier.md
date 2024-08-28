在Java中，Supplier是一个函数式接口，它并不接受任何参数，但返回一个值。它的定义如下：

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

Supplier接口中只包含一个抽象方法`get()`，该方法不接受任何参数，但返回类型为T的值。由于Supplier接口只有一个抽象方法，因此它是一个函数式接口，可以使用lambda表达式或方法引用来创建Supplier的实例。

Supplier接口通常用于延迟计算或产生值的场景。它可以提供一个方法来生成一些值，而不需要在调用时立即计算，这样可以延迟计算或将计算委托给调用方，从而实现懒加载的效果。

在Java中，Supplier接口的使用非常广泛，特别是在函数式编程、流式操作和并发编程中。例如，Stream API中的一些方法（如`Stream.generate(Supplier<T>)`和`Stream.of(T...)`）接受Supplier作为参数来生成元素。

下面是一个简单的示例，演示了如何使用Supplier来生成随机数：

```java
import java.util.Random;
import java.util.function.Supplier;

public class SupplierExample {
    public static void main(String[] args) {
        // 使用lambda表达式创建一个Supplier
        Supplier<Integer> randomNumberSupplier = () -> new Random().nextInt(100);

        // 使用Supplier生成随机数
        int randomNumber = randomNumberSupplier.get();
        System.out.println("Random number: " + randomNumber);
    }
}
```

在这个示例中，我们通过lambda表达式创建了一个Supplier，它会生成一个0到99之间的随机数。然后，我们调用了Supplier的`get()`方法来获取随机数的值，并打印出来。

总的来说，Supplier是一个非常有用的函数式接口，它可以用于延迟计算、生成值、提供默认值等场景，在Java的函数式编程和流式操作中经常被使用到。