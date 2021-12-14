# [Linux下将一个文件压缩分包成多个小文件](https://www.cnblogs.com/anyux/p/9890446.html)

压缩分包

```
将文件test分包压缩成10M 的文件：
tar czf - test | split -b 10m - test.tar.gz
```

解压

```
将第一步分拆的多个包解压：
cat test.tar.gz* | tar -xzv
```

