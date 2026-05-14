# Deploy LAMP Stack on Kubernetes Cluster

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy a complete LAMP Stack environment on Kubernetes for hosting enterprise PHP applications.

Your task is to create a Kubernetes cluster using Kind, deploy Apache, PHP, and MySQL components using Kubernetes manifests, expose the application, and verify deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise-grade web applications including:

- PHP applications
- CMS platforms
- Internal portals
- Banking systems
- Enterprise dashboards

The organization currently faces multiple infrastructure challenges:

- Manual server management
- Downtime during deployments
- Poor scalability
- No centralized orchestration
- Difficult application management

To modernize infrastructure, the organization wants to deploy a complete LAMP Stack on Kubernetes.

As a DevOps Engineer, your responsibilities are to:

1. Create a Kubernetes cluster using Kind.
2. Deploy MySQL database.
3. Deploy Apache + PHP application.
4. Configure Kubernetes Services.
5. Verify connectivity between components.
6. Expose application externally.
7. Build production-style Kubernetes architecture.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy MySQL database.
4. Deploy Apache + PHP application.
5. Configure Kubernetes Services.
6. Verify LAMP stack connectivity.
7. Scale applications on Kubernetes.

---

# What is LAMP Stack?

| Component | Description |
|---|---|
| Linux | Operating System |
| Apache | Web Server |
| MySQL | Database |
| PHP | Backend Programming Language |

---

# Architecture Overview

```text
                 +------------------+
                 |      Users       |
                 +--------+---------+
                          |
                          v
                 +------------------+
                 | Apache + PHP App |
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
kind create cluster --name lamp-cluster
```

Verify:

```bash
kubectl cluster-info
```

Expected output:

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
kubectl create namespace lamp-stack
```

Verify:

```bash
kubectl get namespaces
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
  namespace: lamp-stack

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
              value: lampdb

          ports:
            - containerPort: 3306
```

---

# Step 9: Apply MySQL Deployment

```bash
kubectl apply -f mysql-deployment.yaml
```

Expected output:

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
  namespace: lamp-stack

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
kubectl get pods -n lamp-stack
```

Expected:

```text
mysql Running
```

---

# Step 13: Create Apache + PHP Deployment Manifest

```bash
nano php-apache-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: php-apache
  namespace: lamp-stack

spec:
  replicas: 2

  selector:
    matchLabels:
      app: php-apache

  template:
    metadata:
      labels:
        app: php-apache

    spec:
      containers:

        - name: php-apache

          image: php:8.2-apache

          env:
            - name: DB_HOST
              value: mysql-service

            - name: DB_USER
              value: root

            - name: DB_PASSWORD
              value: root123

          ports:
            - containerPort: 80
```

---

# Step 14: Apply Apache + PHP Deployment

```bash
kubectl apply -f php-apache-deployment.yaml
```

Expected output:

```text
deployment.apps/php-apache created
```

---

# Step 15: Verify Deployments

```bash
kubectl get deployments -n lamp-stack
```

Expected:

```text
mysql
php-apache
```

---

# Step 16: Verify Pods

```bash
kubectl get pods -n lamp-stack
```

Expected:

```text
All Pods Running
```

---

# Step 17: Create Apache Service Manifest

```bash
nano apache-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: apache-service
  namespace: lamp-stack

spec:
  type: NodePort

  selector:
    app: php-apache

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32006
```

---

# Step 18: Apply Apache Service

```bash
kubectl apply -f apache-service.yaml
```

Expected output:

```text
service/apache-service created
```

---

# Step 19: Verify Services

```bash
kubectl get svc -n lamp-stack
```

Expected:

```text
mysql-service
apache-service
```

---

# Step 20: Port Forward Apache Service

```bash
kubectl port-forward svc/apache-service \
8080:80 -n lamp-stack
```

---

# Step 21: Access LAMP Application

Open browser:

```text
http://localhost:8080
```

Apache default page should load.

---

# Step 22: Verify MySQL Connectivity

Run temporary MySQL client Pod:

```bash
kubectl run mysql-client \
-it --rm \
--image=mysql:5.7 \
--restart=Never \
-n lamp-stack -- bash
```

Inside Pod:

```bash
mysql -h mysql-service \
-u root \
-p
```

Enter password:

```text
root123
```

Expected:

```text
MySQL connection successful
```

---

# Step 23: Scale Apache Deployment

```bash
kubectl scale deployment php-apache \
--replicas=4 -n lamp-stack
```

Verify:

```bash
kubectl get deployments -n lamp-stack
```

Expected:

```text
READY 4/4
```

---

# Step 24: Verify Scaled Pods

```bash
kubectl get pods -n lamp-stack
```

Expected:

```text
4 Running Pods
```

---

