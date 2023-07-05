# Assessment 4 - Twoge

### Dockerfile
```
FROM python:alpine

RUN apk update && \
    apk add --no-cache build-base libffi-dev openssl-dev

COPY . /app
WORKDIR /app

RUN pip install -r requirements.txt

EXPOSE 8080
CMD python app.py
```
### Namespace
```
apiVersion: v1
kind: Namespace
metadata:
  name: twogespace
  labels:
    name: twogespace
```
### Application Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: twoge-dep
  labels:
    app: twoge-k8s
spec:
  selector:
    matchLabels:
      app: twoge-k8s
  replicas: 1
  template:
    metadata:
      labels:
        app: twoge-k8s
    spec:
      containers:
        - name: twoge-container
          image: npcsloan/assessment4
          ports:
            - containerPort: 8080
          readinessProbe:
            tcpSocket:
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            tcpSocket:
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
          env:
            - name: DB_DATABASE
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: db_name
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: username
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: postgres-configmap
                  key: database_url
            - name: DB_PORT
              valueFrom:
                configMapKeyRef:
                  name: postgres-configmap
                  key: database_port
```
### Application Service
```
apiVersion: v1
kind: Service
metadata:
  name: twoge-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: twoge-k8s
```
### Database Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres
        env:
          - name: POSTGRES_DB
            valueFrom:
              secretKeyRef:
                name: postgres-secret
                key: db_name
          - name: POSTGRES_USER
            valueFrom:
              secretKeyRef:
                name: postgres-secret
                key: username
          - name: POSTGRES_PASSWORD
            valueFrom:
              secretKeyRef:
                name: postgres-secret
                key: password
```
### Database Service
```
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  selector:
    app: postgres
  type: ClusterIP
  ports:
  - port: 5432
```
### Configmap
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-configmap
data:
  database_url: "postgres-service"
  database_port: "5432"
```
### Secrets
```
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
type: Opaque
data:
  db_name: <your database name>
  username: <your username>
  password: <your password>
```
### Resource Quota
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-quota
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```
