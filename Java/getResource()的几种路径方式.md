**1. 前言**

在Java中获取资源的时候，经常用到getResource和getResourceAsStream，本文总结一下这两种获取资源文件的路径差异

# **2.Class.getResource(String path)**

**path不以'/'开头时，默认是从此类所在的包下取资源；**

**path以'/'开头时，则是从项目的ClassPath根下获取资源。**在这里'/'表示ClassPath的[根目录](https://so.csdn.net/so/search?q=根目录&spm=1001.2101.3001.7020)。

JDK设置这样的规则，是很好理解的，path不以'/'开头时，我们就能获取与当前类所在的路径相同的资源文件，而以'/'开头时可以获取ClassPath根下任意路径的资源。

```
public class Test
{
    public static void main(String[] args)
    {
        System.out.println(Test.class.getResource(""));
        System.out.println(Test.class.getResource("/"));
    }
}

---
file:/D:/work_space/java/bin/net/swiftlet/
file:/D:/work_space/java/bin/
```



# **3.Class.getClassLoader().getResource(String path)**

**path不能以'/'开头**，**path是指类加载器的加载范围**，在资源加载的过程中，使用的逐级向上委托的形式加载的，**'/'表示Boot ClassLoader，类加载器中的加载范围**，因为这个类加载器是C++实现的，所以加载范围为null。如下所示：

```
public class Test
{
    public static void main(String[] args)
    {
        System.out.println(Test.class.getClassLoader().getResource(""));
        System.out.println(Test.class.getClassLoader().getResource("/"));
    }
}
---
file:/D:/work_space/java/bin/
null
```

**从上面可以看出：**
class.getResource("/") == class.getClassLoader().getResource("")
其实，Class.getResource和ClassLoader.getResource本质上是一样的，都是使用ClassLoader.getResource加载资源的。下面请看一下jdk的Class源码:

```
  public java.net.URL getResource(String name)
    {
        name = resolveName(name);
        ClassLoader cl = getClassLoader0();
        if (cl==null)
        {
            // A system class.
            return ClassLoader.getSystemResource(name);
        }
        return cl.getResource(name);
    }
```

从上面就可以看才出来：Class.getResource和ClassLoader.getResource本质上是一样的。至于为什么Class.getResource(String path)中path可以'/'开头，是因为在name = resolveName(name);进行了处理：

```
 private String resolveName(String name)
    {
        if (name == null)
        {
            return name;
        }
        if (!name.startsWith("/"))
        {
            Class c = this;
            while (c.isArray()) {
                c = c.getComponentType();
            }
            String baseName = c.getName();
            int index = baseName.lastIndexOf('.');
            if (index != -1)
            {
                name = baseName.substring(0, index).replace('.', '/')
                    +"/"+name;
            }
        } else
        {//如果是以"/"开头，则去掉
            name = name.substring(1);
        }
        return name;
    }
```





#  **4.Class.getResourceAsStream(String path)**

**path不以'/'开头时，默认是指所在类的相对路径，从这个相对路径下取资源；**

**path以'/'开头时，则是从项目的ClassPath根下获取资源，就是要写相对于classpath根下的绝对路径**





```
com  
   |-github  
          |-demo  
          |    |-A.class  
          |    |-1.txt  
          |-B.class  
          |-2.txt 
```



相对路径：InputStream is= **A**.class.getResourceAsStream("**1.txt**")；

路径不是以/开头，说明这是一个相对路径，相对的是A.class这个文件，所以，这里的“1.txt”所指的正确位置是**与A.class处于同一目录下的1.txt文件**，这一文件是存在的，所引不会报错。

如果我们按相对路径的方式**通过A去加载2.txt**，则路径应该这样描述：

InputStream is= **A**.class.getResourceAsStream("**../2.txt**")；  

是的，用“.."表示上一级目录。



绝对路径：

InputStream is= **A**.class.getResourceAsStream("**/com/github/demo/1.txt**")；





#  **5.Class.getClassLoader.getResourceAsStream(String path)**

**Class.getClassLoader.getResourceAsStream(String path) ：默认则是从ClassPath根下获取，path不能以’/'开头，最终是由ClassLoader获取资源。**

如果以‘/’ 开头，则 返回的是classLoader加载器**Boot ClassLoader**的加载范围，所以返回的也是**null，所以不能以 / 开头。**

class.getResourceAsStream最终调用是ClassLoader.getResourceAsStream，这两个方法最终调用还是ClassLoader.getResource()方法加载资源。所以上面的规则也适用于此。

事例：
InputStream resourceAsStream = ClassLoader.getSystemResourceAsStream("com/github/demo/1.txt");