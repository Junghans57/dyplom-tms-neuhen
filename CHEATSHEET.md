# Шпаргалка команд для проекта

## Проверка статуса
kubectl get pods -n fintech
kubectl get nodes
kubectl top pods -n fintech

## Доступ к Grafana
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80

## Логи
kubectl logs -n fintech -l app=frontend --tail=50

## Git
git status
git add .
git commit -m "message"
git push origin main
