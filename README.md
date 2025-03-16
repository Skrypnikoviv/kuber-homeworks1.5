# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2»
Не успел приложить скрины только оформил если вернете на дороботку устраню )
---

## Задание 1: Создание Deployment и Service для приложений backend и frontend

### 1. Создание Deployment для frontend

Создадим Deployment для приложения `frontend` на основе образа `nginx` с тремя репликами.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

### 2. Создание Deployment для backend

Создадим Deployment для приложения `backend` на основе образа `multitool`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
```

### 3. Создание Service для frontend и backend

Создадим Service для обеспечения доступа к приложениям внутри кластера.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
```

### 4. Проверка доступности приложений внутри кластера

Создадим отдельный Pod с `multitool` для проверки доступности приложений.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multitool-pod
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool
    command: ["sleep", "infinity"]
```

Подключимся к Pod и проверим доступность `frontend` и `backend`:

```bash
kubectl exec -it multitool-pod -- curl http://frontend-service
kubectl exec -it multitool-pod -- curl http://backend-service:8080
```

#### Пример вывода:

- Для `frontend`:
  ```bash
  <!DOCTYPE html>
  <html>
  <head>
  <title>Welcome to nginx!</title>
  ...
  ```

- Для `backend`:
  ```bash
  WBITT Network MultiTool (with NGINX) - backend-service:8080 - 10.244.0.5
  ```

---

## Задание 2: Создание Ingress и обеспечение доступа снаружи кластера

### 1. Включение Ingress-controller в MicroK8S

Включим Ingress-controller в MicroK8S:

```bash
microk8s enable ingress
```

### 2. Создание Ingress

Создадим Ingress для обеспечения доступа к `frontend` и `backend` по разным путям.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: "" # Оставляем пустым для доступа по IP
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
```

### 3. Проверка доступа снаружи кластера

Узнаем IP-адрес кластера MicroK8S:

```bash
microk8s kubectl get nodes -o wide
```

Пример вывода:
```
скрин
```

Используем IP-адрес (например, `127.0.0.1`) для проверки доступа:

- Доступ к `frontend`:
  ```bash
 скрин
  ```

  Вывод:
  ```html
скрин
  ...
  ```

- Доступ к `backend`:
  ```bash
  Скрин
  ```

  Вывод:
  ```
Скрин
  ```

---

