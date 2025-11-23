# Logstash Pipeline Configuration (Production Ready ELK + Kafka)

این صفحه شامل یک فایل پیکربندی کامل برای اجرای Logstash در یک معماری **۵ نودی ELK + Kafka** است.
فایل شامل بخش‌های Input، Filter و Output بوده و برای استفاده مستقیم در Docker Compose آماده شده است.

---

## 📁 ساختار پوشه

```
project-root/
│
├── docker-compose.yml
│
└── pipeline/
    └── logstash.conf
```

---

## 📜 فایل کامل: `pipeline/logstash.conf`

```conf
###########################################
# INPUT
###########################################
input {

  # Input from Filebeat (port 5044)
  beats {
    port => 5044
  }

  # Input from Kafka cluster
  kafka {
    bootstrap_servers => "IP_SERVER4:9092,IP_SERVER5:9092"
    topics => ["app-logs"]
    group_id => "logstash-group"
    consumer_threads => 4
    decorate_events => true
  }
}

###########################################
# FILTER
###########################################
filter {

  # Parse messages formatted as JSON
  json {
    source => "message"
    skip_on_invalid_json => true
  }

  # Grok pattern for standard application logs
  grok {
    match => {
      "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{DATA:service} - %{GREEDYDATA:log}"
    }
    ignore_failure => true
  }

  # GeoIP enrichment when client_ip exists
  if [client_ip] {
    geoip {
      source => "client_ip"
    }
  }

  # Cleanup unnecessary fields
  mutate {
    remove_field => ["host", "agent", "ecs", "tags"]
  }
}

###########################################
# OUTPUT
###########################################
output {

  # Send logs to Elasticsearch Ingest Nodes
  elasticsearch {
    hosts => ["http://IP_SERVER4:9201","http://IP_SERVER5:9201"]
    index => "logs-%{+YYYY.MM.dd}"
    manage_template => false
    retry_on_conflict => 3
    action => "index"
  }

  # Debug mode (optional)
  # stdout { codec => rubydebug }
}
```

---

## 🚀 نکات مهم

- IP ها را با مقادیر واقعی جایگزین کنید.
- Logstash تمام فایل‌های داخل مسیر `pipeline/` را به صورت خودکار لود می‌کند.
- برای هر نوع لاگ (nginx, apache, application) می‌توان فیلترهای اختصاصی اضافه کرد.
