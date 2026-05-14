# Deploy Drupal App on Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy a Drupal application on Kubernetes for scalable and highly available enterprise content management.

Your task is to create a Kubernetes cluster using Kind, deploy Drupal and MySQL using Kubernetes manifests, configure persistent storage, expose the application using Services, and verify deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise-grade platforms including:

- Corporate websites
- CMS platforms
- Internal portals
- Enterprise dashboards
- Digital publishing systems

The organization currently faces several infrastructure challenges:

- Manual deployments
- Downtime during updates
- Poor scalability
- Difficult rollback management
- Lack of orchestration
- Data persistence issues

To modernize infrastructure, the organization wants to deploy Drupal on Kubernetes.

Drupal will provide:

- Content management
- Dynamic web applications
- Enterprise publishing
- User management
- CMS functionality

MySQL will provide:

- Persistent backend storage
- User data management
- Application metadata storage

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Deploy MySQL database.
3. Deploy Drupal application.
4. Configure Persistent Volumes.
5. Configure Kubernetes Services.
6. Verify application connectivity.
7. Build enterprise-grade Kubernetes architecture.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy MySQL.
4. Deploy Drupal.
5. Configure Persistent Storage.
6. Configure Services.
7. Verify application accessibility.

---

# What is Drupal?

Drupal is an open-source Content Management System (CMS) used for:

- Enterprise websites
- Digital platforms
- Corporate portals
- Government websites
- Publishing systems

---

# Architecture Overview

```text
                     +----------------------+
                     |        Users         |
                     +----------+-----------+
                                |
                                v
                     +----------------------+
                     | Drupal Service       |
                     +----------+-----------+
                                |
                                v
                     +----------------------+
                     | Drupal Pods          |
                     +----------+-----------+
                                |
                                v
                     +----------------------+
                     | MySQL Service        |
                     +----------+-----------+
                                |
                                v
                     +----------------------+
                     | MySQL Pod            |
                     +----------+-----------+
                                |
                                v
                     +----------------------+
                     | Persistent Volume    |
                     +----------------------+
```

---

# Real Industry Use Cases

| Use Case | Purpose |
|---|---|
| Corporate Websites | Enterprise content |
| Government Portals | Public services |
| CMS Platforms | Content management |
| Publishing Systems | Digital publishing |
| Enterprise Apps | Scalable architecture |

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
kind create cluster --name drupal-cluster
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
kubectl create namespace drupal-app
```

---

# Step 8: Create MySQL Persistent Volume Manifest

```bash
nano mysql-pv.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: mysql-pv

spec:
  capacity:
    storage: 5Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/drupal-mysql
```

---

# Step 9: Apply Persistent Volume

```bash
kubectl apply -f mysql-pv.yaml
```

---

# Step 10: Create Persistent Volume Claim

```bash
nano mysql-pvc.yaml
```

Add:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: mysql-pvc
  namespace: drupal-app

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 5Gi
```

---

# Step 11: Apply PVC

```bash
kubectl apply -f mysql-pvc.yaml
```

---

# Step 12: Verify PVC

```bash
kubectl get pvc -n drupal-app
```

Expected:

```text
STATUS: Bound
```

---

# Step 13: Create MySQL Secret

```bash
kubectl create secret generic mysql-secret \
--from-literal=MYSQL_ROOT_PASSWORD=root123 \
--from-literal=MYSQL_DATABASE=drupaldb \
--from-literal=MYSQL_USER=drupaluser \
--from-literal=MYSQL_PASSWORD=drupalpass \
-n drupal-app
```

---

# Step 14: Verify Secret

```bash
kubectl get secrets -n drupal-app
```

---

# Step 15: Create MySQL Deployment Manifest

```bash
nano mysql-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: mysql-deployment
  namespace: drupal-app

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

        - name: mysql-container

          image: mysql:8

          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD

            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_DATABASE

            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_USER

            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_PASSWORD

          ports:
            - containerPort: 3306

          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql

      volumes:
        - name: mysql-storage

          persistentVolumeClaim:
            claimName: mysql-pvc
```

---

# Step 16: Apply MySQL Deployment

```bash
kubectl apply -f mysql-deployment.yaml
```

---

# Step 17: Create MySQL Service

```bash
nano mysql-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mysql-service
  namespace: drupal-app

spec:
  selector:
    app: mysql

  ports:
    - port: 3306
      targetPort: 3306

  type: ClusterIP
```

---

