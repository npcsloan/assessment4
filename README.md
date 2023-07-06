# Assessment 4 - Twoge
## Presentation Goal
1. Showcase deployment of a Python Flask-based Twitter alternative called 'Twoge' using first EKS and then Minikube
2. Present diagrams, describe relationship between different resources
3. Go over files and commands used; ask for questions; focus only on areas of particular interest
## Project Diagrams
### Configuring from Local Machine
<img src="https://github.com/npcsloan/assessment4/blob/main/Local-to-EKS.png">

### Architecture of EKS Cluster
<img src="https://github.com/npcsloan/assessment4/blob/main/EKS-cluster-diagram.png">

## Deploy using EKS
1. Fork [repo](https://github.com/chandradeoarya/twoge) and git clone k8s branch
### Dockerfile
2. Create Dockerfile and run ```docker build -t npcsloan/assessment4 .```
3. Push to dockerhub ```docker push npcsloan/assessment4```
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
4. Create namespace file
```
apiVersion: v1
kind: Namespace
metadata:
  name: twogespace
  labels:
    name: twogespace
```
### Configmap
5. Create Configmap file
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
6. Create Secrets file
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
### Application Deployment
7. Create application deployment file using docker image created in step 1
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
          readinessProbe: # checks to see if pod is ready for traffic
            tcpSocket:
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe: # checks if pod is running
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
8. Configure connectivity of application deployment via Service file
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
9. Create database deployment file using postgres image
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
10. Configure connectivity of database deployment via Service file
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
### Resource Quota
11. Create resource quota file with desired specs
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
12. Create EKS Cluster
```
eksctl create cluster --region us-west-1 --node-type t2.small --nodes 1 --nodes-min 1 --nodes-max 1 --name austin-assessment4
```
13. Configure EKS Volume (still need to create PV and PVC)
```
eksctl utils associate-iam-oidc-provider --region=eu-central-1 --cluster=YourClusterNameHere --approve
```
```
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster austin-assessment4 \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-only \
  --role-name AmazonEKS_EBS_CSI_DriverRole
```
```
eksctl create addon --name aws-ebs-csi-driver --cluster austin-assessment4 \
--service-account-role-arn arn:aws:iam::$(aws sts get-caller-identity --query Account \
--output text):role/AmazonEKS_EBS_CSI_DriverRole --force
```
14. Apply namespace ```kubectl apply -f namespace```
15. Switch namespaces ```kubectl config set-context --current --namespace=twogespace```
16. Apply Configmap and Secrets file ```kubectl apply -f variables```
17. Apply app and database deployments and services ```kubectl apply -f twoge-kube```
18. Use ```kubectl get all``` to check deployment status; once all is stable apply resource quota to namespace ```kubectl apply -f resourcequota```
19. Run ```kubectl get services``` to get address of app service. Go to address in browser to see running application:
<img src="https://github.com/npcsloan/assessment4/blob/main/eks-twoge.png">

## Using Minikube
1. Change context to minikube ```kubectl config use-context minikube```
2. Redo above steps 14-18 to apply files
3. Run ```minikube service twoge-service --namespace twogespace --url```; paste url in browser:
<img src="https://github.com/npcsloan/assessment4/blob/main/localhost-twoge.png">
