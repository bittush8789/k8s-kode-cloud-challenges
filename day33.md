# Kubernetes LEMP Setup

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy a complete LEMP Stack environment on Kubernetes for hosting enterprise-grade PHP applications.

Your task is to create a Kubernetes cluster using Kind, deploy Nginx, PHP-FPM, and MySQL components using Kubernetes manifests, expose the application, and verify deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise applications including:

- PHP web applications
- CMS platforms
- Internal portals
- Banking systems
- Enterprise dashboards

The organization currently faces several infrastructure challenges:

- Manual server management
- Downtime during deployments
- Poor scalability
- No centralized orchestration
- Difficult infrastructure maintenance

To modernize infrastructure, the organization wants to deploy a complete LEMP Stack on Kubernetes.

As a DevOps Engineer, your responsibilities are to:

1. Create a Kubernetes cluster using Kind.
2. Deploy MySQL database.
3. Deploy Nginx web server.
4. Deploy PHP-FPM application layer.
5. Configure Kubernetes Services.
6. Verify communication between components.
7. Build production-style Kubernetes architecture.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy MySQL database.
4. Deploy Nginx + PHP-FPM.
5. Configure Kubernetes Services.
6. Verify LEMP stack connectivity.
7. Scale applications on Kubernetes.

---

# What is LEMP Stack?

| Component | Description |
|---|---|
| Linux | Operating System |
| Nginx | Web Server |
| MySQL | Database |
| PHP-FPM | PHP Processing Engine |

---

# Architecture Overview

```text
                 +------------------+
                 |      Users       |
                 +--------+---------+
                          |
                          v
                 +------------------+
                 | Nginx Service    |
                 +--------+---------+
                          |
                          v
                 +------------------+
                 | Nginx Pods       |
                 +--------+---------+
                          |
                          v
                 +------------------+
                 | PHP-FPM Pods     |
                 +--------+---------+
                          |
                          v
                 +------------------+
                 | MySQL Service    |
                 +--------+---------+
                          |
                          v
                 +------------------+
                 | MySQL Pod        |
                 +------------------+
```

---

# Solution

# Step 1: Update Ubuntu Packages

```bash
sudo apt update && sudo apt upgrade -y
```

---

# Step 2: Install Docker

```bash
sudo apt install docker.io -y
```

Enable Docker:

```bash
sudo systemctl enable docker

sudo systemctl start docker
```

Verify:

```bash
docker --version
```

---

# Step 3: Install Kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
```

Make executable:

```bash
chmod +x ./kind
```

Move binary:

```bash
sudo mv ./kind /usr/local/bin/kind
```

Verify:

```bash
kind --version
```

---

# Step 4: Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s \
https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Make executable:

```bash
chmod +x kubectl
```

Move binary:

```bash
sudo mv kubectl /usr/local/bin/
```

Verify:

```bash
kubectl version --client
```

---

# Step 5: Create Kind Kubernetes Cluster

```bash
kind create cluster --name lemp-cluster
```

Verify:

```bash
kubectl cluster-info
```

Expected:

```text
Kubernetes control plane is running
```

---

# Step 6: Verify Nodes

```bash
kubectl get nodes
```

Expected:

```text
STATUS: Ready
```

---

# Step 7: Create Namespace

```bash
kubectl create namespace lemp-stack
```

---

# Step 8: Create MySQL Deployment Manifest

```bash
nano mysql-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: mysql
  namespace: lemp-stack

spec:
  replicas: 1

  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql

    spec:
      containers:

        - name: mysql

          image: mysql:5.7

          env:
            - name: MYSQL_ROOT_PASSWORD
              value: root123

            - name: MYSQL_DATABASE
              value: lempdb

          ports:
            - containerPort: 3306
```

---

# Step 9: Apply MySQL Deployment

```bash
kubectl apply -f mysql-deployment.yaml
```

Expected:

```text
deployment.apps/mysql created
```

---

# Step 10: Create MySQL Service Manifest

```bash
nano mysql-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mysql-service
  namespace: lemp-stack

spec:
  selector:
    app: mysql

  ports:
    - port: 3306
      targetPort: 3306
```

---

# Step 11: Apply MySQL Service

```bash
kubectl apply -f mysql-service.yaml
```

---

# Step 12: Verify MySQL Pods

```bash
kubectl get pods -n lemp-stack
```

Expected:

```text
mysql Running
```

---

# Step 13: Create PHP-FPM Deployment Manifest

```bash
nano php-fpm-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: php-fpm
  namespace: lemp-stack

