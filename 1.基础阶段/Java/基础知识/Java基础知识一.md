# 一、Java程序初始化顺序

对于Java程序，初始化一般遵循以下几个原则（优先级依次递减）：

（1）**静态对象（变量）优先于非静态对象（变量）初始化，其中，静态对象（变量）只初始化一次，而非静态对象（变量）可能会初始化多次**；

（2）**静态代码块优先于非静态代码块初始化**；

（3）**父类优先于子类进行初始化**；

（4）**按照成员变量的定义顺序进行初始化，即使变量定义散布于方法定义之中，它们依然在任何方法（包括构造函数）被调用之前先初始化**；

Java程序初始化工作，可以在许多不同的代码块中来完成，例如静态代码块、构造函数等，它们执行的顺序如下：

**父类静态变量 --》父类静态代码块 --》子类静态变量 --》子类静态代码块 --》父类非静态变量 --》父类非静态代码块 --》父类构造函数 --》子类非静态变量 --》子类非静态代码块 --》子类构造函数**。

```java
package com.example.demo;

/**
 * Test
 *
 * @author zhangyuewen
 * @since 2022/8/24
 **/
class Parent {
    static {
        System.out.println("Parent static block");
    }

    {
        System.out.println("Parent block");
    }

    public Parent() {
        System.out.println("Parent constructor block");
    }
}

public class Test extends Parent {
    static {
        System.out.println("Child static block");
    }

    {
        System.out.println("Child block");
    }

    public Test() {
        System.out.println("Child constructor block");
    }

    public static void main(String[] args) {
        Test test = new Test();
    }
}
```

> Parent static block
> Child static block
> Parent block
> Parent constructor block
> Child block
> Child constructor block



# 二、Clone方法

## 一、clone简介

clone 就是复制 ， 在Java语言中， clone方法被对象调用，所以会复制对象。所谓的复制对象，首先要分配一个和源对象同样大小的空间，在这个空间中创建一个新的对象。

## 二、Java中对象的创建

- 使用new操作符创建一个对象
- 使用clone方法复制一个对象

new与clone创建对象的区别
new操作符的本意是分配内存。程序执行到new操作符时， 首先去看new操作符后面的类型，根据类型分配内存，再调用构造函数，填充对象的各个域，这一步就叫对象的初始化。初始化完毕后，可以把他的引用（地址）发布到外部，在外部就可以通过引用操纵这个对象。
clone在第一步是和new相似的，都是分配内存，调用clone方法时，分配的内存和源对象一样，然后再使用源对象中对应的各个域，填充新对象的域。同样可以可以把这个新对象的引用发布到外部 。

## 三、复制对象or复制引用：

### 第一个例子（复制引用）：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219101401203.png" alt="image-20221219101401203" style="zoom:50%;" />

由上图得打印的地址值是相同的，既然地址相同，那么肯定是同一个对象。p和p1只是引用而已，他们都指向了一个相同的对象Person(23, “zhangsan”) 。 这种现象叫做“引用的复制”。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219101425689.png" alt="image-20221219101425689" style="zoom:50%;" />

### 第二个例子（复制对象）：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219101445736.png" alt="image-20221219101445736" style="zoom:50%;" />

内存分析：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219101511950.png" alt="image-20221219101511950" style="zoom:50%;" />

## 四、[深拷贝](https://so.csdn.net/so/search?q=深拷贝&spm=1001.2101.3001.7020) 浅拷贝

Person中有两个成员变量，分别是name和age， name是String类型， age是int类型

```java
public class Person  implements Cloneable{
    private int age;
    private String name;

    public Person(int age,String name){
        this.age = age;
        this.name = name;
    }

    public Person(){}

    public int getAge(){
        return age;
    }
    public String getName(){
        return name;
    }

    protected Object clone()throws CloneNotSupportedException{
        return (Person)super.clone();
    }
}

```

age是基本数据类型， 对它的拷贝直接将一个4字节的整数值拷贝过来就行。但name是String类型的， 只是一个引用， 指向一个真正的String对象，对它的拷贝有两种：

浅拷贝:直接将源对象中的name的引用值拷贝给新对象的name字段；
深拷贝：根据Person源对象中的name指向的字符串对象创建一个新的相同的字符串对象，将这个新字符串对象的引用赋给新拷贝的Person对象的name字段。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219101702213.png" alt="image-20221219101702213" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219101711090.png" alt="image-20221219101711090" style="zoom:50%;" />

### 浅拷贝

下面通过代码进行验证：Java中的clone方法是浅拷贝。

如果两个Person对象的name的地址值相同， 说明两个对象的name都指向同一个String对象， 也就是浅拷贝， 而如果两个对象的name的地址值不同， 那么就说明指向不同的String对象， 也就是在拷贝Person对象的时候， 同时拷贝了name引用的String对象， 也就是深拷贝。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219101806957.png" alt="image-20221219101806957" style="zoom:50%;" />

### 深拷贝

> 如果想要实现深拷贝，可以通过覆盖Object中的clone方法的方式。
> 要在clone对象时进行深拷贝，就要implements Clonable接口，覆盖并实现clone方法，除了调用父类中的clone方法得到新的对象， 还要将该类中的引用变量也clone出来。如果只是用Object中默认的clone方法，是浅拷贝的。

