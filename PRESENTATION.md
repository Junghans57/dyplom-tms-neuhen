# Дипломный проект: Финтех-платформа для обработки транзакций

## Слайд 1: Титульный лист
**Тема:** Разработка и развертывание микросервисной финтех-платформы с использованием облачных технологий и практик DevOps/SRE

**Выполнил:** Максим Неухен  
**Учебное заведение:** TMS  
**Год:** 2026

---

## Слайд 2: Требования дипломного проекта

### Обязательные требования
✅ Fork общедоступного репозитория с микросервисами  
✅ Автоматизация инфраструктуры (IaC) - развертывание одной командой  
✅ CI/CD: lint, сборка, тесты, загрузка артефактов  
✅ Автоматический deployment в main  
✅ Уведомления о результатах сборки и развертывания  
✅ Документация  
✅ Мониторинг инфраструктуры и приложения

### Опциональные улучшения (реализовано)
✅ SSL/TLS  
✅ Масштабируемость (HPA, реплики, балансировщики)  
✅ Контейнеризация (Docker)  
✅ Kubernetes (Amazon EKS)  
✅ Различные типы тестов  
✅ Log aggregation (EFK Stack)

---

## Слайд 3: Исходный репозиторий

### GoogleCloudPlatform/microservices-demo

**Выбор репозитория:**
- Популярный open-source проект от Google Cloud Platform
- Демонстрирует микросервисную архитектуру e-commerce приложения
- 11 микросервисов на 5 языках программирования
- Используется для обучения и демонстрации cloud-native технологий

**Адаптация для проекта:**
- Создан fork репозитория
- Адаптирована инфраструктура для AWS вместо GCP
- Добавлены CI/CD пайплайны GitHub Actions
- Настроен мониторинг и логирование
- Создана полная документация

**Репозиторий:** `dyplom-tms-neuhen`

---

## Слайд 4: Архитектура системы

### Микросервисная архитектура (11 сервисов)

| Сервис | Язык | Назначение |
|--------|------|------------|
| frontend | Go | Веб-интерфейс |
| cartservice | C# | Корзина (Redis) |
| productcatalogservice | Go | Каталог товаров |
| currencyservice | Node.js | Конвертация валют |
| paymentservice | Node.js | Обработка платежей |
| shippingservice | Go | Расчет доставки |
| emailservice | Python | Email-уведомления |
| checkoutservice | Go | Оформление заказа |
| recommendationservice | Python | Рекомендации |
| adservice | Java | Реклама |
| loadgenerator | Python | Нагрузочное тестирование |

### Взаимодействие
- **Протокол:** gRPC + Protocol Buffers
- **База данных:** Redis (cartservice)
- **Балансировка:** Kubernetes Services + LoadBalancer

---

## Слайд 5: Инфраструктура AWS (IaC)

### Terraform - Infrastructure as Code

**Развертывание одной командой:**
```bash
cd terraform && terraform init && terraform apply -auto-approve
```
⏱️ **Время:** ~15-20 минут

**Создаваемые ресурсы:**

**VPC (Virtual Private Cloud):**
- CIDR: 10.0.0.0/16
- 2 Availability Zones (eu-central-1a, eu-central-1b)
- Private subnets: 10.0.1.0/24, 10.0.2.0/24
- Public subnets: 10.0.101.0/24, 10.0.102.0/24
- NAT Gateway + Internet Gateway

**EKS Cluster:**
- Kubernetes 1.30
- Managed Node Group: t3.medium (2 nodes)
- Auto-scaling: 1-5 узлов
- RBAC, IAM roles, Security Groups

**State Management:**
- Terraform state (локально или S3)
- Version control через Git

---

## Слайд 6: CI/CD Pipeline - Детальный обзор

### GitHub Actions - 4 этапа

#### 1️⃣ **Lint and Test** (для всех веток)
**Matrix strategy** - параллельная проверка всех 10 сервисов:

**Go сервисы** (frontend, checkout, catalog, shipping):
```bash
go fmt ./...     # Форматирование
go vet ./...     # Статический анализ
go test ./...    # Unit-тесты
```

**Node.js сервисы** (currency, payment):
```bash
npm install
npm run lint     # ESLint
npm test         # Jest/Mocha
```

**Python сервисы** (email, recommendation):
```bash
flake8 .         # PEP8 проверка
pylint .         # Статический анализ
pytest           # Unit-тесты
```

