# Kubernetes Deployment для Hello-DevOps приложения
### 🎯Назначение приложения
# Hello-DevOps - это демонстрационное веб-приложение, созданное для изучения DevOps практик. Оно предоставляет простой REST API и веб-интерфейс для управления задачами.

Основные функции:

1. Отображение приветственного сообщения
2. REST API эндпоинты
3. Статус здоровья приложения
4. Метрики для мониторинга
   
### 📊 Параметры Deployment
|Параметр|Значение|	Описание|
|---|---|---|
|Имя приложения|	hello-devops-app|	Уникальный идентификатор|
|Количество реплик|	3|	Число запущенных копий|
|Порт контейнера|	3000|	Внутренний порт приложения|
|Версия образа|	v1.0.0|	Тег Docker образа|
|Лимиты| CPU	500m|	Максимальное использование CPU|
|Лимиты памяти|	512Mi|	Максимальное использование памяти|
|Стратегия обновления|	RollingUpdate|	Бесшовное обновление|
|Проверка готовности|	/health	Эндпоинт| готовности|
|Проверка живучести|	/live|	Эндпоинт живучести|

### 🚀 YAML манифест Deployment
``` apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-devops-app
  namespace: default
  labels:
    app: hello-devops
    version: v1.0.0
    environment: production
spec:
  replicas: 3
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: hello-devops
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: hello-devops
        version: v1.0.0
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: hello-devops-container
        image: xdesh4ka/hello-devops:v1.0.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3000
          protocol: TCP
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "3000"
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /live
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 1
        securityContext:
          runAsNonRoot: true
          runAsUser: 1000
          allowPrivilegeEscalation: false
```


### 🛠 Команды kubectl
#### Применение манифестов
# Применить все манифесты из директории
kubectl apply -f k8s/

# Применить конкретный манифест
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Применить с проверкой синтаксиса
kubectl apply --validate=true --dry-run=client -f deployment.yaml