```java
import javax.rmi.PortableRemoteObject;
//深拷贝
public class demo4 {
    static class Body implements Cloneable{
        public Head head;
        public Body(){}
        public Body(Head head){
            this.head = head;
        }
        @Override
        protected Object clone()throws CloneNotSupportedException{
            Body newBody = (Body)super.clone();
            newBody.head = (Head)head.clone();
            return newBody;
        }
    }

    static class Head implements Cloneable{
        public Face face;
        public Head(){}
        public Head(Face face){
            this.face = face;
        }
        @Override
        protected Object clone()throws CloneNotSupportedException{
            Head newhead = (Head) super.clone();
            newhead.face = (Face) face.clone();
            return newhead;
        }
    }

    static class Face implements Cloneable{
        protected Object clone()throws  CloneNotSupportedException{
            return super.clone();
        }
    }

    public static void main(String[] args) throws CloneNotSupportedException{
        Body body = new Body(new Head(new Face()));
        Body body1 = (Body)body.clone();
        System.out.println("body == body1: "+(body == body1));
        System.out.println("body.head == body1.head: "+(body.head == body1.head));
        System.out.println("body.head.face == body1.head.face: "+(body.head.face == body1.head.face));
    }
}


```



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219102144995.png" alt="image-20221219102144995" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219102200327.png" alt="image-20221219102200327" style="zoom:50%;" />

要想深拷贝一个Body对象，要把Body对象内引用的其他对象都进行拷贝:

Body类实现Cloneable接口，clone方法，拷贝自己的同时拷贝它所引用的Head对象
Head类实现Cloneable接口、clone方法，拷贝自己的同时也要拷贝它所引用的Face对象
Face类也要实现Cloneable接口、clone方法，拷贝自己



# 三、反射

    1、反射介绍
            Reflection(反射) 是 Java 程序开发语言的特征之一，它允许运行中的 Java 程序对自身进行检查。被private封装的资源只能类内部访问，外部是不行的，但反射能直接操作类私有属性。反射可以在运行时获取一个类的所有信息，（包括成员变量，成员方法，构造器等），并且可以操纵类的字段、方法、构造器等部分。
            要想解剖一个类，必须先要获取到该类的字节码文件对象。而解剖使用的就是Class类中的方法。所以先要获取到每一个字节码文件对应的Class类型的对象。
    
        反射就是把java类中的各种成分映射成一个个的Java对象。
        例如：一个类有：成员变量、方法、构造方法、包等等信息，利用反射技术可以对一个类进行解剖，把一个个组成部分映射成一个个对象。（其实：一个类中这些成员方法、构造方法、在加入类中都有一个类来描述）
        加载的时候：Class对象的由来是将 .class 文件读入内存，并为之创建一个Class对象。
    
        Class类
    
        Class 类的实例表示正在运行的 Java 应用程序中的类和接口。也就是jvm中有N多的实例每个类都有该Class对象。（包括基本数据类型）
        Class 没有公共构造方法。Class 对象是在加载类时由 Java 虚拟机以及通过调用类加载器中的defineClass 方法自动构造的。也就是这不需要我们自己去处理创建，JVM已经帮我们创建好了。
    
        我们知道Spring框架可以帮我们创建和管理对象。需要对象时，我们无需自己手动new对象，直接从Spring提供的容器中的Beans获取即可。Beans底层其实就是一个Map<String,Object>，最终通过getBean(“user”)来获取。而这其中最核心的实现就是利用反射技术。   
    
        Bean
    
        1、Java面向对象，对象有方法和属性，那么就需要对象实例来调用方法和属性（即实例化）；
    
        2、凡是有方法或属性的类都需要实例化，这样才能具象化去使用这些方法和属性；
    
        3、规律：凡是子类及带有方法或属性的类都要加上注册Bean到Spring IoC的注解；（@Component , @Repository , @ Controller , @Service , @Configration）
    
        4、把Bean理解为类的代理或代言人（实际上确实是通过反射、代理来实现的），这样它就能代表类拥有该拥有的东西了
    
        5、在Spring中，你标识一个@符号，那么Spring就会来看看，并且从这里拿到一个Bean（注册）或者给出一个Bean（使用）
2.1 获取类对应的字节码的对象（三种）
① 调用某个类的对象的getClass()方法，即：对象.getClass()；

Person p = new Person();
Class clazz = p.getClass();
        注意：此处使用的是Object类中的getClass()方法，因为所有类都继承Object类，所以调用Object类中的getClass()方法来获取。

② 调用类的class属性类获取该类对应的Class对象，即：类名.class

Class clazz = Person.class;
③ 使用Class类中的forName()静态方法（最安全，性能最好）即：Class.forName(“类的全路径”)

Class clazz = Class.forName("类的全路径");
        注意：在运行期间，一个类，只有一个Class对象产生。

        三种方式常用第三种，第一种对象都有了还要反射干什么。第二种需要导入类的包，依赖太强，不导包就抛编译错误
## 2.2 常用方法

​    当我们获得了想要操作的类的Class对象后，可以通过Class类中的方法获取和查看该类中的方法和属性。

