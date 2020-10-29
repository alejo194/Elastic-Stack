# Elastic-Stack

#### Elastic-Stack 学习笔记
##### ELK docker-compose.yml
```bash
version: '2.2'
services:
   elasticsearch:
     image: docker.elastic.co/elasticsearch/elasticsearch:6.5.4
     container_name: elasticsearch
     cpuset: 0,1,2,3
     cpu_count: 4
     cpu_percent: 70
     cpus: 2
     mem_limit: 4g
     memswap_limit: 2g
     mem_reservation: 512m
     ports:
       - 9200:9200
       - 9300:9300
     environment:
       - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
       - discovery.type=single-node
       - bootstrap.memory_lock=true
     ulimits:
      memlock:
        soft: -1
        hard: -1
     restart: always
     volumes:
       - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
       - ./elasticsearch/config/jvm.options:/usr/share/elasticsearch/config/jvm.options
       - esdata:/usr/share/elasticsearch/data
     #network_mode:
       #elknet
     networks:
       - elknet
     logging:
       driver: "json-file"
       options:
         max-size: "50m"
         max-file: "10"
  # elasticsearch2:
  #   image: docker.elastic.co/elasticsearch/elasticsearch:6.5.4
  #   container_name: elasticsearch2
  #   cpu_percent: 50
  #   cpus: 0.5
  #   mem_limit: 2G
  #   memswap_limit: 1G
  #   mem_reservation: 512m
  #   environment:
  #     - cluster.name=docker-cluster
  #     - bootstrap.memory_lock=true
  #     - "ES_JAVA_OPTS=-Xms2G -Xmx2G"
  #     - "discovery.zen.ping.unicast.hosts=elasticsearch"
  #   ulimits:
  #     memlock:
  #       soft: -1
  #       hard: -1
  #   volumes:
  #     - esdata2:/usr/share/elasticsearch/data
  #   networks:
  #     - elknet
  #   logging:
  #     driver: "json-file"
  #     options:
  #       max-size: "50m"
  #       max-file: "10"
   logstash:
     image: docker.elastic.co/logstash/logstash:6.5.4
     container_name: logstash
     cpuset: 0,1,2,3
     cpu_count: 4
     cpu_percent: 70
     cpus: 2
     mem_limit: 4G
     memswap_limit: 2G
     mem_reservation: 512m
     restart: always
     ports:
       - 5044:5044
       - 9600:9600
     links:
       - elasticsearch
     networks:
       - elknet
     volumes:
       - /var/maxwin/pipeline/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
       - /var/maxwin/pipeline/logstash.yml:/usr/share/logstash/config/logstash.yml
     command: ["-f", "/usr/share/logstash/pipeline/logstash.conf"]
     logging:
       driver: "json-file"
       options:
         max-size: "50m"
         max-file: "10"
   kibana:
     image: docker.elastic.co/kibana/kibana:6.5.4
     #image: kibana:5
     container_name: kibana
     #cpu_percent: 50
     #cpus: 1
     #mem_limit: 4G
     #memswap_limit: 2G
     #mem_reservation: 512m
     environment:
       SERVER_NAME: kibana
       ELASTICSEARCH_URL: http://*.*.*.*:9200
       XPACK_MONITORING_ENABLED: "false"
       OPS_INTERVAL: 60000
       #ELASTICSEARCH_USERNAME: maxwin
       #ELASTICSEARCH_PASSWORD: maxwin.maxwin
     ports:
       - 5601:5601
     networks:
       - elknet
     links:
       - elasticsearch
     depends_on:
       - elasticsearch
     logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "10"
volumes:
  esdata:
    driver: local
  #esdata2:
    #driver: local
networks:
  elknet:
```

log 管理<br/>

Beat<br/>
Logstash<br/>
Elasticsearch<br/>
Kibana<br/>
