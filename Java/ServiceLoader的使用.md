前言
ServiceLoader是jdk6里面引进的一个特性。它用来实现SPI，一种服务发现机制。SPI的全名为Service Provider Interface，主要是应用于厂商自定义组件或插件中。
Java SPI机制的思想：我们系统里抽象的各个模块，往往有很多不同的实现方案，比如日志模块、xml解析模块、jdbc模块等方案。面向的对象的设计里，我们一般推荐模块之间基于接口编程，模块之间不对实现类进行硬编码。一旦代码里涉及具体的实现类，就违反了可拔插的原则，如果需要替换一种实现，就需要修改代码。

JDK（我）提供了一种帮第三方实现者加载服务（如数据库驱动、日志库）的便捷方式，只要第三方服务遵循约定（把类名写在/META-INF里），那当我启动时会去扫描所有jar包里符合约定的类名，再调用forName加载，但我的ClassLoader是没法加载的，那就把它加载到当前执行线程的线程上下文类加载器里，后续第三方想怎么操作（驱动实现类的static代码块）就是自己的事了。



使用示例
第一步：准备类文件
创建接口类：

```
package com.cmb.dtpframework.adapter;
import java.util.List;
public interface ISpringFactoriesAdapter {     
	List<T> loadFactories(Class<T> factoryClass, ClassLoader classLoader);    
	List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader);
}
```

创建实现类：

```
package com.cmb.dtpframework.adapter;
import org.springframework.core.io.support.SpringFactoriesLoader;import java.util.List;
public class SpringFactoriesAdapterImpl implements ISpringFactoriesAdapter{    
	@Override    
	public <T> List<T> loadFactories(Class<T> factoryClass, ClassLoader classLoader) {        
		return SpringFactoriesLoader.loadFactories(factoryClass, classLoader);    
	}    
	@Override    
	public List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {        
		return SpringFactoriesLoader.loadFactoryNames(factoryClass, classLoader);    
	}
}
```

第二步：按格式创建文件
在META-INF/ services下创建一个文件
文件名：com.cmb.dtpframework.adapter.ISpringFactoriesAdapter（要注入对象的接口类全名）
文件内容：com.cmb.dtpframework.adapter.SpringFactoriesAdapterImpl（用于指定实现类）



第三步：使用

![image-20220510205030058](/Users/zyw/Library/Application Support/typora-user-images/image-20220510205030058.png)

