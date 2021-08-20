

# 特定域查询语言(DSL)

Elasticsearch provides a full Query DSL (Domain Specific Language) based on JSON to define queries. Think of the Query DSL as an AST (Abstract Syntax Tree) of queries, consisting of two types of clauses:

**Leaf query clauses** (叶子查询子句)

Leaf query clauses look for a particular value in a particular field, such as the [`match`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html), [`term`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html) or [`range`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-range-query.html) queries. These queries can be used by themselves.



**Compound query clauses**（复合查询子句）

Compound query clauses wrap other leaf **or** compound queries and are used to combine multiple queries in a logical fashion (such as the [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html) or [`dis_max`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-dis-max-query.html) query), or to alter their behaviour (such as the [`constant_score`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-constant-score-query.html) query).



## Query and filter context

### Relevance scores

By default, Elasticsearch sorts matching search results by **relevance score**, which measures how well each document matches a query.

The relevance score is a positive floating point number, returned in the `_score` metadata field of the [search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html) API. The higher the `_score`, the more relevant the document. While each query type can calculate relevance scores differently, score calculation also depends on whether the query clause is run in a **query** or **filter** context.



###  Query context

In the query context, a query clause answers the question “*How well does this document match this query clause?*” Besides deciding whether or not the document matches, the query clause also calculates a relevance score in the `_score` metadata field.

Query context is in effect whenever a query clause is passed to a `query` parameter, such as the `query` parameter in the [search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#request-body-search-query) API.



###  Filter context

In a filter context, a query clause answers the question “*Does this document match this query clause?*” The answer is a simple Yes or No — no scores are calculated. Filter context is mostly used for filtering structured data, e.g.

- *Does this `timestamp` fall into the range 2015 to 2016?*
- *Is the `status` field set to `"published"`*?

Frequently used filters will be cached automatically by Elasticsearch, to speed up performance.

Filter context is in effect whenever a query clause is passed to a `filter` parameter, such as the `filter` or `must_not` parameters in the [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html) query, the `filter` parameter in the [`constant_score`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-constant-score-query.html) query, or the [`filter`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-filter-aggregation.html) aggregation.



###  Example of query and filter contexts[edit](https://github.com/elastic/elasticsearch/edit/7.13/docs/reference/query-dsl/query_filter_context.asciidoc)

Below is an example of query clauses being used in query and filter context in the `search` API. This query will match documents where all of the following conditions are met:

- The `title` field contains the word `search`.
- The `content` field contains the word `elasticsearch`.
- The `status` field contains the exact word `published`.
- The `publish_date` field contains a date from 1 Jan 2015 onwards.

```
GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }},
        { "match": { "content": "Elasticsearch" }}
      ],
      "filter": [ 
        { "term":  { "status": "published" }},
        { "range": { "publish_date": { "gte": "2015-01-01" }}}
      ]
    }
  }
}

```



