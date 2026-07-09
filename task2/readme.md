# Задание 2

## Динамическое масштабирование по памяти

### Подготовка кластера
```sh
minikube start --cpus=4 --memory=4096
minikube addons enable metrics-server
```

### Сборка и загрузка образа
```sh
cd scaletestapp
minikube docker-env --shell powershell | Invoke-Expression
minikube image build -t scaletestapp:latest .
```

### Настройка кластера
```sh
cd .. #(работа из /task2)
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa-memory.yaml
```

### Запуск теста:
```sh
#получить url, по которому доступен сервис для тестирования:
minikube service test-app-svc --url
#запустить тест 
locust -f locustfile.py --host=http://127.0.0.1:62528
```
На 100 пользователях с нагрузкой 10/s изменений кол-ва подов не было. При увеличении нагрузки до 1000 пользователей с 100/s, число подов было увеличено до 2-х.


## Динамическое масштабирование по RPS через Prometheus

### Подключение Prometheus
```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace -f prometheus-values.yaml

kubectl apply -f servicemonitor.yaml
```

### Проверка метрик
```sh
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090
```

### Установка адаптера, для подсчета RPS
```sh
helm install prometheus-adapter prometheus-community/prometheus-adapter -f prometheus-adapter.yaml
```

### Подключение кастомного HPA по RPS
```sh
kubectl apply -f hpa-custom.yaml
```