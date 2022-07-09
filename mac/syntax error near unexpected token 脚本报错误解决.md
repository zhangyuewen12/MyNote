# syntax error near unexpected token 脚本报错误解决

在linux服务器上运行一个python3文件时，发现执行时就报错：[syntax](https://so.csdn.net/so/search?q=syntax&spm=1001.2101.3001.7020) error near unexpected token，仔细查找了还是没找到错误，后来发现脚本内容每行尾行都添加了^M的字符，查看方式：vi -b 打开脚本文件.

使用[vim](https://so.csdn.net/so/search?q=vim&spm=1001.2101.3001.7020) -b命令查看文件内容如下：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220307093129011.png" alt="image-20220307093129011" style="zoom:50%;" />

基本解决方法有三个：

1.用vim编辑器替换

vim -b filename

在命令模式下执行%s/^M//g进行替换

注意：上述命令行中的“^M”符，不是“^”再加上“M”，而是由“Ctrl+v”、“Ctrl+M”键生成的。

2.使用dos2unix命令进行转换

Dos2unix在有些版本的系统中默认是安装的。

在Linux中，文本文件用"\n"表示回车换行，而Windows用"\r\n"表示回车换行。所以在Linux中使用Windows的文本文件常常会出现错误。为了避免这种错误，Linux提供了两种文本格式相互转化的命令：dos2unix和unix2dos，dos2unix把"\r\n"转化成"\n"，unix2dos把"\n"转化成"\r\n"。

命令dos2unix和unix2dos的使用非常简单，格式为：dos2unix filename

如果想了解更多，可以查看手册。man dos2unix

3.使用文本处理工具

cat filename | tr -d "/r" > newfile 去掉^M生成一个新文件。