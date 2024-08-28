基础版[MobaXterm](https://so.csdn.net/so/search?q=MobaXterm&spm=1001.2101.3001.7020)有14个会话记录的上限，超过限制时，再新建的 session 将不会被存储。

可以通过一下方式进行解除限制。

基于 github 项目 

需要本地有 python3 环境。

```python
git clone https://github.com/zhangyuewen12/MobaXterm-keygen.git
cd MobaXterm-keygen
python3 MobaXterm-Keygen.py <UserName> <Version>
```

UserName 可以随意填，Version 填自己安装的版本即可（help中可以看到）。

运行命令后，会生成一个 Custom.mxtpro 文件，将该文件复制到 mobaxterm 的安装目录即可。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230207095812018.png" alt="image-20230207095812018" style="zoom:50%;" />