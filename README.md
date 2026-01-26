# Дипломный проект: Финтех-платформа для обработки транзакций

## 📋 Описание проекта

Современная микросервисная финтех-платформа для обработки транзакций, построенная на основе демонстрационного приложения Google Cloud Platform. Проект демонстрирует полный цикл DevOps/SRE практик: от разработки и контейнеризации до автоматизированного развертывания в облаке AWS с мониторингом и логированием.

### Исходный репозиторий
Проект основан на [GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo) - общедоступном репозитории с микросервисным приложением электронной коммерции.

## 🏗️ Архитектура

Платформа состоит из **11 микросервисов**, написанных на различных языках программирования и взаимодействующих через gRPC:

| Сервис | Язык | Описание |
|--------|------|----------|
| **frontend** | Go | Веб-интерфейс пользователя |
| **cartservice** | C# | Управление корзиной покупок (Redis) |
| **productcatalogservice** | Go | Каталог продуктов |
| **currencyservice** | Node.js | Конвертация валют |
| **paymentservice** | Node.js | Обработка платежей |
| **shippingservice** | Go | Расчет стоимости доставки |
| **emailservice** | Python | Отправка email-уведомлений |
| **checkoutservice** | Go | Оформление заказа |
| **recommendationservice** | Python | Рекомендации товаров |
| **adservice** | Java | Рекламные объявления |
| **loadgenerator** | Python/Locust | Генерация нагрузки |

### Схема взаимодействия
```
┌─────────────┐
│   Frontend  │
└──────┬──────┘
       │
       ├─────► ProductCatalog ──► Currency
       ├─────► Cart ──► Redis
       ├─────► Recommendation
       ├─────► Checkout ──┬──► Payment
       │                  ├──► Shipping
       │                  ├──► Email
       │                  └──► Currency
       └─────► Ad
```

## 🚀 Технологический стек

### Языки программирования
- Go, C#, Node.js, Python, Java

### Инфраструктура
- **Облако:** AWS (Amazon Web Services)
- **Оркестрация:** Kubernetes (Amazon EKS 1.30)
- **IaC:** Terraform
- **Контейнеризация:** Docker
- **Управление конфигурацией:** Helm, Kustomize

### CI/CD
- **CI/CD платформа:** GitHub Actions
- **Container Registry:** Amazon ECR
- **Deployment:** Helm

### Мониторинг и логирование
- **Мониторинг:** Prometheus + Grafana
- **Трейсинг:** OpenTelemetry Collector
- **Логирование:** EFK Stack (Elasticsearch, Fluentd, Kibana)

### Коммуникация
- **Протокол:** gRPC
- **Формат данных:** Protocol Buffers

## 📦 Структура проекта

```
dyplom-tms-neuhen/
├── terraform/                          # Infrastructure as Code
│   ├── main.tf                        # Основная конфигурация (VPC, EKS)
│   ├── variables.tf                   # Переменные
│   └── terraform.tfstate              # State файлы
├── services/fintech-transaction-platform/
│   ├── src/                           # Исходный код всех микросервисов
│   │   ├── adservice/                # Java
│   │   ├── cartservice/              # C#
│   │   ├── checkoutservice/          # Go
│   │   ├── currencyservice/          # Node.js
│   │   ├── emailservice/             # Python
│   │   ├── frontend/                 # Go
│   │   ├── loadgenerator/            # Python/Locust
│   │   ├── paymentservice/           # Node.js
│   │   ├── productcatalogservice/    # Go
│   │   ├── recommendationservice/    # Python
│   │   └── shippingservice/          # Go
│   ├── kubernetes-manifests/         # K8s манифесты
│   ├── helm-chart/                   # Helm chart
│   │   ├── templates/               # Шаблоны манифестов
│   │   ├── values.yaml              # Конфигурация
│   │   └── Chart.yaml               # Метаданные
│   ├── istio-manifests/             # Service mesh конфигурация
│   ├── kustomize/                   # Kustomize overlays
│   └── docs/                        # Документация
├── .github/
│   └── workflows/
│       └── ci-cd.yml               # CI/CD pipeline
├── PRESENTATION.md                  # Презентация для защиты
└── README.md                       # Этот файл
```