**Java сервисы** (ad):
```bash
./gradlew check  # Checkstyle
./gradlew test   # JUnit тесты
```

**C# сервисы** (cart):
```bash
dotnet format --verify-no-changes
dotnet test
```

---

## Слайд 7: CI/CD Pipeline - Сборка и деплой

#### 2️⃣ **Build and Push** (только main)
**Matrix strategy** - параллельная сборка 11 образов:

```bash
# Для каждого сервиса:
1. docker build -t ECR_REGISTRY/fintech-{service}:{SHA}
2. docker tag ECR_REGISTRY/fintech-{service}:latest
3. docker push в Amazon ECR
```

**Артефакты:**
- 11 Docker-образов в Amazon ECR
- Тегирование по git commit SHA + latest
- Автоматическое создание ECR репозиториев

#### 3️⃣ **Deploy** (только main)
```bash
# Helm deployment
helm upgrade --install fintech ./helm-chart \
  --namespace fintech \
  --create-namespace \
  --wait

# Проверка развертывания
kubectl get pods -n fintech
kubectl get services -n fintech
```

#### 4️⃣ **Notification**
Telegram-уведомления:
- ✅ SUCCESS / ❌ FAILED / ⏭️ SKIPPED
- Детали: Repository, Branch, Commit, Author
- Ссылка на workflow run
- Результаты всех этапов

---

## Слайд 8: Мониторинг - Prometheus + Grafana

### Установка одной командой:
```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
```

### Компоненты:
- **Prometheus** - сбор метрик (30s interval)
- **Grafana** - визуализация
- **Alertmanager** - алертинг
- **Node Exporter** - метрики узлов
- **kube-state-metrics** - метрики Kubernetes

### Метрики:
📊 **Инфраструктура:**
- CPU/Memory по узлам и подам
- Disk I/O, Network bandwidth
- Kubernetes resources (deployments, pods, services)

📊 **Приложение:**
- HTTP requests rate (RPS)
- Request latency (p50, p95, p99)
- Error rates по сервисам
- gRPC метрики

### Алерты:
- High CPU/Memory usage
- Pod restarts
- Service unavailable
- High error rate

---

## Слайд 9: Логирование - EFK Stack

### Elasticsearch + Fluentd + Kibana

**Установка:**
```bash
# Elasticsearch - хранение логов
helm install elasticsearch elastic/elasticsearch

# Fluentd - сборщик логов (DaemonSet на каждом узле)
helm install fluentd fluent/fluentd

# Kibana - UI для поиска и визуализации
helm install kibana elastic/kibana
```

### Архитектура сбора логов:
```
Pod logs → Fluentd (DaemonSet) → Elasticsearch → Kibana
```

### Возможности:
- 📝 Централизованное хранение логов всех подов
- 🔍 Full-text search по логам
- 📊 Визуализация и дашборды
- 🔔 Алерты на основе логов
- 📈 Log retention policies

### Альтернатива: Loki + Promtail
- Легковеснее EFK
- Интеграция с Grafana
- Label-based indexing

---

## Слайд 10: Масштабируемость и отказоустойчивость

### Горизонтальное масштабирование

**Horizontal Pod Autoscaler (HPA):**
```yaml
minReplicas: 2
maxReplicas: 10
metrics:
  - CPU utilization: 70%
  - Memory utilization: 80%
```

**Cluster Autoscaler:**
- Автоматическое добавление/удаление узлов EKS
- Min: 1 узел, Max: 5 узлов
- Scale-up при недостатке ресурсов
- Scale-down при low utilization

### Отказоустойчивость

**Multi-AZ deployment:**
- 2 Availability Zones
- Pod anti-affinity rules
- Автоматический failover

**Health checks:**
- Liveness probes - перезапуск зависших подов
- Readiness probes - исключение не готовых подов из балансировки
- Startup probes - для медленно стартующих сервисов

**Graceful shutdown:**
- PreStop hooks
- Завершение активных запросов
- Deregistration из Service

---

## Слайд 11: Безопасность

### Многоуровневая защита

**🔒 Сетевая безопасность:**
- Private subnets для worker nodes
- NAT Gateway для исходящего трафика
- Security Groups (firewall rules)
- Network Policies в Kubernetes

**🔒 Контейнерная безопасность:**
- Non-root пользователи в контейнерах
- Read-only filesystem где возможно
- Security Context enabled
- Resource limits (CPU/Memory)
- Image scanning (опционально)

