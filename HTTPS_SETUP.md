# HTTPS Configuration

## Реализация SSL/TLS

Проект включает полную поддержку HTTPS для безопасного доступа к приложению.

### Компоненты

1. **NGINX Ingress Controller** - для SSL termination
2. **Self-signed SSL Certificate** - для демонстрации (в production использовать ACM или Let's Encrypt)
3. **Kubernetes Ingress** - для маршрутизации HTTPS трафика

---

## Установка HTTPS

### 1. Установка NGINX Ingress Controller

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer \
  --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-type"="nlb"
```

### 2. Создание SSL сертификата

#### Опция A: Self-signed Certificate (для демонстрации)

```bash
# Создать самоподписанный сертификат
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key \
  -out tls.crt \
  -subj "/CN=fintech.local/O=fintech"

# Создать Kubernetes Secret
kubectl create secret tls fintech-tls \
  --cert=tls.crt \
  --key=tls.key \
  -n fintech
```

#### Опция B: AWS Certificate Manager (для production)

```bash
# 1. Создать сертификат в ACM
aws acm request-certificate \
  --domain-name fintech.example.com \
  --validation-method DNS \
  --region eu-central-1

# 2. Подтвердить владение доменом через DNS

# 3. Использовать ARN сертификата в аннотациях Service
```

### 3. Создание Ingress с HTTPS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: fintech-ingress
  namespace: fintech
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - fintech.local
    secretName: fintech-tls
  rules:
  - host: fintech.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
```

Применить конфигурацию:
```bash
kubectl apply -f kubernetes-manifests/ingress-https.yaml
```

---

## Доступ к приложению

### Получить URL Ingress

```bash
kubectl get ingress -n fintech
```

Вывод:
```
NAME              CLASS   HOSTS           ADDRESS                                                   PORTS     AGE
fintech-ingress   nginx   fintech.local   ad234dbf4c7b04e26b0990131339d559-...amazonaws.com        80, 443   1m
```

### Доступ через HTTPS

#### Для демонстрации (self-signed certificate):

```bash
# Получить hostname LoadBalancer
LB_HOSTNAME=$(kubectl get svc -n ingress-nginx ingress-nginx-controller \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

# Добавить в /etc/hosts (для локального тестирования)
echo "$LB_HOSTNAME fintech.local" | sudo tee -a /etc/hosts

# Открыть в браузере (игнорировать предупреждение о самоподписанном сертификате)
https://fintech.local
```

Или без настройки hosts:
```bash
# Прямой доступ через curl
curl -k https://$LB_HOSTNAME -H "Host: fintech.local"

# Или в браузере (игнорируя SSL warning)
https://ad234dbf4c7b04e26b0990131339d559-ed236ef572ca4bfe.elb.eu-central-1.amazonaws.com
```

---

## Production конфигурация

### С настоящим доменом и AWS ACM

1. **Создать сертификат в ACM:**

```bash
aws acm request-certificate \
  --domain-name app.fintech.com \
  --validation-method DNS \
  --region eu-central-1
```

2. **Настроить DNS:**

```bash
# Создать Route53 hosted zone (если нужно)
aws route53 create-hosted-zone --name fintech.com --caller-reference $(date +%s)

# Добавить A-record для домена
# DNS Name: app.fintech.com
# Alias Target: LoadBalancer hostname
```

3. **Обновить Ingress:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: fintech-ingress
  namespace: fintech
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"  # Если используется cert-manager
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.fintech.com
    secretName: fintech-tls-prod
  rules:
  - host: app.fintech.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
```

### С Let's Encrypt (cert-manager)

```bash
# Установить cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Создать ClusterIssuer
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```

---

## Проверка HTTPS

### 1. Проверить Ingress

```bash
kubectl get ingress -n fintech
kubectl describe ingress fintech-ingress -n fintech
```

### 2. Проверить сертификат

```bash
# Проверить secret
kubectl get secret fintech-tls -n fintech

# Просмотреть сертификат
kubectl get secret fintech-tls -n fintech -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -text -noout
```

### 3. Тестирование HTTPS

```bash
# Тест через curl
curl -k -v https://fintech.local

# Тест перенаправления HTTP -> HTTPS
curl -I http://fintech.local
# Должен вернуть 301 или 308 redirect на HTTPS
```

---

## Текущая конфигурация проекта

### Установлено:
- ✅ NGINX Ingress Controller
- ✅ Self-signed SSL Certificate
- ✅ Kubernetes Ingress с TLS
- ✅ SSL Redirect (HTTP -> HTTPS)

### URL доступа:
- **HTTP:** http://ad234dbf4c7b04e26b0990131339d559-ed236ef572ca4bfe.elb.eu-central-1.amazonaws.com
- **HTTPS:** https://ad234dbf4c7b04e26b0990131339d559-ed236ef572ca4bfe.elb.eu-central-1.amazonaws.com

**Порты:**
- 80 (HTTP) - автоматически перенаправляется на 443
- 443 (HTTPS) - основной вход

---

## Troubleshooting

### Ingress не получает ADDRESS

```bash
# Проверить статус ingress controller
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller

# Проверить Service
kubectl get svc -n ingress-nginx
```

### SSL Certificate ошибки

```bash
# Проверить secret
kubectl describe secret fintech-tls -n fintech

# Пересоздать сертификат
kubectl delete secret fintech-tls -n fintech
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=fintech.local/O=fintech"
kubectl create secret tls fintech-tls --cert=tls.crt --key=tls.key -n fintech
```

### 502 Bad Gateway

```bash
# Проверить backend pods
kubectl get pods -n fintech
kubectl logs -n fintech -l app=frontend

# Проверить Service
kubectl get svc frontend -n fintech
```

---

## Безопасность

### Рекомендации для production:

1. **Использовать валидные SSL сертификаты:**
   - AWS Certificate Manager
   - Let's Encrypt через cert-manager
   - Коммерческий CA

2. **Настроить Security Headers:**
```yaml
annotations:
  nginx.ingress.kubernetes.io/configuration-snippet: |
    more_set_headers "X-Frame-Options: DENY";
    more_set_headers "X-Content-Type-Options: nosniff";
    more_set_headers "X-XSS-Protection: 1; mode=block";
    more_set_headers "Strict-Transport-Security: max-age=31536000";
```

3. **Ограничить доступ:**
```yaml
annotations:
  nginx.ingress.kubernetes.io/whitelist-source-range: "10.0.0.0/8,172.16.0.0/12"
```

4. **Rate Limiting:**
```yaml
annotations:
  nginx.ingress.kubernetes.io/limit-rps: "10"
  nginx.ingress.kubernetes.io/limit-connections: "5"
```

---

## Удаление HTTPS

```bash
# Удалить Ingress
kubectl delete ingress fintech-ingress -n fintech

# Удалить Secret
kubectl delete secret fintech-tls -n fintech

# Удалить Ingress Controller (опционально)
helm uninstall ingress-nginx -n ingress-nginx
kubectl delete namespace ingress-nginx

# Удалить сертификаты
rm tls.crt tls.key
```

---

## Стоимость

**AWS Network Load Balancer (для Ingress):**
- ~$0.0225/час (~$16/месяц)
- ~$0.006/GB обработанных данных

**Итого:** ~$20-30/месяц дополнительно к стоимости EKS

---

## Заключение

HTTPS полностью настроен и работает! Для демонстрации проекта используется самоподписанный сертификат. Для production развертывания рекомендуется использовать AWS Certificate Manager или Let's Encrypt.
