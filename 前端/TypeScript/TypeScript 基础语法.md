# TypeScript 基础语法

TypeScript 程序由以下几个部分组成：

- 模块
- 函数
- 变量
- 语句和表达式
- 注释

### 

## TypeScript 与面向对象



# TypeScript 联合类型

```
联合类型（Union Types）可以通过管道(|)将变量设置多种类型，赋值时可以根据设置的类型来赋值。

注意：只能赋值指定的类型，如果赋值其它类型就会报错。

创建联合类型的语法格式如下：

Type1|Type2|Type3 
实例
声明一个联合类型：

TypeScript
var val:string|number 
val = 12 
console.log("数字为 "+ val) 
val = "Runoob" 
console.log("字符串为 " + val)
编译以上代码，得到以下 JavaScript 代码：

JavaScript
var val;
val = 12;
console.log("数字为 " + val);
val = "Runoob";
console.log("字符串为 " + val);
输出结果为：

数字为 12
字符串为 Runoob
如果赋值其它类型就会报错：

var val:string|number 
val = true 
```

## 2.Typescript里 泛型中 & 是什么意思 ？

```
An intersection type combines multiple types into one. This allows you to add together existing types to get a single type that has all the features you need. For example, Person & Serializable & Loggable is a Person and Serializable and Loggable. That means an object of this type will have all members of all three types.

表示这个变量同时拥有所有类型所需要的成员，可以作为其中任何一个类型使用。
```

