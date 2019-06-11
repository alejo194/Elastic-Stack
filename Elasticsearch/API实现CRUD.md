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

##### 高亮搜索结果
    GET /lib3/user/_search {"query":{"match":{"interests":"changge"}},"highlight":{"fields":{"interests":{}}}}
    
#### Filter查询
filter是不计算相关性的，同时可以cache,因此，filter速度要快于query