## ⚙️ Установка и развертывание

### Требования

- AWS CLI настроен с credentials
- Terraform >= 1.5.0
- kubectl
- Helm >= 3.0
- Docker (опционально, для локальной разработки)

### 1️⃣ Развертывание инфраструктуры (одной командой)

```bash
# Клонирование репозитория
git clone https://github.com/yourusername/dyplom-tms-neuhen.git
cd dyplom-tms-neuhen

# Развертывание инфраструктуры AWS (VPC + EKS)
cd terraform
terraform init
terraform apply -auto-approve

# Получение имени кластера
export CLUSTER_NAME=$(terraform output -raw cluster_name)
export AWS_REGION="eu-central-1"

# Настройка kubectl
aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
```

**⏱️ Время развертывания:** ~15-20 минут

### 2️⃣ Развертывание приложения

```bash
# Переход в директорию с Helm chart
cd ../services/fintech-transaction-platform

# Установка приложения через Helm
helm install fintech ./helm-chart \
  --namespace fintech \
  --create-namespace \
  --wait

# Проверка статуса
kubectl get pods -n fintech
kubectl get services -n fintech
```

### 3️⃣ Доступ к приложению

```bash
# Получение external IP frontend
kubectl get service frontend-external -n fintech

# Ожидание назначения LoadBalancer
# Открыть в браузере: http://<EXTERNAL-IP>
```

### 4️⃣ Мониторинг

```bash
# Установка Prometheus и Grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace

# Доступ к Grafana (port-forward)
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80

# Открыть: http://localhost:3000
# Логин: admin / prom-operator
```

### 5️⃣ Логирование (EFK Stack)

```bash
# Установка Elasticsearch
helm repo add elastic https://helm.elastic.co
helm install elasticsearch elastic/elasticsearch \
  --namespace logging \
  --create-namespace

# Установка Kibana
helm install kibana elastic/kibana --namespace logging

# Установка Fluentd
helm repo add fluent https://fluent.github.io/helm-charts
helm install fluentd fluent/fluentd --namespace logging

# Доступ к Kibana
kubectl port-forward -n logging svc/kibana-kibana 5601:5601
# Открыть: http://localhost:5601
```

## 🔄 CI/CD Pipeline

### Описание этапов

GitHub Actions pipeline автоматически выполняет следующие этапы:

#### 1. **Lint and Test** (для всех веток)
- Проверка кода линтерами
- Запуск unit-тестов
- Статический анализ кода

#### 2. **Build and Push** (только для main)
- Сборка Docker-образов для всех микросервисов
- Загрузка образов в Amazon ECR
- Тегирование по git commit SHA

#### 3. **Deploy** (только для main)
- Автоматическое развертывание в EKS через Helm
- Обновление приложения в namespace `fintech`
- Проверка успешности развертывания

#### 4. **Notification**
- Отправка уведомлений в Telegram о результате сборки и развертывания

### Настройка секретов GitHub

Необходимо добавить в GitHub Secrets:

```
AWS_ACCESS_KEY_ID          # AWS credentials
AWS_SECRET_ACCESS_KEY      # AWS credentials
TELEGRAM_BOT_TOKEN         # Token бота Telegram
TELEGRAM_CHAT_ID           # ID чата для уведомлений
```

### Триггеры pipeline

```yaml
# Автоматический запуск при:
- push в любую ветку (lint + test)
- push в main (full pipeline с deployment)
- pull request в main (lint + test)
```

### Пример workflow

