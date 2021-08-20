# Scala中的下划线使用总结
## 1. 方法转化为函数  
```
例如：
def m1(x:Int,y:Int)=x*y    
val f1=m1 _
```

## 2. 集合中的每一个元素
```
例如：
val list=List(1,2,3,4)
val list1=list.map(_ * 10)
```
## 3. 获取元组Tuple中的元素
```
例如：
 val t=("hadoop",3.14,100)
  t._1   
  t._2   
  t._3
```
## 4. 模式匹配
```
例如：
val word="hadoop"
val result =word match{
  case "hadoop" => 1    
      case "spark"  => 2
      case  _       => 0     //以上都没有匹配到才会被执行
     }
```
##5. 队列
```
val list=List(1,2,3,4)
list match{
         case List(_,_*) =>1
         case _ =>2
      }
```
**例子中的“case List(_,_*) =>1”中的下划线"_"是通配符，在模式匹配中表示可以匹配任意的一个对象，而“_*”在模式匹配中则表示可以匹配任意数目的元素（包括0个元素），那么这一整句的意思就是：如果匹配到这个变量属于List类型，并且这个队列中至少有一个任意类型的元素，那么返回1**
## 6. 导包引入的时候
```
例如：
import scala.collection.mutable._
表示引入的时候将scala.collection.mutable包下面所有的类都导入
```
## 7. 初始化变量
```
例如：
var name:String=_
//在这里，name也可以声明为null，例：var name:String=null。这里的下划线和null的作用是一样的。
var age:Int=_
//在这里，age也可以声明为0，例：var age:Int=0。这里的下划线和0的作用是一样的。
```
