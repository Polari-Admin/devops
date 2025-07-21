# 🔍 Production-Ready ELK Stack with Remote Filebeat

This repository contains a complete and production-friendly setup of the **ELK Stack (Elasticsearch, Logstash, Kibana)** with a **remote Filebeat agent** to forward logs over the network. It is designed to be modular, scalable, and secure.

---

## 🧱 Architecture

```
[ Application Servers ]     [ Filebeat Agent ]
         |                        |
         | ----> Beats/TCP ------|
                                 v
                           [ Logstash Pipeline ]
                                 |
                             [ Elasticsearch ]
                                 |
                              [ Kibana UI ]
```

---

## 📁 Folder Structure

```
elk-stack/
├── docker-compose.yml
├── elasticsearch/
│   └── elasticsearch.yml
├── logstash/
│   ├── logstash.conf
│   └── pipelines.yml
├── kibana/
│   └── kibana.yml
```

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/elk-prod-stack.git
cd elk-prod-stack
```

### 2. Launch the ELK Stack

```bash
docker-compose up -d
```

---

## ⚙️ Configuration Overview

### 🔹 Elasticsearch (`elasticsearch/elasticsearch.yml`)

```yaml
cluster.name: "elk-prod-cluster"
network.host: 0.0.0.0
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: false
```

---

### 🔹 Kibana (`kibana/kibana.yml`)

```yaml
server.host: "0.0.0.0"
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
elasticsearch.username: "elastic"
elasticsearch.password: "changeme"
```

---

### 🔹 Logstash (`logstash/pipelines.yml`)

```yaml
- pipeline.id: main
  path.config: "/usr/share/logstash/pipeline/logstash.conf"
```

---

### 🔹 Logstash Pipeline (`logstash/logstash.conf`)

```conf
input {
  beats {
    port => 5044
  }
  tcp {
    port => 5000
    codec => json_lines
  }
}

filter {
  if [type] == "nginx" {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
    date {
      match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    user => "elastic"
    password => "changeme"
    index => "logs-%{+YYYY.MM.dd}"
  }
  stdout {
    codec => rubydebug
  }
}
```

---

## 📤 Remote Filebeat Setup

### 🔹 1. Install Filebeat

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.13.4-amd64.deb
sudo dpkg -i filebeat-8.13.4-amd64.deb
```

### 🔹 2. Configure Filebeat (`/etc/filebeat/filebeat.yml`)

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access.log
      - /var/log/nginx/error.log
    fields:
      type: nginx
    fields_under_root: true

output.logstash:
  hosts: ["<logstash-server-ip>:5044"]
```

### 🔹 3. Start Filebeat

```bash
sudo filebeat modules enable nginx
sudo filebeat setup
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

---

## 📊 Accessing Kibana

Visit:

```
http://localhost:5601
```

- Username: `elastic`
- Password: `changeme`

> Create index pattern: `logs-*` with time field `@timestamp`

---

## 🛡️ Production Recommendations

| Area         | Recommendation                                            |
|--------------|------------------------------------------------------------|
| TLS          | Enable TLS for Logstash and Elasticsearch traffic         |
| Auth         | Replace default passwords & use secure secrets storage    |
| Scaling      | Use multi-node Elasticsearch for performance and HA       |
| Pipelines    | Create separate pipelines for structured logs             |
| Indexing     | Use Index Lifecycle Management (ILM)                      |
| Monitoring   | Add Metricbeat to monitor system and ELK health           |

---

## 🧪 Troubleshooting

- ❌ Kibana not loading?
  - Check `docker-compose logs kibana`
- ❌ No logs in Kibana?
  - Check Filebeat logs on the remote server
  - Ensure `output.logstash.hosts` is reachable

---

## 📄 License

MIT License — Use it freely and adapt it to your infrastructure.

---

## 🙋‍♂️ Contributions

PRs and issues are welcome for enhancements, new log formats, or dashboards.