# Step 18: Apply MySQL Service

```bash
kubectl apply -f mysql-service.yaml
```

---

# Step 19: Verify MySQL Resources

```bash
kubectl get all -n drupal-app
```

Expected:

```text
MySQL Pod Running
```

---

# Step 20: Create Drupal Deployment Manifest

```bash
nano drupal-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: drupal-deployment
  namespace: drupal-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: drupal

  template:
    metadata:
      labels:
        app: drupal

    spec:
      containers:

        - name: drupal-container

          image: drupal:10-apache

          env:
            - name: DRUPAL_DATABASE_HOST
              value: mysql-service

            - name: DRUPAL_DATABASE_NAME
              value: drupaldb

            - name: DRUPAL_DATABASE_USER
              value: drupaluser

            - name: DRUPAL_DATABASE_PASSWORD
              value: drupalpass

          ports:
            - containerPort: 80
```

---

# Step 21: Apply Drupal Deployment

```bash
kubectl apply -f drupal-deployment.yaml
```

---

# Step 22: Verify Drupal Pods

```bash
kubectl get pods -n drupal-app
```

Expected:

```text
Drupal Pods Running
```

---

# Step 23: Create Drupal Service Manifest

```bash
nano drupal-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: drupal-service
  namespace: drupal-app

spec:
  selector:
    app: drupal

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32004

  type: NodePort
```

---

# Step 24: Apply Drupal Service

```bash
kubectl apply -f drupal-service.yaml
```

---

# Step 25: Verify Drupal Service

```bash
kubectl get svc -n drupal-app
```

Expected:

```text
drupal-service
```

---

# Step 26: Verify Endpoints

```bash
kubectl get endpoints -n drupal-app
```

Expected:

```text
Pod IPs visible
```

---

# Step 27: Port Forward Drupal Service

```bash
kubectl port-forward svc/drupal-service \
8080:80 -n drupal-app
```

---

# Step 28: Access Drupal Application

Open browser:

```text
http://localhost:8080
```

Expected:

```text
Drupal installation page
```

---

# Step 29: Configure Drupal Setup

Use the following database configuration:

| Field | Value |
|---|---|
| Database Name | drupaldb |
| Database User | drupaluser |
| Database Password | drupalpass |
| Database Host | mysql-service |

---

# Step 30: Verify Application Logs

Get Pod names:

```bash
kubectl get pods -n drupal-app
```

Check Drupal logs:

```bash
kubectl logs <drupal-pod-name> \
-n drupal-app
```

Check MySQL logs:

```bash
kubectl logs <mysql-pod-name> \
-n drupal-app
```

---

# Step 31: Scale Drupal Deployment

```bash
kubectl scale deployment drupal-deployment \
--replicas=4 -n drupal-app
```

Verify:

```bash
kubectl get deployments -n drupal-app
```

Expected:

```text
READY 4/4
```

---

# Step 32: Verify Scaled Pods

```bash
kubectl get pods -n drupal-app
```

Expected:

```text
4 Drupal Pods Running
```

---

# Step 33: Perform Rolling Update

Update Drupal image:

```bash
kubectl set image deployment/drupal-deployment \
drupal-container=drupal:latest \
-n drupal-app
```

---

# Step 34: Verify Rollout Status

```bash
kubectl rollout status deployment/drupal-deployment \
-n drupal-app
```

---

# Step 35: Verify Rollout History

```bash
kubectl rollout history deployment/drupal-deployment \
-n drupal-app
```

---

# Step 36: Rollback Deployment

```bash
kubectl rollout undo deployment/drupal-deployment \
-n drupal-app
```

---

# Step 37: Troubleshooting Drupal Issues

# Check Pods

```bash
kubectl get pods -n drupal-app
```

---

# Check Logs

```bash
kubectl logs <pod-name> -n drupal-app
```

---

# Describe Pods

```bash
kubectl describe pod <pod-name> \
-n drupal-app
```

---

# Common Issues

| Issue | Description |
|---|---|
| CrashLoopBackOff | Application startup failure |
| MySQL Connection Failed | Wrong DB config |
| PVC Pending | Storage issue |
| No Endpoints | Service selector issue |
| ImagePullBackOff | Invalid image |

---

# Step 38: Verify Cluster Resources

```bash
kubectl get all -n drupal-app

kubectl get pv

kubectl get pvc -n drupal-app
```

Expected:

```text
Pods
Services
Deployments
ReplicaSets
PersistentVolumes
PersistentVolumeClaims
```