```
//获取包名、类名
clazz.getPackage().getName()//包名
clazz.getSimpleName()//类名
clazz.getName()//完整类名
 
//获取成员变量定义信息
getFields()//获取所有公开的成员变量,包括继承变量
getDeclaredFields()//获取本类定义的成员变量,包括私有,但不包括继承的变量
getField(变量名)
getDeclaredField(变量名)
 
//获取构造方法定义信息
getConstructor(参数类型列表)//获取公开的构造方法
getConstructors()//获取所有的公开的构造方法
getDeclaredConstructors()//获取所有的构造方法,包括私有
getDeclaredConstructor(int.class,String.class)
 
//获取方法定义信息
getMethods()//获取所有可见的方法,包括继承的方法
getMethod(方法名,参数类型列表)
getDeclaredMethods()//获取本类定义的的方法,包括私有,不包括继承的方法
getDeclaredMethod(方法名,int.class,String.class)
 
//反射新建实例
clazz.newInstance();//执行无参构造创建对象
clazz.newInstance(222,"韦小宝");//执行有参构造创建对象
clazz.getConstructor(int.class,String.class)//获取构造方法
 
//反射调用成员变量
clazz.getDeclaredField(变量名);//获取变量
clazz.setAccessible(true);//使私有成员允许访问
f.set(实例,值);//为指定实例的变量赋值,静态变量,第一参数给null
f.get(实例);//访问指定实例变量的值,静态变量,第一参数给null
 
//反射调用成员方法
Method m = Clazz.getDeclaredMethod(方法名,参数类型列表);
m.setAccessible(true);//使私有方法允许被调用
m.invoke(实例,参数数据);//让指定实例来执行该方法
```



# 四、Lambda表达式的语法

**基本语法:** `(parameters) -> expression 或 (parameters) ->{ statements; }`

**Lambda表达式由三部分组成：**

- 1.paramaters：类似方法中的形参列表，这里的参数是函数式接口里的参数。这里的参数类型可以明确的声明 也可不声明而由JVM隐含的推断。另外当只有一个推断类型时可以省略掉圆括号。
- 2.->：可理解为“被用于”的意思
- 3.方法体：可以是表达式也可以代码块，是函数式接口里方法的实现。代码块可返回一个值或者什么都不反 回，这里的代码块块等同于方法的方法体。如果是表达式，也可以返回一个值或者什么都不反回。

```
// 1. 不需要参数,返回值为 2
()->2
// 2. 接收一个参数(数字类型),返回其2倍的值
x->2*x
// 3. 接受2个参数(数字),并返回他们的和
(x,y) -> x+y
// 4. 接收2个int型整数,返回他们的乘积
(int x,int y) -> x * y
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)
(String s) -> System.out.print(s)
```



## 2.函数式接口

要了解`Lambda`表达式,首先需要了解什么是函数式接口，函数式接口定义：一个接口有且只有一个抽象方法 。

> **注意：**
>
> 1.如果一个接口只有一个抽象方法，那么该接口就是一个函数式接口
> 2.如果我们在某个接口上声明了@FunctionalInterface注解，那么编译器就会按照函数式接口的定义来要求该接口，这样如果有两个抽象方法，程序编译就会报错的。所以，从某种意义上来说，只要你保证你的接口 中只有一个抽象方法，你可以不加这个注解。加上就会自动进行检测的。

**定义方式：**

```
@FunctionalInterface
interface NoParameterNoReturn {
    //注意：只能有一个抽象方法
    void test();
}
```

**但是这种方式也是可以的：**

```
@FunctionalInterface
interface NoParameterNoReturn {
    void test();
 
    default void test2() {
        System.out.println("JDK1.8新特性，default默认方法可以有具体的实现");
    }
}
```

## 3. Lambda表达式的基本使用

**首先，我们实现准备好几个接口：**

```
@FunctionalInterface
interface NoParameterNoReturn {
    //注意：只能有一个抽象方法
    void test();
}
 
//无返回值一个参数
@FunctionalInterface
interface OneParameterNoReturn {
    void test(int a);
}
 
//无返回值多个参数
@FunctionalInterface
interface MoreParameterNoReturn {
    void test(int a, int b);
}
 
//有返回值无参数
@FunctionalInterface
interface NoParameterReturn {
    int test();
}
 
//有返回值一个参数
@FunctionalInterface
interface OneParameterReturn {
    int test(int a);
}
 
//有返回值多参数
@FunctionalInterface
interface MoreParameterReturn {
    int test(int a, int b);
}
```

我们在上面提到过，`Lambda`表达式本质是一个匿名函数，函数的方法是：返回值 方法名 参数列表 方法体。在，Lambda表达式中我们只需要关心：参数列表 方法体。

**具体使用见以下示例代码：**

```java
@FunctionalInterface
interface NoParameterNoReturn {
    //注意：只能有一个抽象方法
    void test();
}

//无返回值一个参数
@FunctionalInterface
interface OneParameterNoReturn {
    void test(int a);
}

//无返回值多个参数
@FunctionalInterface
interface MoreParameterNoReturn {
    void test(int a, int b);
}

//有返回值无参数
@FunctionalInterface
interface NoParameterReturn {
    int test();
}

//有返回值一个参数
@FunctionalInterface
interface OneParameterReturn {
    int test(int a);
}

//有返回值多参数
@FunctionalInterface
interface MoreParameterReturn {
    int test(int a, int b);
}


public class TestDemo2 {
    public static void main(String[] args) {

        NoParameterNoReturn noParameterNoReturn = () -> {
            System.out.println("无参数无返回值");
        };
        //test方法的主体内容在上述括号内
        noParameterNoReturn.test();


        OneParameterNoReturn oneParameterNoReturn = (int a) -> {
            System.out.println("无参数一个返回值：" + a);
        };
        oneParameterNoReturn.test(10);


        MoreParameterNoReturn moreParameterNoReturn = (int a, int b) -> {
            System.out.println("无返回值多个参数：" + a + " " + b);
        };
        moreParameterNoReturn.test(20, 30);


        NoParameterReturn noParameterReturn = () -> {
            System.out.println("有返回值无参数！");
            return 40;
        };
        //接收函数的返回值
        int ret = noParameterReturn.test();
        System.out.println(ret);

        OneParameterReturn oneParameterReturn = (int a) -> {
            System.out.println("有返回值有参数！");
            return a;
        };

        ret = oneParameterReturn.test(50);
        System.out.println(ret);


        MoreParameterReturn moreParameterReturn = (int a, int b) -> {
            System.out.println("有返回值多个参数！");
            return a + b;
        };
        ret = moreParameterReturn.test(60, 70);
        System.out.println(ret);
    }
}
```

