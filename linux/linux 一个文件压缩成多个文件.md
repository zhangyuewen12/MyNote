# [Linux下将一个文件压缩分包成多个小文件](https://www.cnblogs.com/anyux/p/9890446.html)

压缩分包

```
将文件test分包压缩成10M 的文件：
cat hudi-hadoop-mr.tar |split -b 18m - hudi-hadoop-mr.tar
cat hudi-flink.tar |split -b 18m - hudi-flink.tar
cat flink-1.15.2.tar |split -b 18m - flink-1.15.2.tar
```

解压

```
将第一步分拆的多个包解压：
cat test.tar.gz* > test.tar

```

