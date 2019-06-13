#### 使用Elasticsearch API实现CRUD
添加索引：
PUT /lib/
{
"settings":{
    "index":{
       "number_of_shards": 5,
       "number_of_replicas": 1
    }
  }
}
PUT lib

#### 基本查询（Query查询）
##### 准备基本数据
     PUT /lib3 {
       "settings"：{
          "number_of_shards":3,
          "number_of_replicas":0
       },
       "mappings":{
         "user":{
            "properties":{
                "name":{"type":"text"},
                "address":{"type":"text"},
                "age":{"type":"integer"},
                "interests":{"type":"text"},
                "birthday":{"type":"data"}
            }
         }
       }
     }
     
     GET /lib3/user/_search?q=name:lisi
     
     GET /lib3/user/_search?q=name:zhaoliu&sort=age:desc
##### term查询和terms查询
    term query 会去倒排索引中询找确切的term,它并不知道分词器的存在，这种查询适合keyword、numeric、date.
    term: 查询某个字段里含有某个关键词的文档
    GET /lib3/user/_search/{"query":{"term":{"interests":"changge"}}}
    terms: 查询某个字段含有多个关键词的文档
    GET /lib3/user/_search/{"query":{"term":{"interests":["changge","hejiu"]}}}
##### 控制查询返回的数量
> from: 从哪一个文档开始  size：需要的个数

    GET /lib3/user/_search {"from":0,"size":2,"query":{"term":{"interests":["changge","hejiu"]}}}
    
##### 返回版本号
    GET /lib3/user/_search {"version":true,"query":{"term":{"interests":["changge","hejiu"]}}}
##### match查询
    match query: 知道分词器的存在，会对filed进行分词操作，然后在查询
    GET /lib3/user/_search {"query":{"match":{"name":"zhaoliu lisi"}}}
    GET /lib3/user/_search {"query":{"match":{"age":20}}}
    match_all: 查询所有文档
    GET /lib3/user/_search {"query":{"match_all":{}}}
    multi_match: 可以指定多个字段
    GET /lib3/user/_search {"query":{"multi_match":{"query":"lvyou","fields":["interests","name"]}}}
    match_phrase:短语匹配查询
    Elasticsearch引擎首先分析（analyze）查询字符串，从分析后的文本中构建短语查询，这意味着必须匹配短语中的所有分词，并保证各个分词的相对位置不变：
    GET /lib3/user/_search {
       "query":{
          "match_phrase":{"interests":"duanlian, shuoxiangsheng"}
       }
    }
##### 排序
    使用sort实现排序：desc降序，asc升序
    GET /lib3/user/_search {"query":{"match_all":{}},"sort":[{"age":{"order":"asc"}}]}
    
##### 指定返回的字段
 > _source、"_source":{"includes":"addr*","excludes":["name","bir*"]}
 
    GET /lib3/user/_search {"_source":["address","name"],"query":{"match":{"interests":"changge"}}}
##### 控制加载的字段  

##### fuzzy实现模糊查询
value: 查询的关键字 <br/>
boost: 查询的权值，默认值是1.0 <br/>
mim_similarity: 设置匹配的最小相似度，默认值为0.5，对于字符串，取值为0-1（包括0和1）；对于数值，取值可能大于1，对于日期型取值为1d,1m等，1d代表一天 <br/>
prefix_length: 指明区分词项的共同前缀长度，默认0 <br/>
max_expansions: 查询中的词项可以扩展的数目，默认可以无限大 <br/>

    GET /lib3/user/_seach {"query":{"fuzzy":{"interests":"chagge"}}}
    GET /lib3/user/_seach {"query":{"fuzzy":{"interest":{"value":"chagge"}}}}
    
##### 前缀匹配查询
    GET /lib3/user/_search {"query":{"match_phrase_prefix":{"name":{"query":"赵"}}}}
##### 范围查询
    range实现范围查询
    参数：from,to,include_lower,include_upper,boost
    include_lower是否包含范围的左边界，默认是true
    include_upper:是否包含范围的右边界，默认是true
    GET /lib3/user/_search {"query":{"range":{"birthday":{"from":"1990-10-10","to":"2018-05-01"}}}}
    GET /lib3/user/_search {"query":{"range":{"age":{"from":20,"to":25,"include_lower":true,"include_upper":false}}}}
##### 高亮搜索结果
    GET /lib3/user/_search {"query":{"match":{"interests":"changge"}},"highlight":{"fields":{"interests":{}}}}
    
#### Filter查询
    filter是不计算相关性的，同时可以cache,因此，filter速度要快于query
    POST /lib4/items/_bulk {"index":{"_id":1}}
    {"price":40,"itemID":"ID100123"}
    {"index":{"_id":2}}
    {"price":50,"itemID":"ID100124"}
    {"index":{"_id":3}}
    {"price":25,"itemID":"ID100125"}
    {"index":{"_id":4}}
    {"price":30,"itemID":"ID100126"}
    {"index":{"_id":5}}
    {"price":null,"itemID":"ID100127"}