## 4. Lambda在集合当中的使用

为了能够让Lambda和Java的集合类集更好的一起使用，集合当中，也新增了部分接口，以便与Lambda表达式对接。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219105526316.png" alt="image-20221219105526316" style="zoom:50%;" />

以上方法的作用可自行查看我们发的帮助手册。我们这里会示例一些方法的使用。注意：Collection的forEach()方法是从接口 java.lang.Iterable 拿过来的。

### Collection接口

forEach() 方法演示

**该方法在接口 Iterable 当中，原型如下：**

```
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```

**forEach()**方法表示：对容器中的每个元素执行action指定的动作

**可以看到我们的参数Consumer其实是一个函数式接口：**

```
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    /**
     * Returns a composed {@code Consumer} that performs, in sequence, this
     * operation followed by the {@code after} operation. If performing either
     * operation throws an exception, it is relayed to the caller of the
     * composed operation.  If performing this operation throws an exception,
     * the {@code after} operation will not be performed.
     *
     * @param after the operation to perform after this operation
     * @return a composed {@code Consumer} that performs in sequence this
     * operation followed by the {@code after} operation
     * @throws NullPointerException if {@code after} is null
     */
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

匿名内部类的方式：

```
public` `class` `TestDemo2 {
  ``public` `static` `void` `main(String[] args) {
    ``ArrayList<String> list = ``new` `ArrayList<>();
    ``list.add(``"Hello"``);
    ``list.add(``"bit"``);
    ``list.add(``"hello"``);
    ``list.add(``"lambda"``);
    ``list.forEach(``new` `Consumer<String>() {
      ``@Override
      ``public` `void` `accept(String s) {
        ``//简单遍历集合中的元素
        ``System.out.println(s);
      ``}
    ``});
  ``}
}
```

**我们可以修改为如下代码：**

```
public class TestDemo2 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("Hello");
        list.add("bit");
        list.add("hello");
        list.add("lambda");
     
         list.forEach((String s) -> {
            System.out.println(s);
        });
    }
}
```

**同时还可以简化代码：**

```
public class TestDemo2 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("Hello");
        list.add("bit");
        list.add("hello");
        list.add("lambda");
 
 
        list.forEach(s -> System.out.println(s));
    }
}
```

### List接口

### sort()方法的演示

**sort方法源码：**该方法根据c指定的比较规则对容器元素进行排序。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219110656072.png" alt="image-20221219110656072" style="zoom:50%;" />

**可以看到其参数是Comparator，我们点进去看下：**又是一个函数式接口

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219110629183.png" alt="image-20221219110629183" style="zoom:50%;" />

**这个接口中有一个抽象方法叫做compare方法：**

```
int compare(T o1, T o2);
```

**使用示例：**

```
public class TestDemo2 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("Hello");
        list.add("bit");
        list.add("hello");
        list.add("lambda");
 
 
        /*
        对list集合中的字符串按照长度进行排序
         */
        list.sort(new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o1.length() - o2.length();
            }
        });
 
        /*
        输出排序后最终的结果
         */
        list.forEach(s -> System.out.println(s));
    }
}
```

**修改为lambda表达式：**

```
public class TestDemo2 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("Hello");
        list.add("bit");
        list.add("hello");
        list.add("lambda");
 
 
        /*
        对list集合中的字符串按照长度进行排序
         */
        list.sort((String o1, String o2) -> {
                    return o1.length() - o2.length();
                }
        );
 
        /*
        输出排序后最终的结果:
        bit
        Hello
        hello
        lambda
         */
        list.forEach(s -> System.out.println(s));
    }
}
```

**此时还可以对代码进行简化：**

```
public class TestDemo2 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("Hello");
        list.add("bit");
        list.add("hello");
        list.add("lambda");
 
 
        /*
        对list集合中的字符串按照长度进行排序
         */
        list.sort((o1, o2) ->
                o1.length() - o2.length()
 
        );
 
        /*
        输出排序后最终的结果:
        bit
        Hello
        hello
        lambda
         */
        list.forEach(s -> System.out.println(s));
    }
}
```



# 五、多态的实现机制

多态是面向程序设计中代码重用的一个重要机制，它表示当同一个操作作用在不同对象的时候，会出现不同的行为。比如对于“+”操作，3+4对于整数是用来表示整数相加，对于字符串是表示字符串相互连接。

### 多态存在的三个必要条件

- 继承
- 重写
- 父类引用指向子类对象：Parent p = new Child();

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221219152524824.png" alt="image-20221219152524824" style="zoom:50%;" />

```java
class Shape {
    void draw() {}
}
 
class Circle extends Shape {
    void draw() {
        System.out.println("Circle.draw()");
    }
}
 
class Square extends Shape {
    void draw() {
        System.out.println("Square.draw()");
    }
}
 
