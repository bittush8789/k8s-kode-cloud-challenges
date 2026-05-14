# Fix Issue with LAMP Environment in Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries is facing issues while deploying a LAMP stack application on Kubernetes. The Apache web server, MySQL database, and PHP application are not communicating correctly.

Your task is to troubleshoot and fix issues in a Kubernetes-based LAMP environment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries hosts enterprise PHP applications using a LAMP stack architecture.

The production environment contains:

- Linux
- Apache
- MySQL
- PHP
- Kubernetes

The engineering team currently faces several critical issues:

- PHP application unable to connect to MySQL
- Apache container crashing
- MySQL authentication errors
- Kubernetes Service communication failures
- Environment variable misconfigurations
- Incorrect database credentials

Applications affected:

- Internal dashboards
- Banking portals
- CMS applications
- Enterprise web systems

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Deploy MySQL database.
3. Deploy PHP Apache application.
4. Configure Services and environment variables.
5. Troubleshoot connectivity issues.
6. Fix broken LAMP environment.
7. Restore healthy application communication.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy MySQL.
4. Deploy Apache + PHP application.
5. Configure Kubernetes Services.
6. Fix LAMP stack communication issues.
7. Verify healthy deployment.

---

# What is LAMP Stack?

LAMP stands for:

| Component | Description |
|---|---|
| Linux | Operating System |
| Apache | Web Server |
| MySQL | Database |
| PHP | Backend Language |

---

# Architecture Overview

```text
                 +-------------------+
                 | Browser / Users   |
                 +---------+---------+
                           |
                           v
                 +-------------------+
                 | Apache + PHP Pod  |
                 +---------+---------+
                           |
                           v
                 +-------------------+
                 | MySQL Service     |
                 +---------+---------+
                           |
                           v
                 +-------------------+
                 | MySQL Pod         |
                 +-------------------+
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

---

# Step 6: Create Namespace

```bash
kubectl create namespace lamp-stack
```

---

# Step 7: Create MySQL Deployment Manifest

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

# Step 8: Apply MySQL Deployment

```bash
kubectl apply -f mysql-deployment.yaml
```

---

# Step 9: Create MySQL Service Manifest

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

# Step 10: Apply MySQL Service

```bash
kubectl apply -f mysql-service.yaml
```

---

# Step 11: Verify MySQL Pods

```bash
kubectl get pods -n lamp-stack
```

Expected:

```text
mysql Running
```

---

# Step 12: Create Broken PHP Apache Deployment

```bash
nano php-apache.yaml
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
              value: wrong-mysql-service

            - name: DB_USER
              value: root

            - name: DB_PASSWORD
              value: wrongpassword

          ports:
            - containerPort: 80
```

---

# Step 13: Apply Broken Deployment

```bash
kubectl apply -f php-apache.yaml
```

---

# Step 14: Verify Deployments

```bash
kubectl get deployments -n lamp-stack
```

---

# Step 15: Verify Pods

```bash
kubectl get pods -n lamp-stack
```

Expected:

```text
Running Pods
```

But application connectivity will fail.

---

# Step 16: Create Apache Service

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
      nodePort: 32004
```

---

# Step 17: Apply Apache Service

```bash
kubectl apply -f apache-service.yaml
```

---

# Step 18: Verify Services

```bash
kubectl get svc -n lamp-stack
```

---

# Step 19: Check Application Logs

Get pod name:

```bash
kubectl get pods -n lamp-stack
```

Check logs:

```bash
kubectl logs <php-pod-name> -n lamp-stack
```

Expected issue:

```text
Database connection failed
```

---

# Step 20: Describe Deployment

```bash
kubectl describe deployment php-apache \
-n lamp-stack
```

Verify incorrect environment variables.

---

# Step 21: Fix MySQL Hostname

Edit deployment:

```bash
kubectl edit deployment php-apache \
-n lamp-stack
```

Replace:

```yaml
value: wrong-mysql-service
```

with:

```yaml
value: mysql-service
```

---

# Step 22: Fix Database Password

Replace:

```yaml
value: wrongpassword
```

with:

```yaml
value: root123
```

---

# Step 23: Save Deployment

Kubernetes automatically rolls out updated Pods.

---

# Step 24: Verify Rollout Status

```bash
kubectl rollout status deployment/php-apache \
-n lamp-stack
```

Expected:

```text
successfully rolled out
```

---

# Step 25: Verify Updated Pods

```bash
kubectl get pods -n lamp-stack
```

Expected:

```text
Running
```

---

# Step 26: Verify Service Connectivity

Port forward service:

```bash
kubectl port-forward svc/apache-service \
8080:80 -n lamp-stack
```

---

# Step 27: Access Application

Open browser:

```text
http://localhost:8080
```

Apache page should load successfully.

---

# Step 28: Verify MySQL Service Connectivity

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

Password:

```text
root123
```

Expected:

```text
MySQL connection successful
```

---

# Step 29: Verify Cluster Resources

```bash
kubectl get all -n lamp-stack
```

---

# Step 30: Troubleshooting Commands

| Command | Purpose |
|---|---|
| kubectl logs | Container logs |
| kubectl describe | Detailed resource info |
| kubectl get svc | Verify services |
| kubectl get endpoints | Service endpoints |
| kubectl rollout status | Deployment status |
| kubectl exec | Access containers |

---

# Step 31: Delete Resources

```bash
kubectl delete deployment php-apache -n lamp-stack

kubectl delete deployment mysql -n lamp-stack

kubectl delete svc apache-service -n lamp-stack

kubectl delete svc mysql-service -n lamp-stack
```

---

# Step 32: Delete Namespace

```bash
kubectl delete namespace lamp-stack
```

---

# Step 33: Delete Kind Cluster

```bash
kind delete cluster --name lamp-cluster
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

# php-apache.yaml

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
      nodePort: 32004
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- MySQL deployed successfully
- Apache + PHP deployed successfully
- LAMP stack communication fixed
- Service connectivity restored
- Kubernetes troubleshooting workflow implemented
- Enterprise-grade LAMP deployment operational

---

# Real Industry Use Cases

| Use Case | Description |
|---|---|
| CMS Platforms | WordPress hosting |
| Banking Apps | PHP portals |
| Enterprise Apps | Internal dashboards |
| Kubernetes Operations | App troubleshooting |
| DevOps | Production debugging |

---

# Skills Covered

- Kubernetes Troubleshooting
- LAMP Stack Deployment
- MySQL on Kubernetes
- Apache + PHP on Kubernetes
- Kubernetes Services
- Environment Variables
- DevOps Debugging
- Incident Resolution

---

# Real Enterprise Workflow

```text
Application Failure
        |
        v
Check Pods & Services
        |
        v
Inspect Logs
        |
        v
Fix Configuration
        |
        v
Rollout Updated Deployment
        |
        v
Restore Production
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

kubectl apply -f php-apache.yaml

kubectl apply -f apache-service.yaml

kubectl get pods -n lamp-stack

kubectl logs <php-pod-name> -n lamp-stack

kubectl describe deployment php-apache \
-n lamp-stack

kubectl edit deployment php-apache \
-n lamp-stack

kubectl rollout status deployment/php-apache \
-n lamp-stack

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

kubectl get all -n lamp-stack

kubectl delete namespace lamp-stack

kind delete cluster --name lamp-cluster
```