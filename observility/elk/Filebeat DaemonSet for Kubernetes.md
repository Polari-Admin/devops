# Filebeat DaemonSet for Kubernetes → Kafka

این فایل شامل کانفیگ کامل **Filebeat** برای جمع‌آوری لاگ پادهای Kubernetes و ارسال مستقیم به **Kafka** است.
 قابل استفاده مستقیم در GitHub و مستندسازی

## 📁 ساختار پوشه

```
k8s-filebeat/
│
├── filebeat-config.yaml      # ConfigMap شامل filebeat.yml
├── filebeat-daemonset.yaml   # DaemonSet برای نصب روی کل کلاستر

```

📜 ConfigMap: `filebeat-config.yam`

```conf
apiVersion: v1
kind: ConfigMap
metadata:
  name: filebeat-config
  namespace: kube-system
data:
  filebeat.yml: |-
    filebeat.autodiscover:
      providers:
        - type: kubernetes
          node: ${NODE_NAME}
          hints.enabled: true
          hints.default_config:
            type: container
            paths:
              - /var/log/containers/*${data.kubernetes.container.id}.log

    processors:
      - add_kubernetes_metadata:
          in_cluster: true
      - drop_fields:
          fields: ["host", "agent", "ecs", "input"]

    output.kafka:
      hosts: ["IP_SERVER4:9092", "IP_SERVER5:9092"]
      topic: "app-logs"
      partition.round_robin:
        reachable_only: true
      required_acks: 1
      compression: gzip
      max_message_bytes: 10485760

```

---

📜 DaemonSet: `filebeat-daemonset.yaml`

```conf
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat
  namespace: kube-system
labels:
  k8s-app: filebeat
spec:
  selector:
    matchLabels:
      k8s-app: filebeat
  template:
    metadata:
      labels:
        k8s-app: filebeat
    spec:
      serviceAccountName: filebeat
      tolerations:
        - key: "node-role.kubernetes.io/master"
          operator: "Exists"
          effect: "NoSchedule"
      containers:
        - name: filebeat
          image: docker.elastic.co/beats/filebeat:8.12.0
          args: ["-c", "/etc/filebeat.yml", "-e"]
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
          volumeMounts:
            - name: config
              mountPath: /etc/filebeat.yml
              subPath: filebeat.yml
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
            - name: varlog
              mountPath: /var/log
              readOnly: true
      volumes:
        - name: config
          configMap:
            name: filebeat-config
        - name: varlibdockercontainers
          hostPath:
            path: /var/lib/docker/containers
        - name: varlog
          hostPath:
            path: /var/log

```

---

## 

