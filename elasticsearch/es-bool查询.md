# [Elasticsearch查询——布尔查询Bool Query](https://www.cnblogs.com/royfans/p/9786429.html)

Bool查询现在包括四种子句，must，filter,should,must_not。

## bool查询的使用

Bool查询对应Lucene中的BooleanQuery，它由一个或者多个子句组成，每个子句都有特定的类型。

#### must

返回的文档必须满足must子句的条件，并且参与计算分值

#### filter

返回的文档必须满足filter子句的条件。但是不会像Must一样，参与计算分值

#### should

返回的文档可能满足should子句的条件。在一个Bool查询中，如果没有must或者filter，有一个或者多个should子句，那么只要满足一个就可以返回。`minimum_should_match`参数定义了至少满足几个子句。

#### must_nout

返回的文档必须不满足must_not定义的条件。

> 如果一个查询既有filter又有should，那么至少包含一个should子句



```e
{
    "bool" : {
        "must" : {
            "term" : { "user" : "kimchy" }
        },
        "filter": {
            "term" : { "tag" : "tech" }
        },
        "must_not" : {
            "range" : {
                "age" : { "from" : 10, "to" : 20 }
            }
        },
        "should" : [
            {
                "term" : { "tag" : "wow" }
            },
            {
                "term" : { "tag" : "elasticsearch" }
            }
        ],
        "minimum_should_match" : 1,
        "boost" : 1.0
    }
}
```