class Triangle extends Shape {
    void draw() {
        System.out.println("Triangle.draw()");
    }
}
```

当使用多态方式调用方法时，首先检查父类中是否有该方法，如果没有，则编译错误；如果有，再去调用子类的同名方法。

多态的好处：可以使程序有良好的扩展，并可以对所有类的对象进行通用处理。

以下是一个多态实例的演示，详细说明请看注释：

```java
public class Test {
    public static void main(String[] args) {
      show(new Cat());  // 以 Cat 对象调用 show 方法
      show(new Dog());  // 以 Dog 对象调用 show 方法
                
      Animal a = new Cat();  // 向上转型  
      a.eat();               // 调用的是 Cat 的 eat
      Cat c = (Cat)a;        // 向下转型  
      c.work();        // 调用的是 Cat 的 work
  }  
            
    public static void show(Animal a)  {
      a.eat();  
        // 类型判断
        if (a instanceof Cat)  {  // 猫做的事情 
            Cat c = (Cat)a;  
            c.work();  
        } else if (a instanceof Dog) { // 狗做的事情 
            Dog c = (Dog)a;  
            c.work();  
        }  
    }  
}
 
abstract class Animal {  
    abstract void eat();  
}  
  
class Cat extends Animal {  
    public void eat() {  
        System.out.println("吃鱼");  
    }  
    public void work() {  
        System.out.println("抓老鼠");  
    }  
}  
  
class Dog extends Animal {  
    public void eat() {  
        System.out.println("吃骨头");  
    }  
    public void work() {  
        System.out.println("看家");  
    }  
}
```



# 六、abstract class 和interface 的异同

## 1、接口

**概念：**接口是对行为的抽象，也可以说是对类局部（行为）的抽象。

**说明：**接口中可以含有变量和方法，但是，接口中的变量会被隐式地指定为public static final。而方法会被隐式地指定为public abstract方法且只能是public abstract方法

```
interface interfaceA {
    // 接口中的变量必须进行初始化
    // 因为接口中的变量会被隐式地指定为public static final
    // 实现类中有，但是不能访问
    int a2 = 0;
    public static final int b2 = -10;
 
    // 接口中的所有方法都只能是声明
    // 方法会被隐式地指定为public abstract方法且只能是public abstract方法
    public void Fun2();
 
    public abstract void abstractFun2();
}
```

## 2、抽象类

**概念：**抽象类是对一种事物的抽象，即对类抽。抽象类是对整个类整体进行抽象，包括属性、行为。[Java抽象类](https://so.csdn.net/so/search?q=Java抽象类&spm=1001.2101.3001.7020)和Java接口一样，都用来声明一个新的类型。并且作为一个类型的等级结构的起点。

说明：

- 抽象类中不一定有抽象方法，但是有抽象方法的类一定要定义为抽象类；
- 具体类可以实例化，抽象类不可以实例化；
- 对于抽象类，如果需要添加新的方法，可以直接在抽象类中添加具体的实现，子类可以不进行变更；
- 抽象方法只有声明，没有具体的实现。抽象类是为了继承而存在的，如果你定义了一个抽象类，却不去继承它，就等于白白的创建了这个类；
- 对于一个父类，如果它的一个方法在父类中实现没有任何意义，必须根据子类的实际需求来进行不同的实现，那么就可以将这个方法声明为abstract方法，此时这个类也就成为了abstract抽象类。

```
abstract class abstractA {
    int a1;
    int b1 = -1;
    // 抽象方法必须声明，不能写实现
    public abstract void abstractFun1();
 