spec:
  replicas: 2

  selector:
    matchLabels:
      app: php-fpm

  template:
    metadata:
      labels:
        app: php-fpm

    spec:
      containers:

        - name: php-fpm

          image: php:8.2-fpm

          env:
            - name: DB_HOST
              value: mysql-service

            - name: DB_USER
              value: root

            - name: DB_PASSWORD
              value: root123

          ports:
            - containerPort: 9000
```

---

# Step 14: Apply PHP-FPM Deployment

```bash
kubectl apply -f php-fpm-deployment.yaml
```

---

# Step 15: Create PHP-FPM Service

```bash
nano php-fpm-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: php-fpm-service
  namespace: lemp-stack

spec:
  selector:
    app: php-fpm

  ports:
    - port: 9000
      targetPort: 9000
```

---

# Step 16: Apply PHP-FPM Service

```bash
kubectl apply -f php-fpm-service.yaml
```

---

# Step 17: Create Nginx Deployment Manifest

```bash
nano nginx-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx
  namespace: lemp-stack

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:

        - name: nginx

          image: nginx

          ports:
            - containerPort: 80
```

---

# Step 18: Apply Nginx Deployment

```bash
kubectl apply -f nginx-deployment.yaml
```

---

# Step 19: Create Nginx Service Manifest

```bash
nano nginx-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service
  namespace: lemp-stack

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32007
```

---

# Step 20: Apply Nginx Service

```bash
kubectl apply -f nginx-service.yaml
```

---

# Step 21: Verify Deployments

```bash
kubectl get deployments -n lemp-stack
```

Expected:

```text
mysql
php-fpm
nginx
```

---

# Step 22: Verify Pods

```bash
kubectl get pods -n lemp-stack
```

Expected:

```text
All Pods Running
```

---

# Step 23: Verify Services

```bash
kubectl get svc -n lemp-stack
```

Expected:

```text
mysql-service
php-fpm-service
nginx-service
```

---

# Step 24: Port Forward Nginx Service

```bash
kubectl port-forward svc/nginx-service \
8080:80 -n lemp-stack
```

---

# Step 25: Access LEMP Application

Open browser:

```text
http://localhost:8080
```

Nginx welcome page should load.

---

# Step 26: Verify MySQL Connectivity

Run temporary MySQL client Pod:

```bash
kubectl run mysql-client \
-it --rm \
--image=mysql:5.7 \
--restart=Never \
-n lemp-stack -- bash
```

Inside Pod:

```bash
mysql -h mysql-service \
-u root \
-p
```

Password:

```text
root123
```

Expected:

```text
MySQL connection successful
```

---

# Step 27: Verify PHP-FPM Pods

```bash
kubectl get pods -n lemp-stack \
-l app=php-fpm
```

---

# Step 28: Scale Nginx Deployment

```bash
kubectl scale deployment nginx \
--replicas=4 -n lemp-stack
```

Verify:

```bash
kubectl get deployments -n lemp-stack
```

Expected:

```text
READY 4/4
```

---

# Step 29: Verify Scaled Pods

```bash
kubectl get pods -n lemp-stack
```

Expected:

```text
4 Nginx Pods Running
```

---

# Step 30: Inspect Deployments

```bash
kubectl describe deployment nginx \
-n lemp-stack
```

---

# Step 31: Inspect Services

```bash
kubectl describe svc nginx-service \
-n lemp-stack
```

---

# Step 32: Check Application Logs

Get Pod name:

```bash
kubectl get pods -n lemp-stack
```

Check logs:

```bash
kubectl logs <pod-name> -n lemp-stack
```

---

# Step 33: Verify Cluster Resources

```bash
kubectl get all -n lemp-stack
```

Expected:

```text
Pods
Services
Deployments
ReplicaSets
```

---

# Step 34: Perform Rolling Update

Update Nginx image:

```bash
kubectl set image deployment/nginx \
nginx=nginx:latest \
-n lemp-stack
```

Verify rollout:

```bash
kubectl rollout status deployment/nginx \
-n lemp-stack
```

---

# Step 35: Verify Rollout History

```bash
kubectl rollout history deployment/nginx \
-n lemp-stack
```

---

# Step 36: Rollback Deployment

```bash
kubectl rollout undo deployment/nginx \
-n lemp-stack
```

---

# Step 37: Troubleshooting LEMP Issues

# Check Pods

```bash
kubectl get pods -n lemp-stack
```

---

# Check Logs

```bash
kubectl logs <pod-name> -n lemp-stack
```

---

# Check Services

```bash
kubectl get svc -n lemp-stack
```

---

# Common Issues

| Issue | Description |
|---|---|
| MySQL Crash | Wrong password |
| PHP-FPM Failure | DB connectivity issue |
| Nginx Unreachable | Service issue |
| Pod CrashLoop | Startup failure |

---

# Step 38: Delete Resources

```bash
kubectl delete deployment nginx -n lemp-stack

