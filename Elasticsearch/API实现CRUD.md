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
