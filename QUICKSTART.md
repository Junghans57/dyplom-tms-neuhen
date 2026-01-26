# 🚀 Быстрый старт - Дипломный проект

## Для защиты проекта

### 1. Закоммитить изменения (ОБЯЗАТЕЛЬНО)
```bash
cd /Users/maksimneuhen/dyplom-tms-neuhen
git add .
git commit -m "feat: complete diploma project with documentation and improved CI/CD"
git push origin main
```

### 2. Документы для защиты
- **PRESENTATION.md** - Презентация на 20 слайдов (открыть и читать)
- **README.md** - Техническое описание проекта
- **REQUIREMENTS_CHECKLIST.md** - Соответствие требованиям
- **PROJECT_VERIFICATION_REPORT.md** - Отчет о проверке

---

## Развертывание с нуля

### Шаг 1: Инфраструктура (15-20 мин)
```bash
cd terraform
terraform init
terraform apply -auto-approve
```

### Шаг 2: Настройка kubectl (1 мин)
```bash
aws eks update-kubeconfig --region eu-central-1 --name fintech-eks
```

### Шаг 3: Приложение (5 мин)
```bash
cd ../services/fintech-transaction-platform
helm install fintech ./helm-chart --namespace fintech --create-namespace
```

### Шаг 4: Мониторинг (3 мин)
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
```

### Шаг 5: Проверка
```bash
kubectl get pods -n fintech
kubectl get service frontend-external -n fintech
```

---

## Доступ к интерфейсам

### Приложение
```bash
kubectl get service frontend-external -n fintech
# Открыть: http://<EXTERNAL-IP>
```

### Grafana
```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
# Открыть: http://localhost:3000
# Логин: admin / prom-operator
```

### Prometheus
```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090
# Открыть: http://localhost:9090
```

---

## CI/CD

### Секреты в GitHub
Добавить в Settings → Secrets:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_CHAT_ID`

### Триггеры
- **Push в любую ветку** → Lint + Test
- **Push в main** → Lint + Test + Build + Deploy + Notify

---

## Быстрые команды

### Проверить статус
```bash
kubectl get pods -n fintech
kubectl get pods -n monitoring
kubectl top nodes
kubectl top pods -n fintech
```

### Логи
```bash
kubectl logs -n fintech -l app=frontend -f
kubectl logs -n fintech -l app=loadgenerator -f
```

### Удалить все
```bash
helm uninstall fintech -n fintech
helm uninstall monitoring -n monitoring
cd terraform && terraform destroy -auto-approve
```

---

## Ключевые цифры для защиты

- **11 микросервисов** на 5 языках
- **100% соответствие** требованиям
- **~30 минут** развертывание с нуля
- **4 документа**, 2021 строка
- **Matrix CI/CD** - параллельная проверка

---

## Контакты и ресурсы

- **GitHub:** github.com/yourusername/dyplom-tms-neuhen
- **Исходный проект:** GoogleCloudPlatform/microservices-demo
- **Документация AWS EKS:** https://docs.aws.amazon.com/eks/
- **Helm Charts:** https://helm.sh/docs/

---

**Удачной защиты! 🎓**