```bash
# 1. Создать новую ветку
git checkout -b feature/new-service

# 2. Внести изменения
# ... code changes ...

# 3. Коммит и push
git add .
git commit -m "feat: add new feature"
git push origin feature/new-service

# ✅ Автоматически запустятся: lint + test

# 4. Merge в main через PR
gh pr create --base main --head feature/new-service

# После merge:
# ✅ Автоматически: lint + test + build + deploy + notification
```

## 📊 Мониторинг и наблюдаемость

### Prometheus метрики

- CPU/Memory использование по микросервисам
- Количество запросов (RPS)
- Latency запросов
- Ошибки и их типы
- Состояние подов и узлов

### Grafana дашборды

Предустановленные дашборды:
- **Kubernetes Cluster Overview** - общая статистика кластера
- **Pods Monitoring** - мониторинг подов
- **Node Exporter** - метрики узлов
- **Application Metrics** - метрики приложения

### Distributed Tracing

OpenTelemetry Collector собирает трейсы gRPC-запросов между микросервисами для анализа производительности и отладки.

### Log Aggregation

EFK Stack собирает логи со всех подов:
- **Elasticsearch** - хранение и индексация
- **Fluentd** - сбор и обработка логов
- **Kibana** - визуализация и поиск

## 🔒 Безопасность

### Реализованные меры

✅ **Контейнерная безопасность:**
- Security Context включен
- Non-root пользователи
- Read-only filesystem где возможно
- Resource limits для всех подов

✅ **Сетевая безопасность:**
- Network Policies для изоляции сервисов
- Private subnets для worker nodes
- NAT Gateway для исходящего трафика
- Security Groups на уровне AWS

✅ **Управление доступом:**
- RBAC в Kubernetes
- IAM роли для EKS
- Service Accounts для каждого сервиса
- Least privilege principle

✅ **Секреты:**
- GitHub Secrets для CI/CD
- Kubernetes Secrets
- AWS Secrets Manager (опционально)

## 📈 Масштабируемость

### Горизонтальное масштабирование

```yaml
# Пример HPA для frontend
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### Cluster Autoscaler

EKS автоматически масштабирует узлы:
- Min: 1 узел
- Max: 5 узлов
- Instance type: t3.medium

## 🧪 Тестирование

### Unit Tests

```bash
# Пример для Go сервисов
cd services/fintech-transaction-platform/src/frontend
go test ./...

# Пример для Python сервисов
cd services/fintech-transaction-platform/src/emailservice
pytest
```

### Integration Tests

```bash
# Запуск integration тестов
cd services/fintech-transaction-platform
./scripts/run-integration-tests.sh
```

### Load Testing

```bash
# Load Generator автоматически генерирует нагрузку
kubectl logs -n fintech -l app=loadgenerator -f
```

## 🛠️ Разработка

### Локальная разработка

```bash
# 1. Установить Docker Desktop с Kubernetes
# 2. Применить манифесты локально
kubectl apply -f services/fintech-transaction-platform/kubernetes-manifests/

# 3. Проброс порта для доступа
kubectl port-forward svc/frontend 8080:80
# Открыть: http://localhost:8080
```

### Обновление сервиса

```bash
# 1. Внести изменения в код
# 2. Собрать образ
docker build -t myrepo/frontend:v2 ./src/frontend

# 3. Загрузить в registry
docker push myrepo/frontend:v2

# 4. Обновить через Helm
helm upgrade fintech ./helm-chart \
  --set frontend.image.tag=v2 \
  --namespace fintech
```

## 🗑️ Удаление ресурсов

```bash
# 1. Удалить приложение
helm uninstall fintech -n fintech

# 2. Удалить мониторинг
helm uninstall monitoring -n monitoring

# 3. Удалить логирование
helm uninstall elasticsearch kibana fluentd -n logging

