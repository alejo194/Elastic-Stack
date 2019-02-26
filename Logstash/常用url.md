#### elasticsearch  url查询
indexs 列表<br/>
curl 'localhost:9200/_cat/indices?v'<br/>
```bash
curl -XGET 'localhost:9200/logstash-$DATE/_search?pretty&q=fields.type:syslog'

curl -XGET 'http://localhost:9200/logstash-$DATE/_search?pretty&q=client:iphone'
```
