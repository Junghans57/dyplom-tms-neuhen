# Мониторинг и логирование

## Prometheus и Grafana

### Установка

```bash
# Добавить Helm репозиторий
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Установить kube-prometheus-stack (Prometheus + Grafana + Alertmanager)
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \
  --set grafana.adminPassword=admin123
```

### Доступ к Grafana

```bash
# Port-forward для доступа к Grafana
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80

# Открыть в браузере: http://localhost:3000
# Логин: admin
# Пароль: admin123
```

### Доступ к Prometheus

```bash
# Port-forward для доступа к Prometheus
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090

# Открыть в браузере: http://localhost:9090
```

### Создание ServiceMonitor для микросервисов

Создайте файл `servicemonitor.yaml`:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: fintech-services
  namespace: fintech
  labels:
    release: monitoring
spec:
  selector:
    matchLabels:
      app: fintech
  endpoints:
  - port: http
    interval: 30s
    path: /metrics
```

Применить:
```bash
kubectl apply -f servicemonitor.yaml
```

### Готовые дашборды Grafana

После входа в Grafana:
1. Перейти в **Dashboards** → **Browse**
2. Предустановленные дашборды:
   - **Kubernetes / Compute Resources / Cluster**
   - **Kubernetes / Compute Resources / Namespace (Pods)**
   - **Kubernetes / Compute Resources / Workload**
   - **Node Exporter / Nodes**

## EFK Stack (Elasticsearch, Fluentd, Kibana)

### Установка Elasticsearch

```bash
# Добавить Helm репозиторий
helm repo add elastic https://helm.elastic.co
helm repo update

# Установить Elasticsearch
helm install elasticsearch elastic/elasticsearch \
  --namespace logging \
  --create-namespace \
  --set replicas=1 \
  --set minimumMasterNodes=1 \
  --set resources.requests.memory=1Gi \
  --set resources.requests.cpu=500m
```

### Установка Kibana

```bash
# Установить Kibana
helm install kibana elastic/kibana \
  --namespace logging \
  --set elasticsearchHosts=http://elasticsearch-master:9200
```

### Установка Fluentd

```bash
# Добавить Helm репозиторий
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update

# Установить Fluentd
helm install fluentd fluent/fluentd \
  --namespace logging \
  --set elasticsearch.host=elasticsearch-master \
  --set elasticsearch.port=9200
```

### Доступ к Kibana

```bash
# Port-forward для доступа к Kibana
kubectl port-forward -n logging svc/kibana-kibana 5601:5601

# Открыть в браузере: http://localhost:5601
```

### Настройка индекса в Kibana

1. Открыть Kibana: http://localhost:5601
2. Перейти в **Management** → **Stack Management** → **Index Patterns**
3. Создать новый индекс: `fluentd-*`
4. Выбрать time field: `@timestamp`
5. Перейти в **Discover** для просмотра логов

## Alternative: Loki + Promtail + Grafana

### Установка Loki Stack

```bash
# Добавить Helm репозиторий
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Установить Loki Stack (Loki + Promtail + Grafana)
helm install loki grafana/loki-stack \
  --namespace logging \
  --create-namespace \
  --set grafana.enabled=true \
  --set promtail.enabled=true \
  --set loki.persistence.enabled=true \
  --set loki.persistence.size=10Gi
```

### Доступ к Grafana (Loki Stack)

```bash
# Получить пароль
kubectl get secret --namespace logging loki-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

# Port-forward
kubectl port-forward -n logging svc/loki-grafana 3000:80

# Открыть: http://localhost:3000
# Логин: admin
# Пароль: (из команды выше)
```

## Мониторинг метрик приложения

### Пример экспорта метрик (Go)

```go
import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    httpRequestsTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )
    
    httpRequestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "http_request_duration_seconds",
            Help: "HTTP request duration in seconds",
        },
        []string{"method", "endpoint"},
    )
)

func init() {
    prometheus.MustRegister(httpRequestsTotal)
    prometheus.MustRegister(httpRequestDuration)
}

