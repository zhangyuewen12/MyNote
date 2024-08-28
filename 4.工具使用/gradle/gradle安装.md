# Gradle 安装

## 先决

已安装 JDK/JRE（版本 7 或以上），这里是 Win10 系统

在命令行输入：java -version 可查询当前电脑已安装的版本

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210730114419949.png" alt="image-20210730114419949" style="zoom:50%;" />

## 下载

从 [Gralde 官方网站](https://gradle.org/releases/)下载 Gradle 的最新发行包。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210730114503481.png" alt="image-20210730114503481" style="zoom:50%;" />

## 解压

Gradle 发行包是一个 ZIP 文件。完整的发行包包括以下内容(官方发行包有 full 完整版，也有不带源码和文档的版本，可根据需求下载。[Rover12421] 译注):

- Gradle 可执行文件
- 用户手册 (有 PDF 和 HTML 两种版本)
- DSL 参考指南
- API 手册(Javadoc 和 Groovydoc)
- 样例，包括用户手册中的例子，一些完整的构建样例和更加复杂的构建脚本
- 源代码。仅供参考使用,如果你想要自己来编译 Gradle 你需要从源代码仓库中检出发行版本源码，具体请查看 Gradle 官方主页。

## 配置环境变量

运行 gradle 必须将 GRADLE_HOME/bin 加入到你的 PATH 环境变量中。

## 测试安装

运行如下命令来检查是否安装成功.该命令会显示当前的 JVM 版本和 Gradle 版本。

```
gradle -v 
```

