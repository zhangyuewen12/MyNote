rollover index的作用
滚动索引一般可以与索引模板结合使用，实现按一定条件自动创建索引。设置Rollover之后，满足条件后，会自动新建索引，将索引别名转向新索引。当现有的索引太久或者太大时，往往使用rollover index创建新索引。

新建索引模板，模板内容如下：

```
PUT _template/mytemplate
{
  "index_patterns": "ihrs_user_log_*",
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings": {
    "type": {
      "properties": {
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "keyword"
        }
      }
    }
  }
}
```

然后新建一个index，并设置别名为logs_write：

````
PUT </ihrs_user_log_{d}>
{
  "aliases": {
    "logs_write": {}
  }
}
设置rollover index:

POST /logs_write/_rollover 
{
  "conditions": {
    "max_age":   "7d",
    "max_docs":  1000,
    "max_size":  "5gb"
  }
}
````


当别名是logs_write并且创建了超过7天，或者有1000条数据，或者大小超过5gb之后，创建mylog-000002索引，别名logs_write随后指向了mylog-000002