    // 一般方法必须写实现，不能写声明
    public void Fun1() {
        System.out.println("抽象类中的一般方法");
    }
}
```

3、区别和联系
联系：

- 都不能被实例化。

- 都可以包含抽象方法。
- 都可以有默认实现的方法（Java 8 可以用 default 关键在接口中定义默认方法）。

区别： 

- 接口主要用于对类的行为进行约束，你实现了某个接口就具有了对应的行为。抽象类主要用于代码复用，强调的是所属关系（比如说我们抽象了一个发送短信的抽象类）。

- 一个类只能继承一个类，但是可以实现多个接口。

- 接口中的成员变量只能是 public static final 类型的，不能被修改且必须有初始值，而抽象类的成员变量默认 default，可在子类中被重新定义，也可被重新赋值。

- 接口只给出方法的声明，不给出方法的实现。抽象类中可以有抽象方法和一般方法。如果是抽象方法的话，只有方法的声明。如果是一般方法的话，既有声明，也有实现。

  

# 七、break,continue,return的区别

**java关键字break、continue、return主要按三个纬度去区分。**

- ***\*作用不同\****
- ***\*结束不同\****
- ***\*紧跟不同\****

 **一、作用不同**

1、break：执行break操作，跳出所在的当前整个循环，到外层代码继续执行。

2、continue：执行continue操作，跳出本次循环，从下一个迭代继续运行循环，内层循环执行完毕，外层代码继续运行。

3、return：执行return操作，直接返回函数，结束函数执行，所有该函数体内的代码（包括循环体）都不会再执行。

**二、结束不同**

1、break：不仅可以结束其所在的循环，还可结束其外层循环，但一次只能结束一种循环。

2、continue：结束的是本次循环，将接着开始下一次循环。

3、return：同时结束其所在的循环和其外层循环。

**三、紧跟不同**

1、break：需要在break后紧跟一个标签，这个标签用于标识哪个外层循环；也可以不带参数，在循环体内，强行结束循环的执行，结束当前整个循环；总的来说：就近原则，结束当前整个循环。

2、continue：在continue后不需要加参数。

3、return：在return后需要紧跟一个返回值，用于提供给对应方法所需的返回值；也可以不带参数，不带参数就是返回空，其主要目的用于中断函数执行，返回调用函数处。



# 八、switch使用时的注意事项

```
/*
switch语句的使用注意事项：
    1、多个case后面的数据不可以重复
    2、switch后面的小括号当中只能是下列数据类型：
        基本数据类型：byte 、 short、char、int
        引用数据类型：String字符串、enum枚举
        
    3、switch语句格式可以很灵活：前后顺序可以颠倒，而且break语句还可以省略
    
    匹配哪一个case就从哪一个位置向下执行，知道遇到break或者整体结束
    

*/
public class Demo03SwitchNotice{
    public static void main(String[] args){
        int num = 2;
        switch (num){
            case 1:
            System.out.pritln("你好");
            break;
            case 2:
            System.out.println("我好");
            break;
            case 3:
            System.out.println("大家好");
            break;
            default:
            System.out.println("真的好");
            break;
            
        }
    }
}
```



# 九、Java基础的数据类型

**Java中主要有八种基本数据类型：**

**1、整型：\**byte、short、int、long\****

***\*2、字符型：\**\**char\****

***\*3、浮点型：\**\**float、double\****

***\*4、布尔型：\**\**boolean\****

这些数据类型不是对象，而是java中不同的特殊类型，这些基本类型的数据变量在声明之后就会立刻在栈上分配内存。除了这八种基本的数据类型，其他的都是引用类型，不会被分配内存空间，只会存储一个内存地址而已。



# 十、不可变类

不可变类是指类的实例一旦创建后，不能改变其成员变量的值。
与之对应的，可变类的实例创建后可以改变其成员变量的值。 

Java 中八个基本类型的包装类和 String 类都属于不可变类，而其他的大多数类都属于可变类。

效率
当一个对象是不可变的，那么需要拷贝这个对象的内容时，就不用复制它的本身而只是复制它的地址，复制地址（通常一个指针的大小）只需要很小的内存空间，具有非常高的效率。同时，对于引用该对象的其他变量也不会造成影响(字符串常量池)。

此外，不变性保证了hashCode 的唯一性，因此可以放心地进行缓存而不必每次重新计算新的哈希码。而哈希码被频繁地使用, 比如在hashMap 等容器中。将hashCode 缓存可以提高以不变类实例为key的容器的性能。

线程安全

在多线程情况下，一个可变对象的值很可能被其他进程改变，这样会造成不可预期的结果，而使用不可变对象就可以避免这种情况同时省去了同步加锁等过程，因此不可变类是线程安全的

当然，不可变类也有缺点：不可变类的每一次“改变”都会产生新的对象，因此在使用中不可避免的会产生很多垃圾

```
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
for (int i = 0; i < 20; i++) {
    Thread thread = new Thread(()->{
        try {
            sdf.parse("2021-02-02");
        } catch (ParseException e) {
            e.printStackTrace();
        }
    });
    thread.start();
}
```



## 不可变类设计

来看看String类，看看不可变类的设计要素

```java
//类声明也应该是final类型的，防止子类继承，破坏不可变性
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
       
    private final char value[];//String值，final类型不可变
 
    //用于缓存hash值，使用hashCode方法直接返回该值即可，减少了计算的工作量
    private int hash; //懒加载的
    //构造方法
    public String(char value[]) {
        //拷贝value[]，而不是直接把this.value=value；
        this.value = Arrays.copyOf(value, value.length);
    }
    //普通方法
    public String substring(int beginIndex, int endIndex) {
        int subLen = endIndex - beginIndex;
        //返回的是一个新的对象
        return new String(value, beginIndex, subLen);
    }
    //普通方法，返回一个对象属性的拷贝
    public char[] toCharArray() {
        //新创建一个数组
        char result[] = new char[value.length];
        System.arraycopy(value, 0, result, 0, value.length);
        return result;
    }
}
```

可以看出不可变类设计因素包括：

- 类被final修饰，防止子类继承。如果类可以被继承会破坏类的不可变性机制，只要继承类覆盖父类的方法并且继承类可以改变成员变量值，那么一旦子类以父类的形式出现时，不能保证当前类是否可变。
- 类中的属性都是final类型的，不能更改，还必须是private的防止引用属性的内部被更改，防止泄漏地址。
- 没有setter方法更改属性，且getter方法不能返回原本的属性或对象，应该复制一份返回，因为怕把引用泄漏给外部，导致成员中的内容被修改。如String类的toCharArray方法，不会直接返回char[] value给外部，防止value数组中的元素被更改
- 修改对象的属性时要返回新的对象，如subString方法
- 对构造器传入的值，应该是拷贝一份，而不是用原本的值，如果使用传入的参数直接赋值，则传递的只是引用，仍然可以通过外部变量改变参数的的值，从而间接修改不可变类的值





# 十一、值传递和引用传递的区别

值传递:是将实参的值复制一份给形参，形参的值相当于实参的一个副本，形参(副本)的值被改变，不会影响原来实参的值。基础类型属于值传递。
可以将值传递理解成文件的负复印，不管如何修改复印件，都不会对原件产生影响
引用传递:本质上，也是将实参的值复制一份给形参，只不过这个值比较特殊，是引用类型的对象的地址。通过形参对对象进行操作，会改变原来对象的值。

# 十二、字符串的创建和存储的机制

在Java语言中，字符串有着非常重要的作用，字符串的声明与初始化主要有如下两种情况：

1）对于String s1 = new String("abc")语句与String s2 = new String("abc")语句，存在两个引用对象s1，s2，两个内容相同的字符串对象"abc",它们在内存中的地址是不同的。简而言之，只要用到new都会生成新的对象。

2）对于String s1 = "abc"语句与String s2 ="abc",它们的内容相同，如果存在太多内容的对象都要占内存的话，就会浪费太多的空间。在JVM中存在了一个字符串池（或称String Table，字符串表，字符串哈希表，字符串常量池等），其中保存着很多的String类的对象，并且可以被共享使用，s1,s2引用的是同一个常量池中的对象。由于String采用了Flyweight的设计模式，当创建一个字符串常量时，例如String s = "abc",会首先在字符串常量池中查找是否有相同的字符串被定义，其判断依据是String类equals(Object obj)方法的返回值。

若已定义，则直接获取对其的引用，此时就不需要创建新的对象；若没有定义，先创建这个对象，然后把它放到字符串池中，再将它的引用返回，由于String类是不可变类，一旦创建好了就不能修改，因此String对象可以被共享而且不会导致程序的混乱。

具体示例：

String s = "abc"; //把"abc"放到常量区中，在编译时产生

String s ="ab" +"c";//把"ab"+"c"转换为字符串常量"abc"放在常量区

String s =new String("abc");//在运行时把"abc"放到堆里

例如：

String s1 = "abc" //在常量区里存放一个"abc"字符串对象

String s2 = "abc"//s2引用常量区中的对象，因此并不会创建新的对象

String s3 = new String("abc")//在堆中创建新的对象

String s4 = new String("abc")//在堆中又创建一个新的对象

  为了便于理解，可以把String s = new String("abc")语句的执行分为两个部分：第一是新建对象的过程，即new String("abc");第二是赋值的过程，即String s=。由于第二个过程中只是定义了一个名为s的String类型变量，将一个String类型的对象的引用赋给s，因此在这个过程中不会创建新的对象。第一个过程中 new String("abc")会调用String类型的构造函数：

public String(String orinigal){

          //body

}

在调用这个构造函数时，传入了一个字符串常量，因此语句new String("abc")也就等价于"abc"和new String 两个操作。若在字符串池中不存在"abc",则会创建一个字符串常量"abc",并将其添加到字符串池中；若存在，则不创建，然后new String()会在堆中创建一个新的对象，所以s3和s4指向的堆中不同对象，地址自然不同。

 

引申：对于String类型的变量s,赋值语句 s = null 与 s = “ ”是否相同？

对于赋值语句s=null,其中的s是一个字符串类型的引用，它不指向任何一个字符串。而赋值语句s = “ ”中的s是一个字符串类型的引用，他指向另一个字符串（这个字符串的值为“ ”，即空字符串），因此，二者是不同的。

面试题：

new String ("abc")创建了几个对象？

答案：创建了一个或者两个对象。如果常量池中原来有"abc",那么只创建一个对象；如果常量池中原来没有字符串"abc",那么就会创建两个对象。

深度解析：

当JVM遇到上述代码时，会先检索常量池中是否存在“abc”，如果不存在“abc”这个字符串，则会先在常量池中创建这个一个字符串。然后再执行new操作，会在堆内存中创建一个存储“abc”的String对象，对象的引用赋值给str2。此过程创建了2个对象。

当然，如果检索常量池时发现已经存在了对应的字符串，那么只会在堆内创建一个新的String对象，此过程只创建了1个对象。


# 十三、==，equals和hashcode的区别

‘==’
直观印象就是比较两个值是否相等

> "=="
> 是用来比较两个变量（基本类型和对象类型）的值是否相等的， 如果两个变量是基本类型的，那很容易，
> 直接比较值就可以了。如果两个变量是对象类型的，那么它还是比较值，只是它比较的是这两个对象在栈中的引用
> （即地址）。 
> 对象是放在堆中的，栈中存放的是对象的引用（地址）。由此可见'=='是对栈中的值进行比较的。
> 如果要比较堆中对象的内容是否相同，那么就要重写equals方法了。 

equals
直观印象就是比较两个某某是否相等，比较模糊。
对象在堆内存的首地址，即用来比较两个引用变量是否指向同一个对象。
object类中的equals()方法比较的也是两个对象的地址值，如果equals()相等，说明两个对象地址值也相等，当然hashcode()也就相等了.



equals方法对于字符串来说是比较内容的，而对于非字符串来说是比较其指向的对象是否相同的。 == 比较符也是比较指向的对象是否相同的也就是对象在对内存中的的首地址。 String类中重新定义了equals这个方法，而且比较的是值，而不是地址。所以是true。

# 十四、String、StringBuffer、StringBuilder和StringTokenizer的区别

# 十五、finally块中的代码什么时候被执行

问题描述：try{}里有一个return语句，那么紧跟在这个try{}后面的finally{}中的代码是否会被执行？如果会的话，什么时候被执行，在return之前还是return之后？

在Java语言的[异常处理](https://so.csdn.net/so/search?q=异常处理&spm=1001.2101.3001.7020)中，finally块的作用就是为了保证无论出现什么情况，finally块里的代码一定会被执行。由于程序执行return就意味着结束对当前函数的调用并跳出这个函数体，因此任何语句要执行都只能在return前执行（除非碰到exit函数），因此finally块里的代码也是在return之前执行的。此外，如果try-finally或者catch-finally中都有return，那么finally块中的return将会覆盖别处的return语句，最终返回到调用者那里的是finally中return的值。下面通过一个例子来说明这个问题：

```java
public class Test{
    public static int testFinally(){
        try {
            return 1;
        } catch (Exception e) {
            return 0;
        }finally{
            System.out.println("execute finally");
        }
    }
    public static void main(String[] args){
        int result = testFinally();
        System.out.println(result);
    }
}