kubectl delete deployment php-fpm -n lemp-stack

kubectl delete deployment mysql -n lemp-stack

kubectl delete svc nginx-service -n lemp-stack

kubectl delete svc php-fpm-service -n lemp-stack

kubectl delete svc mysql-service -n lemp-stack
```

---

# Step 39: Delete Namespace

```bash
kubectl delete namespace lemp-stack
```

---

# Step 40: Delete Kind Cluster

```bash
kind delete cluster --name lemp-cluster
```

---

# Manifest Files

# mysql-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: mysql
  namespace: lemp-stack

spec:
  replicas: 1

  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql

    spec:
      containers:

        - name: mysql

          image: mysql:5.7

          env:
            - name: MYSQL_ROOT_PASSWORD
              value: root123

            - name: MYSQL_DATABASE
              value: lempdb

          ports:
            - containerPort: 3306
```

---

# php-fpm-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: php-fpm
  namespace: lemp-stack

spec:
  replicas: 2

  selector:
    matchLabels:
      app: php-fpm

  template:
    metadata:
      labels:
        app: php-fpm

    spec:
      containers:

        - name: php-fpm

          image: php:8.2-fpm

          env:
            - name: DB_HOST
              value: mysql-service

            - name: DB_USER
              value: root

            - name: DB_PASSWORD
              value: root123

          ports:
            - containerPort: 9000
```

---

# nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx
  namespace: lemp-stack

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:

        - name: nginx

          image: nginx

          ports:
            - containerPort: 80
```

---

# nginx-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service
  namespace: lemp-stack

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32007
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- MySQL deployed successfully
- PHP-FPM deployed successfully
- Nginx deployed successfully
- LEMP stack communication operational
- Kubernetes Services configured
- Scaling operational
- Enterprise-grade LEMP deployment workflow implemented

---

# Skills Covered

- Kubernetes Deployments
- Kubernetes Services
- MySQL on Kubernetes
- Nginx on Kubernetes
- PHP-FPM on Kubernetes
- LEMP Stack
- Scaling Applications
- Rolling Updates
- DevOps
- Cloud-Native Deployment

---

# Real Enterprise Workflow

```text
Developer Push
       |
       v
Docker Images
       |
       v
Kubernetes Deployments
       |
       +----------------------+
       |          |           |
       v          v           v
Nginx Pods   PHP-FPM Pods   MySQL Pods
       |
       v
Kubernetes Services
       |
       v
User Access
```

---

# Complete Command Sequence

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install docker.io -y

sudo systemctl enable docker

sudo systemctl start docker

docker --version

curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind

kind --version

curl -LO "https://dl.k8s.io/release/$(curl -L -s \
https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/

kubectl version --client

kind create cluster --name lemp-cluster

kubectl create namespace lemp-stack

kubectl apply -f mysql-deployment.yaml

kubectl apply -f mysql-service.yaml

kubectl apply -f php-fpm-deployment.yaml

kubectl apply -f php-fpm-service.yaml

kubectl apply -f nginx-deployment.yaml

kubectl apply -f nginx-service.yaml

kubectl get pods -n lemp-stack

kubectl get svc -n lemp-stack

kubectl port-forward svc/nginx-service \
8080:80 -n lemp-stack

kubectl run mysql-client \
-it --rm \
--image=mysql:5.7 \
--restart=Never \
-n lemp-stack -- bash

mysql -h mysql-service \
-u root \
-p

kubectl scale deployment nginx \
--replicas=4 -n lemp-stack

kubectl describe deployment nginx \
-n lemp-stack

kubectl describe svc nginx-service \
-n lemp-stack

kubectl logs <pod-name> -n lemp-stack

kubectl get all -n lemp-stack

kubectl rollout history deployment/nginx \
-n lemp-stack

kubectl rollout undo deployment/nginx \
-n lemp-stack

kubectl delete namespace lemp-stack

kind delete cluster --name lemp-cluster
```