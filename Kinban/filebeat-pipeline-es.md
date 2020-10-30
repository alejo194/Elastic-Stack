##### kibana 工具栏对es操作
PUT _ingest/pipeline/realtime_web-pipeline
{
    "description": "Pipeline for parsing Realtime_web access logs.",
    "processors": [
        {
            "grok": {
                "field": "message",
                "patterns": [
                    "%{TIMESTAMP_ISO8601:localtime}: {\"time\":\"%{TIMESTAMP_ISO8601:time}\",\"path\":\"%{PATH:path}\",\"query\":\"(?<query>.*)\",\"ip\":\"%{IP:ip}\",\"ips\":(?<ips>.*),\"byHost\":\"(?<byHost>.*)\",\"code\":%{NUMBER:code},\"responseTime\":%{NUMBER:responseTime},\"bytes\":(?<bytes>.*)}"
                ],
                "ignore_missing": true
            }
        },
        {
            "remove": {
                "field": "message"
            }
        },
        {
            "remove": {
                "field": "time"
            }
        },
        {
    "date": {
      "field": "localtime",
      "target_field": "@timestamp",
      "formats": ["dd/MMM/yyyy:H:m:s Z"],
        "ignore_failure": true
    }
  }
        ]
       }