运行结果：
execute finally
1
从上面这个例子中可以看出，在执行return语句前确实执行了finally块中的代码。紧接着，在finally块里放置个return语句，来看看到底最终返回的是哪个return语句的值，例子如下图所示：

public class Test{
    public static int testFinally(){
        try {
            return 1;
        } catch (Exception e) {
            return 0;
        }finally{
            System.out.println("execute finally");
            return 3;
        }
    }
    public static void main(String[] args){
        int result = testFinally();
        System.out.println(result);
    }
}
运行结果：
execute finally
3
从以上运行结果可以看出，当finally块中有return语句时，将会覆盖函数中其他return语句。此外，由于在一个方法内部定义的变量都存储在栈中，当这个函数结束后，其对应的栈就会被回收，此时在其方法体中定义的变量将不存在了，因此，对基本类型的数据，在finally块中改变return的值对返回值没有任何影响，而对引用类型的数据会有影响（详见 Java中的值传递与引用传递详解 ）。下面通过一个例子来说明这个问题：

public class Test{
    public static int testFinally1(){
        int result = 1;
        try {
            result = 2;
            return result;
        } catch (Exception e) {
            return 0;
        }finally{
            result = 3;
            System.out.println("execute finally1");
        }
    }
    public static StringBuffer testFinally2(){
        StringBuffer s = new StringBuffer("Hello");
        try {
            return s;
        } catch (Exception e) {
            return null;
        }finally{
            s.append(" World");
            System.out.println("execute finally2");
        }
    }
    public static void main(String[] args){
        int result = testFinally1();
        System.out.println(result);
        StringBuffer resultRef = testFinally2();
        System.out.println(resultRef);
    }
}
运行结果：
execute finally1
2
execute finally2
Hello World
  