**🔒 Управление доступом:**
- AWS IAM roles для EKS
- Kubernetes RBAC
- Service Accounts для подов
- Least privilege principle

**🔒 Секреты:**
- GitHub Secrets для CI/CD credentials
- Kubernetes Secrets для sensitive data
- AWS Secrets Manager (опционально)
- Encryption at rest

**🔒 SSL/TLS:**
- AWS LoadBalancer + ACM certificates
- HTTPS для external endpoints
- mTLS между сервисами (с Istio)

---

## Слайд 12: Результаты проекта

### ✅ Полностью выполненные требования

| Требование | Реализация | Статус |
|------------|------------|--------|
| Fork репозитория | GoogleCloudPlatform/microservices-demo | ✅ |
| IaC (одна команда) | Terraform: VPC + EKS | ✅ |
| CI/CD: lint | Matrix для 10 сервисов | ✅ |
| CI/CD: сборка | Docker build всех образов | ✅ |
| CI/CD: тесты | Go, Node, Python, Java, C# | ✅ |
| CI/CD: загрузка | Amazon ECR | ✅ |
| CI/CD: deployment | Helm to EKS | ✅ |
| Уведомления | Telegram | ✅ |
| Документация | README + MONITORING + PRESENTATION | ✅ |
| Мониторинг | Prometheus + Grafana | ✅ |
| Log aggregation | EFK Stack | ✅ |

