# [Linux下将一个文件压缩分包成多个小文件](https://www.cnblogs.com/anyux/p/9890446.html)

压缩分包

```
将文件test分包压缩成10M 的文件：
cat hudi-hadoop-mr.tar |split -b 18m - hudi-hadoop-mr.tar
cat hudi-hive-sync-bundle-0.12.0.tar |split -b 18m - hudi-hive-sync-bundle-0.12.0.tar

cat msvbcrt_aio_2018.07.31.zip |split -b 18m - msvbcrt_aio_2018.07.31.zip

cat haozip.tar |split -b 20m - haozip.tar

```

解压

```
将第一步分拆的多个包解压：
cat test.tar.gz* > test.tar
j
```