##### 简单的过滤查询
    GET /lib4/items/_search {"post_filter:{"term":{"price":40}}"}
    GET /lib4/items/_search {"post_filter:{"term":{"price":[25,40]}}"}
    GET /lib4/items/_search {"post_filter:{"term":{"itemID":"ID100123"}}"}
##### bool过滤查询
    可以实现组合过滤查询
    格式：
    {"bool":{"must":[],"should":[],"must_not":[]}}
    must必须满足条件 ---and
    should:可以满足也可以不满足的条件---or
    must_not:不需要满足的条件---not
    GET /lib4/items/_search {"post_filter":{"bool":{"should":[{"term":{"price":25}},{"term":{"itemID":"id100123"}}
    ],
    "must_not":{
    "term":{"price":30}
    }
    }}}
    
##### 范围过滤
    gt >
    lt <
    gte >=
    lte <=
    GET /lib4/items/_search {"post_filter":{"range":{"price":{"gt":25,"lt":50}}}}
##### 过滤非空
    GET /lib4/items/_search {"query":{"bool":{"filter":{"exists":{"field":"price"}}}}}
    GET /lib4/items/_search {"query":{"constant_score":{"filter":{"exists":{"field":"price"}}}}}
#### 聚合查询
    (1)sum
    GET /lib4/items/_search {"size":0, "aggs":{"price_of_sum":{"sum":{"field":"price"}}}}
    (2)min
    GET /lib4/items/_search {"size":0, "aggs":{"price_of_min":{"min":{"field":"price"}}}}
    (3)max
    GET /lib4/items/_search {"size":0, "aggs":{"price_of_max":{"max":{"field":"price"}}}}
    (4)avg
    GET /lib4/items/_search {"size":0, "aggs":{"price_of_avg":{"avg":{"field":"price"}}}}
    (5)cardinality求计数
    GET /lib4/items/_search {"size":0, "aggs":{"price_of_cardi":{"cardinality":{"field":"price"}}}}
    (6)terms分组
    GET /lib4/items/_search {"size":0, "aggs":{"price_group_by":{"terms":{"field":"price"}}}}
    
    对那些有唱歌兴趣的用户按年龄分组 GET /lib3/user/_search {"query":{"match":{"interests":"changge"}},"size":0,"aggs":{
    "age_group_by":{"terms":{"field":"age","order":{"avg_of_age":"desc"}},"aggs":{"avg_of_age":{"avg":{"field":"age"}}}}}}
#### 复合查询
    将多个基本查询组合成单一查询的查询
##### 使用bool查询
    接收以下参数：
    must: 文档必须匹配这些条件才能被包含进来
    must_not: 文档必须不匹配这些条件才能被包含进来
    should: 如果满足这些语句中的任意语句，将增加_score,否则，无任何影响，它们主要用于修正每个文档的相关性得分。
    filter: 必须匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。
    相关性得分是如何组合的，每一个子查询都独自的计算文档的相关性得分。一旦他们的得分被计算出来，bool查询就将这些得分进行合并并且返回一个代表
    整个布尔操作的得分。
    
    下面的查询用于查找title字段匹配how to make millions并且不被标示为spam的文档。那些被标识为starred或在2014的文档，将比另外那些文档拥
    有更高的排名。如果两者都满足，那么它排名将更高：
    {"bool":{"must":{"match":{"title":"how to make millions"}},"must_not":{"match":{"tag":"spam"}},"should":[{
    "match":{"tag":"starred"}},{"range":{"date":{"gte":"2014-01-01"}}}]}}
    
    如果没有must语句，那么至少需要能够匹配其中一条should语句，但，如果存在至少一条must语句，则对should语句的匹配没有要求，如果我们不想
    因为文档的时间而影响得分，可以用filter语句来重写前面的例子：
    {"bool":{"must":{"match":{"title":"how to make millions"}},"must_not":{"match":{"tag":"spam"}},"should":[{
    "match":{"tag":"starred"}},{"range":{"date":{"gte":"2014-01-01"}}}]}}
    
    通过将range查询移到filter语句中，我们将它转成不评分的查询，将不再影响文档的相关性排名。由于它现在是一个不评分的查询，可以使用各种对
    filter查询有效的优化手段来提升性能。
    
    bool查询本身也可以被用做不评分的查询。简单的将它放置到filter语句中并在内部构建布尔逻辑：
    {"bool":{"must":{"match":{"title":"how to make millions"}},"must_not":{"match":{"tag":"spam"}},"should":[{
    "match":{"tag":"starred"}}]，"filter":{"bool":{"must":[{"range":{"date":{"gte":"2014-01-01"}}},{"range":{"price":
    {"lte":29.99}}}],"must_not":[{"term":{"category":"ebooks"}}]}}}}
    
##### constant_score查询
    它将一个不变的常量评分应用于所有匹配的文档。他被经常用于你只需要执行一个filter而没有其它查询（例如：评分查询）的情况下。
    {"constant_score":{"filter":{"category":"ebooks"}}}
    term查询被放置在constant_score中，转成不评分的filter，这种方式可以用来取代只有filter语句的bool查询。
    