### 📊 Метрики проекта
- **Микросервисы:** 11
- **Языки:** 5 (Go, C#, Node.js, Python, Java)
- **Docker образов:** 11
- **Terraform ресурсов:** ~30
- **GitHub Actions jobs:** 4 (lint, build, deploy, notify)
- **Время развертывания:** ~20 минут (с нуля)

---

## Слайд 13: Демонстрация CI/CD в действии

### Сценарий 1: Feature branch
```bash
git checkout -b feature/new-payment-method
# ... изменения в paymentservice ...
git commit -m "feat: add new payment method"
git push origin feature/new-payment-method
```

**GitHub Actions выполнит:**
1. ✅ Lint Node.js code
2. ✅ Run Node.js tests
3. ⏭️ Build & Push (skipped - не main)
4. ⏭️ Deploy (skipped - не main)

### Сценарий 2: Merge в main
```bash
gh pr create --base main --head feature/new-payment-method
gh pr merge
```

**GitHub Actions выполнит:**
1. ✅ Lint & Test (все 11 сервисов параллельно)
2. ✅ Build & Push (11 Docker образов в ECR)
3. ✅ Deploy (Helm upgrade в EKS)
4. ✅ Telegram notification с результатами

**Результат:**
- Новая версия приложения развернута в production
- Уведомление в Telegram
- Логи и метрики в Grafana/Kibana

---

## Слайд 14: Практическая ценность проекта

### Для бизнеса
💰 **Экономия ресурсов:**
- Автоматизация развертывания (экономия времени DevOps)
- Авто-масштабирование (оплата только за использованные ресурсы)
- Быстрый time-to-market (CI/CD)

🚀 **Надежность:**
- Multi-AZ deployment (high availability)
- Автоматическое восстановление (Kubernetes)
- Мониторинг и алерты (проактивное выявление проблем)

📈 **Масштабируемость:**
- Горизонтальное масштабирование (handle traffic spikes)
- Микросервисная архитектура (независимое масштабирование)

### Для разработчиков
🎓 **Обучение:**
- Полный стек современных технологий
- Best practices DevOps/SRE
- Production-ready решения

🔧 **Переиспользование:**
- Готовые Terraform модули
- CI/CD templates
- Helm charts
- Мониторинг конфигурации

---

## Слайд 15: Возможности для развития

### Краткосрочные улучшения (1-2 месяца)
1. **Canary deployments** - постепенное развертывание новых версий
2. **Blue-Green deployments** - zero-downtime deployments
3. **Service Mesh (Istio)** - advanced traffic management, mTLS
4. **Distributed tracing (Jaeger)** - детальный анализ запросов
5. **Terraform state в S3** - shared state для команды

### Долгосрочные улучшения (3-6 месяцев)
1. **Multi-region deployment** - глобальная доступность
2. **Disaster Recovery** - автоматическое восстановление
3. **GitOps (ArgoCD)** - declarative deployment
4. **Policy as Code (OPA)** - автоматизация compliance
5. **Cost optimization** - FinOps практики

### Безопасность
1. **Image scanning** - Trivy, Clair
2. **SIEM integration** - централизованный security monitoring
3. **Secrets rotation** - автоматическое обновление credentials
4. **Penetration testing** - регулярные security audits

---

## Слайд 16: Технологии по требованиям

### ✅ Применяемые инструменты

| Категория | Требование | Реализация |
|-----------|------------|------------|
| **IaC** | Terraform, AWS, EKS | ✅ Terraform, AWS, EKS |
| **Контейнеризация** | Docker | ✅ Docker (11 образов) |
| **CI/CD** | Jenkins / GitHub Actions | ✅ GitHub Actions |
| **Оповещение** | Email/Telegram/Slack | ✅ Telegram |
| **Мониторинг** | Prometheus, Grafana | ✅ Prometheus + Grafana |
| **Логирование** | EFK / Loki | ✅ EFK Stack |

### Дополнительно использовано:
- **Helm** - управление Kubernetes манифестами
- **Kustomize** - вариации конфигураций
- **gRPC** - межсервисное взаимодействие
- **Redis** - персистентное хранилище
- **OpenTelemetry** - observability framework

---

## Слайд 17: Выводы

### Достижения проекта

✅ **Спроектирована и реализована** полнофункциональная микросервисная платформа с 11 сервисами

✅ **Автоматизирована инфраструктура** - развертывание с нуля одной командой (Terraform)

✅ **Настроен CI/CD** - от lint и тестов до автоматического deployment с уведомлениями

✅ **Внедрен комплексный мониторинг** - метрики инфраструктуры и приложения (Prometheus + Grafana)

✅ **Настроено централизованное логирование** - EFK Stack для всех микросервисов

✅ **Обеспечена безопасность** - многоуровневая защита от сети до приложения

✅ **Реализована масштабируемость** - HPA, Cluster Autoscaler, Multi-AZ

✅ **Создана полная документация** - README, мониторинг, презентация

### Полученные навыки
- Проектирование и разработка микросервисных систем
- Infrastructure as Code (Terraform)
- Container orchestration (Kubernetes)
- CI/CD automation (GitHub Actions)
- Monitoring & Observability (Prometheus, Grafana, EFK)
- Cloud platforms (AWS, EKS)
- Security best practices

---

## Слайд 18: Заключение

### Проект демонстрирует:

🎯 **Глубокое понимание** современных DevOps/SRE практик

🎯 **Практические навыки** работы с production-grade инфраструктурой

🎯 **Способность интегрировать** множество технологий в единую систему

🎯 **Готовность к работе** в качестве DevOps/SRE инженера

### Проект полностью соответствует требованиям:
✅ Все обязательные требования выполнены  
✅ Все опциональные улучшения реализованы  
✅ Применены все рекомендуемые инструменты  
✅ Создана исчерпывающая документация  

### Статус проекта:
**✅ Production Ready** - проект готов к использованию в реальных условиях после настройки production-specific параметров безопасности и масштабирования

---

## Слайд 19: Благодарности и контакты

### Благодарности:
- **TMS** за программу обучения
- **Google Cloud Platform** за исходный репозиторий microservices-demo
- **Open Source сообщество** за инструменты и документацию
- **AWS** за облачную платформу

### Контакты:
📧 **Email:** [ваш email]  
💼 **GitHub:** github.com/yourusername/dyplom-tms-neuhen  
💬 **Telegram:** [@yourusername]

### Ресурсы проекта:
📂 **Репозиторий:** github.com/yourusername/dyplom-tms-neuhen  
📄 **Документация:** README.md, MONITORING.md  
📊 **Презентация:** PRESENTATION.md

---

## Слайд 20: Вопросы и демонстрация

### Готов ответить на вопросы по темам:

🔧 **Техническая реализация:**
- Архитектура микросервисов
- Infrastructure as Code (Terraform)
- CI/CD pipeline (GitHub Actions)

📊 **Мониторинг и наблюдаемость:**
- Prometheus и Grafana
- EFK Stack
- Алертинг

☁️ **Облачная инфраструктура:**
- AWS VPC и networking
- Amazon EKS
- Масштабирование и отказоустойчивость

🔒 **Безопасность:**
- Network Policies
- RBAC и IAM
- Secrets management

### Демонстрация:
- 💻 Живая демонстрация приложения
- 📊 Grafana дашборды
- 📝 Kibana логи
- 🔄 Запуск CI/CD pipeline

---

**Благодарю за внимание!**