# Step 25: Inspect Deployments

```bash
kubectl describe deployment php-apache \
-n lamp-stack
```

---

# Step 26: Inspect Services

```bash
kubectl describe svc apache-service \
-n lamp-stack
```

---

# Step 27: Check Application Logs

Get Pod name:

```bash
kubectl get pods -n lamp-stack
```

Check logs:

```bash
kubectl logs <pod-name> -n lamp-stack
```

---

# Step 28: Verify Cluster Resources

```bash
kubectl get all -n lamp-stack
```

Expected:

```text
Pods
Services
Deployments
ReplicaSets
```

---

# Step 29: Perform Rolling Update

Update Apache image:

```bash
kubectl set image deployment/php-apache \
php-apache=php:8.3-apache \
-n lamp-stack
```

Verify rollout:

```bash
kubectl rollout status deployment/php-apache \
-n lamp-stack
```

---

# Step 30: Verify Rollout History

```bash
kubectl rollout history deployment/php-apache \
-n lamp-stack
```

---

# Step 31: Rollback Deployment

```bash
kubectl rollout undo deployment/php-apache \
-n lamp-stack
```

---

# Step 32: Delete Resources

```bash
kubectl delete deployment php-apache -n lamp-stack

kubectl delete deployment mysql -n lamp-stack

kubectl delete svc apache-service -n lamp-stack

kubectl delete svc mysql-service -n lamp-stack
```

---

# Step 33: Delete Namespace

```bash
kubectl delete namespace lamp-stack
```

---

# Step 34: Delete Kind Cluster

```bash
kind delete cluster --name lamp-cluster
```

Expected output:

```text
Deleting cluster "lamp-cluster"
```

---

# Manifest Files

# mysql-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: mysql
  namespace: lamp-stack

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
              value: lampdb

          ports:
            - containerPort: 3306
```

---

# mysql-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mysql-service
  namespace: lamp-stack

spec:
  selector:
    app: mysql

  ports:
    - port: 3306
      targetPort: 3306
```

---

# php-apache-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: php-apache
  namespace: lamp-stack

spec:
  replicas: 2

  selector:
    matchLabels:
      app: php-apache

  template:
    metadata:
      labels:
        app: php-apache

    spec:
      containers:

        - name: php-apache

          image: php:8.2-apache

          env:
            - name: DB_HOST
              value: mysql-service

            - name: DB_USER
              value: root

            - name: DB_PASSWORD
              value: root123

          ports:
            - containerPort: 80
```

---

# apache-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: apache-service
  namespace: lamp-stack

spec:
  type: NodePort

  selector:
    app: php-apache

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32006
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- MySQL deployed successfully
- Apache + PHP deployed successfully
- LAMP stack communication operational
- Kubernetes Services configured
- Scaling operational
- Rolling updates configured
- Enterprise-grade LAMP deployment workflow implemented

---

# Real Industry Use Cases

| Use Case | Description |
|---|---|
| CMS Platforms | WordPress hosting |
| Enterprise Portals | Internal dashboards |
| Banking Apps | PHP applications |
| DevOps | Kubernetes deployments |
| Cloud-Native Apps | Containerized LAMP stack |

---

# Skills Covered

- Kubernetes Deployments
- Kubernetes Services
- MySQL on Kubernetes
- Apache + PHP on Kubernetes
- LAMP Stack
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
       +------------------+
       |                  |
       v                  v
Apache + PHP Pods    MySQL Pods
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

kind create cluster --name lamp-cluster

kubectl create namespace lamp-stack

kubectl apply -f mysql-deployment.yaml

kubectl apply -f mysql-service.yaml

kubectl apply -f php-apache-deployment.yaml

kubectl apply -f apache-service.yaml

kubectl get pods -n lamp-stack

kubectl get svc -n lamp-stack

kubectl port-forward svc/apache-service \
8080:80 -n lamp-stack

kubectl run mysql-client \
-it --rm \
--image=mysql:5.7 \
--restart=Never \
-n lamp-stack -- bash

mysql -h mysql-service \
-u root \
-p

kubectl scale deployment php-apache \
--replicas=4 -n lamp-stack

kubectl describe deployment php-apache \
-n lamp-stack

kubectl describe svc apache-service \
-n lamp-stack

kubectl logs <pod-name> -n lamp-stack

kubectl get all -n lamp-stack

kubectl set image deployment/php-apache \
php-apache=php:8.3-apache \
-n lamp-stack

kubectl rollout status deployment/php-apache \
-n lamp-stack

kubectl rollout history deployment/php-apache \
-n lamp-stack

kubectl rollout undo deployment/php-apache \
-n lamp-stack

kubectl delete namespace lamp-stack

kind delete cluster --name lamp-cluster
```