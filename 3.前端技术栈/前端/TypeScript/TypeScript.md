# TypeScript 安装

安装 typescript：

```
npm install -g typescript
```



安装完成后我们可以使用 **tsc** 命令来执行 TypeScript 的相关代码，以下是查看版本号：

```
$ tsc -v
Version 3.2.2
```

然后我们新建一个 app.ts 的文件，代码如下：

```
var message:string = "Hello World"  
console.log(message)
```



通常我们使用 **.ts** 作为 TypeScript 代码文件的扩展名。

然后执行以下命令将 TypeScript 转换为 JavaScript 代码：

```
tsc app.ts
```

![image-20220311085420476](/Users/zyw/Library/Application Support/typora-user-images/image-20220311085420476.png)

这时候在当前目录下（与 app.ts 同一目录）就会生成一个 app.js 文件，代码如下：

```
var message = "Hello World"; 
console.log(message);
```

使用 node 命令来执行 app.js 文件：

```
$ node app.js 
Hello World
```

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220311085456281.png" alt="image-20220311085456281" style="zoom:50%;" />