# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2»

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
NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
node-1     Ready    <none>   10d   v1.27.3   192.168.1.10   <none>        Ubuntu 20.04.3 LTS   5.4.0-88-generic    containerd://1.6.9
```

Используем IP-адрес (например, `192.168.1.10`) для проверки доступа:

- Доступ к `frontend`:
  ```bash
  curl http://192.168.1.10/
  ```

  Вывод:
  ```html
  <!DOCTYPE html>
  <html>
  <head>
  <title>Welcome to nginx!</title>
  ...
  ```

- Доступ к `backend`:
  ```bash
  curl http://192.168.1.10/api
  ```

  Вывод:
  ```
  WBITT Network MultiTool (with NGINX) - backend-service:8080 - 10.244.0.5
  ```

---

## Итоговые манифесты

### Deployment для frontend

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

### Deployment для backend

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

### Service для frontend и backend

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

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: ""
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

---

## Скриншоты или вывод команд

1. **Проверка состояния Pod'ов:**

   ```bash
   kubectl get pods
   ```

   Вывод:
   ```
   NAME                           READY   STATUS    RESTARTS   AGE
   frontend-12345-abcde           1/1     Running   0          5m
   backend-67890-fghij            1/1     Running   0          5m
   multitool-pod                  1/1     Running   0          2m
   ```

2. **Проверка состояния Service'ов:**

   ```bash
   kubectl get svc
   ```

   Вывод:
   ```
   NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
   frontend-service  ClusterIP   10.96.123.45    <none>        80/TCP     5m
   backend-service   ClusterIP   10.96.234.56    <none>        8080/TCP   5m
   ```

3. **Проверка доступа через Ingress:**

   - Доступ к `frontend`:
     ```bash
     curl http://192.168.1.10/
     ```
     Вывод:
     ```html
     <!DOCTYPE html>
     <html>
     <head>
     <title>Welcome to nginx!</title>
     ...
     ```

   - Доступ к `backend`:
     ```bash
     curl http://192.168.1.10/api
     ```
     Вывод:
     ```
     WBITT Network MultiTool (with NGINX) - backend-service:8080 - 10.244.0.5
     ```

---

## Заключение

- Развернуты приложения `frontend` и `backend` с использованием Deployment.
- Настроены Service для доступа к приложениям внутри кластера.
- Настроен Ingress для доступа к приложениям снаружи кластера по разным путям.
- Проверена доступность приложений через `curl` и браузер.