程序在执行到return时会首先将返回值存储在一个指定的位置，其次去执行finally块，最后再返回。在方法testFinally1中调用return前，先把result的值1存储在一个指定的位置，然后再去执行finally块中的代码，此时修改result的值将不会影响到程序的返回结果。testFinally2中，在调用return前先把s存储到一个指定的位置，由于s为引用类型，因此在finally中修改s将会修改程序的返回结果
  
引申：出现在Java程序中的finally块是不是一定会被执行？
答案：不一定。

下面给出两个finally块不会被执行的例子：
1）、当程序进入try块之前就出现异常时，会直接结束，不会执行finally块中的代码，示例如下：


package com.js;
/**
 * 在try之前发生异常
 * @author jiangshuai
 */
 
public class Test{
    public static void testFinally1(){
        int result = 1/0;
        try {
            System.out.println("try block");
        } catch (Exception e) {
            System.out.println("catch block");
        }finally{
            System.out.println("finally block");
        }
    }
    public static void main(String[] args){
        testFinally1();
    }
}
运行结果：
Exception in thread "main" java.lang.ArithmeticException: / by zero
at com.js.Test.testFinally1(Test.java:9)
at com.js.Test.main(Test.java:19)
程序在执行1/0时会抛出异常，导致没有执行try块，因此finally块也就不会被执行。

  
2）、当程序在try块中强制退出时也不会去执行finally块中的代码，示例如下：


package com.js;
/**
 * 在try之前发生异常
 * @author jiangshuai
 */
 
public class Test{
    public static void testFinally1(){
        try {
            System.out.println("try block");
            System.exit(0);
        } catch (Exception e) {
            System.out.println("catch block");
        }finally{
            System.out.println("finally block");
        }
    }
    public static void main(String[] args){
        testFinally1();
    }
}
运行结果：
try block

上例在try块中通过调用System.exit(0)强制退出了程序，因此导致finally块中的代码没有被执行
```



# 十六、Volatile的作用

问：请谈谈你对volatile的理解？
答：volatile是Java虚拟机提供的轻量级的同步机制，它有３个特性：
１）**保证可见性**
２）**不保证原子性**
３）**禁止指令重排**

1、Volatile保证可见性

这本质上是个硬件问题，其根源在于，cpu的告诉缓存的读取速度远远快于主存，所以，cpu在读取一个变量的时候，会把数据先读取到缓存中，这样下次再访问同一个数据的时候就可以直接从缓存中读取，显然提高了读取的性能。而多核CPU在这样的缓存情况下，就会带来数据读取不一致的问题。

可见性是指当多个线程访问同一个变量时，如果其中一个线程修改了变量的值，那么其他线程也可以立刻看到。Volatile的实现原理是内存屏障。如果发现一个变量在其他CPU中存有副本，那么会发出信号量通知其他CPU将该副本对应的缓存设置为无效状态。

2、禁止指令重排

计算机在执行程序时，为了提高性能，编译器和处理器常常会对指令进行重排：
源代码–>编译器优化重排–>指令并行重排–>内存系统重排–>最终执行指令

单线程环境中，可以确保最终执行结果和代码顺序执行的结果一致。

但是多线程环境中，线程交替执行，由于编译器优化重排的存在，**两个线程使用的变量能否保持一致性是无法确定的，结果无法预测**。

3、不能保证原子性