# 4. Удалить инфраструктуру
cd terraform
terraform destroy -auto-approve
```

## 📚 Документация

- [Архитектура приложения](services/fintech-transaction-platform/docs/architecture.md)
- [Руководство разработчика](services/fintech-transaction-platform/docs/development-guide.md)
- [CI/CD процессы](.github/workflows/ci-cd.yml)
- [Презентация проекта](PRESENTATION.md)

## ✅ Соответствие требованиям дипломного проекта

### Обязательные требования

| Требование | Статус | Реализация |
|------------|--------|------------|
| ✅ Fork общедоступного репозитория | ✅ Выполнено | GoogleCloudPlatform/microservices-demo |
| ✅ Автоматизация инфраструктуры (IaC) | ✅ Выполнено | Terraform: VPC, EKS, одна команда |
| ✅ CI/CD: lint при любом коммите | ✅ Выполнено | GitHub Actions |
| ✅ CI/CD: сборка и тесты | ✅ Выполнено | Docker build, unit tests |
| ✅ CI/CD: загрузка артефактов | ✅ Выполнено | Amazon ECR |
| ✅ CI/CD: deployment в main | ✅ Выполнено | Helm deploy to EKS |
| ✅ Уведомления | ✅ Выполнено | Telegram notifications |
| ✅ Документация | ✅ Выполнено | README + docs |
| ✅ Мониторинг | ✅ Выполнено | Prometheus + Grafana |

### Опциональные улучшения

| Улучшение | Статус | Реализация |
|-----------|--------|------------|
| ✅ SSL/TLS | ✅ Выполнено | AWS LoadBalancer + ACM |
| ✅ Масштабируемость | ✅ Выполнено | HPA, Cluster Autoscaler, реплики |
| ✅ Контейнеризация | ✅ Выполнено | Docker для всех сервисов |
| ✅ Kubernetes | ✅ Выполнено | Amazon EKS 1.30 |
| ✅ Различные типы тестов | ✅ Выполнено | Unit, Integration, Load testing |
| ✅ Автонастройка с нуля | ✅ Выполнено | Terraform + Helm, одна команда |
| ✅ Мониторинг инфраструктуры | ✅ Выполнено | Prometheus node_exporter |
| ✅ Log aggregation | ✅ Выполнено | EFK Stack |
| ✅ Документированный код | ✅ Выполнено | Комментарии в коде |

## 🎓 Применяемые технологии по требованиям

### Развертывание инфраструктуры
- ✅ Terraform
- ✅ AWS
- ✅ EKS
- ✅ Docker

### CI/CD
- ✅ GitHub Actions

### Оповещение
- ✅ Telegram

### Мониторинг
- ✅ Prometheus
- ✅ Grafana
- ✅ OpenTelemetry

### Логирование
- ✅ EFK Stack (Elasticsearch, Fluentd, Kibana)

## 👨‍💻 Автор

**Максим Неухен**

- Дипломный проект TMS
- Год: 2026

## 📄 Лицензия

Apache License 2.0 (унаследовано от исходного проекта)

## 🙏 Благодарности

- Google Cloud Platform за исходный репозиторий microservices-demo
- Сообщество Open Source за инструменты и документацию

---

## 🚀 Быстрый старт (TL;DR)

```bash
# 1. Клонировать репозиторий
git clone https://github.com/yourusername/dyplom-tms-neuhen.git
cd dyplom-tms-neuhen

# 2. Развернуть инфраструктуру (одной командой)
cd terraform && terraform init && terraform apply -auto-approve

# 3. Настроить kubectl
aws eks update-kubeconfig --region eu-central-1 --name fintech-eks

# 4. Развернуть приложение
cd ../services/fintech-transaction-platform
helm install fintech ./helm-chart --namespace fintech --create-namespace

# 5. Получить URL приложения
kubectl get service frontend-external -n fintech

# 🎉 Готово! Приложение доступно по external IP
```

**Время развертывания:** ~20 минут  
**Стоимость AWS:** ~$100-150/месяц (EKS + t3.medium nodes)

---

**Статус проекта:** ✅ Production Ready

**Дата последнего обновления:** 2026-01-26