---

# Step 39: Delete Resources

```bash
kubectl delete deployment drupal-deployment \
-n drupal-app

kubectl delete deployment mysql-deployment \
-n drupal-app

kubectl delete svc drupal-service \
-n drupal-app

kubectl delete svc mysql-service \
-n drupal-app

kubectl delete pvc mysql-pvc \
-n drupal-app

kubectl delete pv mysql-pv

kubectl delete secret mysql-secret \
-n drupal-app
```

---

# Step 40: Delete Namespace

```bash
kubectl delete namespace drupal-app
```

---

# Step 41: Delete Kind Cluster

```bash
kind delete cluster --name drupal-cluster
```

---

# Manifest Files

# mysql-pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: mysql-pv

spec:
  capacity:
    storage: 5Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/drupal-mysql
```

---

# mysql-pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: mysql-pvc
  namespace: drupal-app

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 5Gi
```

---

# mysql-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: mysql-deployment
  namespace: drupal-app

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

        - name: mysql-container

          image: mysql:8

          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD

            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_DATABASE

            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_USER

            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_PASSWORD

          ports:
            - containerPort: 3306

          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql

      volumes:
        - name: mysql-storage

          persistentVolumeClaim:
            claimName: mysql-pvc
```

---

# drupal-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: drupal-deployment
  namespace: drupal-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: drupal

  template:
    metadata:
      labels:
        app: drupal

    spec:
      containers:

        - name: drupal-container

          image: drupal:10-apache

          env:
            - name: DRUPAL_DATABASE_HOST
              value: mysql-service

            - name: DRUPAL_DATABASE_NAME
              value: drupaldb

            - name: DRUPAL_DATABASE_USER
              value: drupaluser

            - name: DRUPAL_DATABASE_PASSWORD
              value: drupalpass

          ports:
            - containerPort: 80
```

---

# drupal-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: drupal-service
  namespace: drupal-app

spec:
  selector:
    app: drupal

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32004

  type: NodePort
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- MySQL deployed successfully
- Drupal deployed successfully
- Persistent storage configured
- Kubernetes Services configured
- Drupal connectivity verified
- Scaling operational
- Rolling updates operational
- Rollback operational
- Enterprise-grade CMS deployment workflow implemented

---

# Skills Covered

- Drupal on Kubernetes
- MySQL on Kubernetes
- Persistent Volumes
- Persistent Volume Claims
- Kubernetes Secrets
- Kubernetes Deployments
- Kubernetes Services
- NodePort Services
- Scaling Applications
- Rolling Updates
- Rollbacks
- Kubernetes Troubleshooting
- DevOps
- Cloud-Native CMS Deployment

---

# Real Enterprise Workflow

```text
Developer Push
       |
       v
Docker Image Build
       |
       v
Container Registry
       |
       +----------------------+
       |                      |
       v                      v
Drupal Deployment      MySQL Deployment
       |                      |
       v                      v
Drupal Pods             MySQL Pod
       |                      |
       +----------+-----------+
                  |
                  v
          Kubernetes Services
                  |
                  v
               Users
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

kind create cluster --name drupal-cluster

kubectl create namespace drupal-app

kubectl apply -f mysql-pv.yaml

kubectl apply -f mysql-pvc.yaml

kubectl create secret generic mysql-secret \
--from-literal=MYSQL_ROOT_PASSWORD=root123 \
--from-literal=MYSQL_DATABASE=drupaldb \
--from-literal=MYSQL_USER=drupaluser \
--from-literal=MYSQL_PASSWORD=drupalpass \
-n drupal-app

kubectl apply -f mysql-deployment.yaml

kubectl apply -f mysql-service.yaml

kubectl apply -f drupal-deployment.yaml

kubectl apply -f drupal-service.yaml

kubectl get pods -n drupal-app

kubectl get svc -n drupal-app

kubectl get endpoints -n drupal-app

kubectl port-forward svc/drupal-service \
8080:80 -n drupal-app

kubectl logs <pod-name> -n drupal-app

kubectl scale deployment drupal-deployment \
--replicas=4 -n drupal-app

kubectl rollout history deployment/drupal-deployment \
-n drupal-app

kubectl rollout undo deployment/drupal-deployment \
-n drupal-app

kubectl get all -n drupal-app

kubectl get pv

kubectl get pvc -n drupal-app

kubectl delete namespace drupal-app

kind delete cluster --name drupal-cluster
```