// Expose metrics endpoint
http.Handle("/metrics", promhttp.Handler())
```

## Алертинг (Alertmanager)

### Создание правил алертов

Создайте файл `prometheus-rules.yaml`:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: fintech-alerts
  namespace: monitoring
spec:
  groups:
  - name: fintech
    interval: 30s
    rules:
    - alert: HighPodCPU
      expr: sum(rate(container_cpu_usage_seconds_total{namespace="fintech"}[5m])) by (pod) > 0.8
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High CPU usage on {{ $labels.pod }}"
        description: "Pod {{ $labels.pod }} has CPU usage above 80%"
    
    - alert: PodMemoryUsage
      expr: sum(container_memory_working_set_bytes{namespace="fintech"}) by (pod) / sum(container_spec_memory_limit_bytes{namespace="fintech"}) by (pod) > 0.9
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High memory usage on {{ $labels.pod }}"
        description: "Pod {{ $labels.pod }} has memory usage above 90%"
    
    - alert: PodDown
      expr: kube_pod_status_phase{namespace="fintech",phase="Running"} == 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Pod is down in fintech namespace"
        description: "Pod {{ $labels.pod }} is not running"
```

Применить:
```bash
kubectl apply -f prometheus-rules.yaml
```

### Настройка Telegram алертов

Создайте файл `alertmanager-config.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: alertmanager-monitoring-kube-prometheus-alertmanager
  namespace: monitoring
type: Opaque
stringData:
  alertmanager.yaml: |
    global:
      resolve_timeout: 5m
    
    route:
      group_by: ['alertname', 'cluster']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 12h
      receiver: 'telegram'
    
    receivers:
    - name: 'telegram'
      telegram_configs:
      - bot_token: 'YOUR_TELEGRAM_BOT_TOKEN'
        chat_id: YOUR_CHAT_ID
        parse_mode: 'HTML'
        message: |
          <b>Alert:</b> {{ .GroupLabels.alertname }}
          <b>Severity:</b> {{ .CommonLabels.severity }}
          <b>Summary:</b> {{ .CommonAnnotations.summary }}
          <b>Description:</b> {{ .CommonAnnotations.description }}
```

Применить:
```bash
kubectl apply -f alertmanager-config.yaml
kubectl rollout restart statefulset alertmanager-monitoring-kube-prometheus-alertmanager -n monitoring
```

## Проверка работы мониторинга

```bash
# Проверить статус подов мониторинга
kubectl get pods -n monitoring

# Проверить статус подов логирования
kubectl get pods -n logging

# Просмотр логов Prometheus
kubectl logs -n monitoring -l app.kubernetes.io/name=prometheus

# Просмотр логов Fluentd
kubectl logs -n logging -l app.kubernetes.io/name=fluentd

# Проверить ServiceMonitors
kubectl get servicemonitors -n fintech

# Проверить PrometheusRules
kubectl get prometheusrules -n monitoring
```

## Полезные Prometheus запросы

```promql
# CPU usage по подам
sum(rate(container_cpu_usage_seconds_total{namespace="fintech"}[5m])) by (pod)

# Memory usage по подам
sum(container_memory_working_set_bytes{namespace="fintech"}) by (pod)

# HTTP requests rate
rate(http_requests_total{namespace="fintech"}[5m])

# HTTP request latency (p95)
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket{namespace="fintech"}[5m]))

# Pod restarts
rate(kube_pod_container_status_restarts_total{namespace="fintech"}[15m])

# Available pods
kube_deployment_status_replicas_available{namespace="fintech"}
```

## Удаление компонентов мониторинга

```bash
# Удалить Prometheus и Grafana
helm uninstall monitoring -n monitoring

# Удалить EFK Stack
helm uninstall elasticsearch kibana fluentd -n logging

# Удалить Loki Stack
helm uninstall loki -n logging

# Удалить namespaces
kubectl delete namespace monitoring
kubectl delete namespace logging
```

## Рекомендации

1. **Production окружение:**
   - Увеличьте replicas Elasticsearch до 3
   - Настройте persistent volumes для данных
   - Настройте retention policy для метрик и логов
   - Используйте внешний Alertmanager для критичных алертов

2. **Безопасность:**
   - Настройте аутентификацию для Grafana
   - Используйте HTTPS для доступа к UI
   - Ограничьте доступ через NetworkPolicies
   - Храните secrets в AWS Secrets Manager

3. **Производительность:**
   - Оптимизируйте scrape intervals
   - Настройте recording rules для часто используемых запросов
   - Используйте compression для логов
   - Настройте sharding для больших объемов данных
