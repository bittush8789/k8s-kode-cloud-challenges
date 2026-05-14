# Deploy MySQL on Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy MySQL on Kubernetes for enterprise-grade database management and persistent storage.

Your task is to create a Kubernetes cluster using Kind, deploy MySQL using Kubernetes manifests, configure persistent storage, expose MySQL internally using Services, and verify deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise applications including:

- Banking systems
- AI/ML platforms
- APIs
- Enterprise dashboards
- E-commerce platforms
- Internal portals

The organization currently faces multiple database infrastructure challenges:

- Manual database management
- Database downtime
- Data loss risks
- Poor scalability
- Lack of container orchestration
- Difficult deployment management

To modernize infrastructure, the organization wants to deploy MySQL on Kubernetes.

MySQL will be used for:

- Application databases
- Persistent enterprise storage
- AI/ML metadata storage
- Backend services
- Transactional systems

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Deploy MySQL database.
3. Configure Persistent Volumes.
4. Configure Kubernetes Services.
5. Verify MySQL connectivity.
6. Scale and manage deployments.
7. Build enterprise-grade Kubernetes database architecture.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy MySQL database.
4. Configure Persistent Storage.
5. Configure Kubernetes Services.
6. Verify MySQL functionality.
7. Learn production database workflows.

---

# What is MySQL?

MySQL is a relational database management system (RDBMS) used for:

- Application databases
- Transactional systems
- Backend services
- Enterprise applications
- Data storage

---

# Architecture Overview

```text
                    +----------------------+
                    |   Client Apps        |
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
| Banking Apps | Transaction storage |
| E-commerce | Product databases |
| AI/ML Platforms | Metadata storage |
| APIs | Backend database |
| Enterprise Apps | Persistent storage |

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
kind create cluster --name mysql-cluster
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
kubectl create namespace mysql-db
```

---

# Step 8: Create Persistent Volume Manifest

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
    storage: 2Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/mysql
```

---

# Step 9: Apply Persistent Volume

```bash
kubectl apply -f mysql-pv.yaml
```

Expected output:

```text
persistentvolume/mysql-pv created
```

---

# Step 10: Create Persistent Volume Claim Manifest

```bash
nano mysql-pvc.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: mysql-pvc
  namespace: mysql-db

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 2Gi
```

---

# Step 11: Apply PVC Manifest

```bash
kubectl apply -f mysql-pvc.yaml
```

---

# Step 12: Verify PVC

```bash
kubectl get pvc -n mysql-db
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
-n mysql-db
```

Expected:

```text
secret/mysql-secret created
```

---

# Step 14: Verify Secret

```bash
kubectl get secrets -n mysql-db
```

---

# Step 15: Create MySQL Deployment Manifest

```bash
nano mysql-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: mysql-deployment
  namespace: mysql-db

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

Expected output:

```text
deployment.apps/mysql-deployment created
```

---

# Step 17: Verify Deployment

```bash
kubectl get deployments -n mysql-db
```

Expected:

```text
READY 1/1
```

---

# Step 18: Verify Pod

```bash
kubectl get pods -n mysql-db
```

Expected:

```text
Running
```

---

# Step 19: Create MySQL Service Manifest

```bash
nano mysql-service.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mysql-service
  namespace: mysql-db

spec:
  selector:
    app: mysql

  ports:
    - port: 3306
      targetPort: 3306

  type: ClusterIP
```

---

# Step 20: Apply Service Manifest

```bash
kubectl apply -f mysql-service.yaml
```

Expected output:

```text
service/mysql-service created
```

---

# Step 21: Verify Service

```bash
kubectl get svc -n mysql-db
```

Expected:

```text
mysql-service
```

---

# Step 22: Verify Endpoints

```bash
kubectl get endpoints -n mysql-db
```

Expected:

```text
Pod IPs visible
```

---

# Step 23: Verify MySQL Connectivity

Run temporary MySQL client Pod:

```bash
kubectl run mysql-client \
-it --rm \
--image=mysql:8 \
--restart=Never \
-n mysql-db -- bash
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

# Step 24: Create Database

Inside MySQL shell:

```sql
CREATE DATABASE companydb;
```

Verify:

```sql
SHOW DATABASES;
```

Expected:

```text
companydb
```

Exit MySQL:

```sql
exit
```

Exit Pod:

```bash
exit
```

---

# Step 25: Verify Persistent Storage

Check Persistent Volumes:

```bash
kubectl get pv
```

Check PVC:

```bash
kubectl get pvc -n mysql-db
```

---

# Step 26: Describe Deployment

```bash
kubectl describe deployment mysql-deployment \
-n mysql-db
```

---

# Step 27: Describe Service

```bash
kubectl describe svc mysql-service \
-n mysql-db
```

---

# Step 28: Check MySQL Logs

Get Pod name:

```bash
kubectl get pods -n mysql-db
```

Check logs:

```bash
kubectl logs <pod-name> -n mysql-db
```

---

# Step 29: Scale Deployment

```bash
kubectl scale deployment mysql-deployment \
--replicas=1 -n mysql-db
```

Note:

MySQL generally runs as a single replica unless using replication or StatefulSets.

---

# Step 30: Perform Rolling Update

Update MySQL image:

```bash
kubectl set image deployment/mysql-deployment \
mysql-container=mysql:latest \
-n mysql-db
```

Verify rollout:

```bash
kubectl rollout status deployment/mysql-deployment \
-n mysql-db
```

---

# Step 31: Verify Rollout History

```bash
kubectl rollout history deployment/mysql-deployment \
-n mysql-db
```

---

# Step 32: Rollback Deployment

```bash
kubectl rollout undo deployment/mysql-deployment \
-n mysql-db
```

---

# Step 33: Troubleshooting MySQL Issues

# Check Pods

```bash
kubectl get pods -n mysql-db
```

---

# Check Logs

```bash
kubectl logs <pod-name> -n mysql-db
```

---

# Describe Pod

```bash
kubectl describe pod <pod-name> \
-n mysql-db
```

---

# Common Issues

| Issue | Description |
|---|---|
| CrashLoopBackOff | MySQL startup failure |
| PVC Pending | Storage issue |
| Access Denied | Wrong password |
| No Endpoints | Service selector issue |
| Connection Refused | Service issue |

---

# Step 34: Verify Cluster Resources

```bash
kubectl get all -n mysql-db

kubectl get pv

kubectl get pvc -n mysql-db
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

# Step 35: Delete Resources

```bash
kubectl delete deployment mysql-deployment \
-n mysql-db

kubectl delete svc mysql-service \
-n mysql-db

kubectl delete pvc mysql-pvc \
-n mysql-db

kubectl delete pv mysql-pv

kubectl delete secret mysql-secret \
-n mysql-db
```

---

# Step 36: Delete Namespace

```bash
kubectl delete namespace mysql-db
```

---

# Step 37: Delete Kind Cluster

```bash
kind delete cluster --name mysql-cluster
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
    storage: 2Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/mysql
```

---

# mysql-pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: mysql-pvc
  namespace: mysql-db

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 2Gi
```

---

# mysql-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: mysql-deployment
  namespace: mysql-db

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

# mysql-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mysql-service
  namespace: mysql-db

spec:
  selector:
    app: mysql

  ports:
    - port: 3306
      targetPort: 3306

  type: ClusterIP
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- MySQL deployed successfully
- Persistent storage configured
- Kubernetes Service configured
- MySQL connectivity verified
- Rolling updates operational
- Rollback operational
- Enterprise-grade database deployment workflow implemented

---

# Skills Covered

- MySQL on Kubernetes
- Persistent Volumes
- Persistent Volume Claims
- Kubernetes Secrets
- Kubernetes Deployments
- Kubernetes Services
- ClusterIP Services
- Rolling Updates
- Rollbacks
- Kubernetes Troubleshooting
- DevOps
- Cloud-Native Databases

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
       v
Kubernetes Deployment
       |
       v
MySQL Pod
       |
       v
Persistent Storage
       |
       v
ClusterIP Service
       |
       v
Applications Access MySQL
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

kind create cluster --name mysql-cluster

kubectl create namespace mysql-db

kubectl apply -f mysql-pv.yaml

kubectl apply -f mysql-pvc.yaml

kubectl get pvc -n mysql-db

kubectl create secret generic mysql-secret \
--from-literal=MYSQL_ROOT_PASSWORD=root123 \
-n mysql-db

kubectl apply -f mysql-deployment.yaml

kubectl get deployments -n mysql-db

kubectl get pods -n mysql-db

kubectl apply -f mysql-service.yaml

kubectl get svc -n mysql-db

kubectl get endpoints -n mysql-db

kubectl run mysql-client \
-it --rm \
--image=mysql:8 \
--restart=Never \
-n mysql-db -- bash

mysql -h mysql-service \
-u root \
-p

kubectl describe deployment mysql-deployment \
-n mysql-db

kubectl describe svc mysql-service \
-n mysql-db

kubectl logs <pod-name> -n mysql-db

kubectl rollout history deployment/mysql-deployment \
-n mysql-db

kubectl rollout undo deployment/mysql-deployment \
-n mysql-db

kubectl get all -n mysql-db

kubectl get pv

kubectl get pvc -n mysql-db

kubectl delete namespace mysql-db

kind delete cluster --name mysql-cluster
```