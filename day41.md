# Deploy Guest Book App on Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy a Guest Book application on Kubernetes for scalable and highly available web application hosting.

Your task is to create a Kubernetes cluster using Kind, deploy Redis and Guest Book frontend applications using Kubernetes manifests, expose the application using Services, and verify deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise-grade applications including:

- Web applications
- Microservices
- Enterprise portals
- Internal dashboards
- Customer-facing platforms

The organization currently faces several operational challenges:

- Manual deployments
- Downtime during updates
- Poor scalability
- Lack of orchestration
- High application latency

To modernize infrastructure, the organization wants to deploy the Guest Book application on Kubernetes.

The Guest Book application will use:

- Redis Master for data storage
- Redis Replicas for scalability
- Frontend application for user interaction

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Deploy Redis Master.
3. Deploy Redis Replicas.
4. Deploy Guest Book frontend.
5. Configure Kubernetes Services.
6. Verify application connectivity.
7. Build enterprise-grade Kubernetes architecture.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy Redis Master.
4. Deploy Redis Replicas.
5. Deploy Frontend Application.
6. Configure Services.
7. Verify application accessibility.

---

# Architecture Overview

```text
                     +----------------------+
                     |        Users         |
                     +----------+-----------+
                                |
                                v
                     +----------------------+
                     | Frontend Service     |
                     +----------+-----------+
                                |
                                v
                     +----------------------+
                     | Frontend Pods        |
                     +----------+-----------+
                                |
                                v
               +----------------+----------------+
               |                                 |
               v                                 v
      +------------------+             +------------------+
      | Redis Master     |             | Redis Replicas   |
      +------------------+             +------------------+
```

---

# Real Industry Use Cases

| Use Case | Purpose |
|---|---|
| Enterprise Apps | Scalable frontend |
| APIs | Backend caching |
| Microservices | Distributed architecture |
| Web Platforms | Session management |
| Cloud Apps | High availability |

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
kind create cluster --name guestbook-cluster
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
kubectl create namespace guestbook-app
```

---

# Step 8: Create Redis Master Deployment Manifest

```bash
nano redis-master-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: redis-master
  namespace: guestbook-app

spec:
  replicas: 1

  selector:
    matchLabels:
      app: redis
      role: master

  template:
    metadata:
      labels:
        app: redis
        role: master

    spec:
      containers:

        - name: redis-master-container

          image: redis:7

          ports:
            - containerPort: 6379
```

---

# Step 9: Apply Redis Master Deployment

```bash
kubectl apply -f redis-master-deployment.yaml
```

Expected output:

```text
deployment.apps/redis-master created
```

---

# Step 10: Create Redis Master Service Manifest

```bash
nano redis-master-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: redis-master-service
  namespace: guestbook-app

spec:
  selector:
    app: redis
    role: master

  ports:
    - port: 6379
      targetPort: 6379

  type: ClusterIP
```

---

# Step 11: Apply Redis Master Service

```bash
kubectl apply -f redis-master-service.yaml
```

---

# Step 12: Verify Redis Master

```bash
kubectl get all -n guestbook-app
```

Expected:

```text
Redis Master Running
```

---

# Step 13: Create Redis Replica Deployment Manifest

```bash
nano redis-replica-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: redis-replica
  namespace: guestbook-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: redis
      role: replica

  template:
    metadata:
      labels:
        app: redis
        role: replica

    spec:
      containers:

        - name: redis-replica-container

          image: redis:7

          ports:
            - containerPort: 6379
```

---

# Step 14: Apply Redis Replica Deployment

```bash
kubectl apply -f redis-replica-deployment.yaml
```

---

# Step 15: Create Redis Replica Service

```bash
nano redis-replica-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: redis-replica-service
  namespace: guestbook-app

spec:
  selector:
    app: redis
    role: replica

  ports:
    - port: 6379
      targetPort: 6379

  type: ClusterIP
```

---

# Step 16: Apply Redis Replica Service

```bash
kubectl apply -f redis-replica-service.yaml
```

---

# Step 17: Verify Redis Replicas

```bash
kubectl get pods -n guestbook-app
```

Expected:

```text
Redis Replica Pods Running
```

---

# Step 18: Create Frontend Deployment Manifest

```bash
nano frontend-deployment.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: frontend
  namespace: guestbook-app

spec:
  replicas: 3

  selector:
    matchLabels:
      app: guestbook
      tier: frontend

  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend

    spec:
      containers:

        - name: php-redis

          image: gcr.io/google-samples/gb-frontend:v5

          ports:
            - containerPort: 80
```

---

# Step 19: Apply Frontend Deployment

```bash
kubectl apply -f frontend-deployment.yaml
```

Expected output:

```text
deployment.apps/frontend created
```

---

# Step 20: Create Frontend Service Manifest

```bash
nano frontend-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: frontend-service
  namespace: guestbook-app

spec:
  selector:
    app: guestbook
    tier: frontend

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32005

  type: NodePort
```

---

# Step 21: Apply Frontend Service

```bash
kubectl apply -f frontend-service.yaml
```

---

# Step 22: Verify Frontend Pods

```bash
kubectl get pods -n guestbook-app
```

Expected:

```text
Frontend Pods Running
```

---

# Step 23: Verify Services

```bash
kubectl get svc -n guestbook-app
```

Expected:

```text
frontend-service
redis-master-service
redis-replica-service
```

---

# Step 24: Verify Endpoints

```bash
kubectl get endpoints -n guestbook-app
```

Expected:

```text
Pod IPs visible
```

---

# Step 25: Port Forward Frontend Service

```bash
kubectl port-forward svc/frontend-service \
8080:80 -n guestbook-app
```

---

# Step 26: Access Guest Book Application

Open browser:

```text
http://localhost:8080
```

Expected:

```text
Guest Book application page
```

---

# Step 27: Verify Redis Connectivity

Run temporary Pod:

```bash
kubectl run redis-client \
-it --rm \
--image=redis:7 \
--restart=Never \
-n guestbook-app -- sh
```

Inside Pod:

```bash
redis-cli -h redis-master-service
```

Test Redis:

```bash
PING
```

Expected:

```text
PONG
```

Exit:

```bash
exit
```

---

# Step 28: Check Application Logs

Get Pod names:

```bash
kubectl get pods -n guestbook-app
```

Check frontend logs:

```bash
kubectl logs <frontend-pod-name> \
-n guestbook-app
```

Check Redis logs:

```bash
kubectl logs <redis-pod-name> \
-n guestbook-app
```

---

# Step 29: Scale Frontend Deployment

```bash
kubectl scale deployment frontend \
--replicas=5 -n guestbook-app
```

Verify:

```bash
kubectl get deployments -n guestbook-app
```

Expected:

```text
READY 5/5
```

---

# Step 30: Scale Redis Replicas

```bash
kubectl scale deployment redis-replica \
--replicas=4 -n guestbook-app
```

Verify:

```bash
kubectl get deployments -n guestbook-app
```

Expected:

```text
READY 4/4
```

---

# Step 31: Verify Scaled Pods

```bash
kubectl get pods -n guestbook-app
```

Expected:

```text
All Pods Running
```

---

# Step 32: Perform Rolling Update

Update frontend image:

```bash
kubectl set image deployment/frontend \
php-redis=gcr.io/google-samples/gb-frontend:v6 \
-n guestbook-app
```

---

# Step 33: Verify Rollout Status

```bash
kubectl rollout status deployment/frontend \
-n guestbook-app
```

---

# Step 34: Verify Rollout History

```bash
kubectl rollout history deployment/frontend \
-n guestbook-app
```

---

# Step 35: Rollback Deployment

```bash
kubectl rollout undo deployment/frontend \
-n guestbook-app
```

---

# Step 36: Troubleshooting Guest Book Issues

# Check Pods

```bash
kubectl get pods -n guestbook-app
```

---

# Check Logs

```bash
kubectl logs <pod-name> -n guestbook-app
```

---

# Describe Pods

```bash
kubectl describe pod <pod-name> \
-n guestbook-app
```

---

# Common Issues

| Issue | Description |
|---|---|
| CrashLoopBackOff | Application crash |
| Redis Connection Failed | Service issue |
| No Endpoints | Wrong selector |
| ImagePullBackOff | Invalid image |
| Frontend Unreachable | Service issue |

---

# Step 37: Verify Cluster Resources

```bash
kubectl get all -n guestbook-app
```

Expected:

```text
Pods
Services
Deployments
ReplicaSets
```

---

# Step 38: Delete Resources

```bash
kubectl delete deployment frontend \
-n guestbook-app

kubectl delete deployment redis-master \
-n guestbook-app

kubectl delete deployment redis-replica \
-n guestbook-app

kubectl delete svc frontend-service \
-n guestbook-app

kubectl delete svc redis-master-service \
-n guestbook-app

kubectl delete svc redis-replica-service \
-n guestbook-app
```

---

# Step 39: Delete Namespace

```bash
kubectl delete namespace guestbook-app
```

---

# Step 40: Delete Kind Cluster

```bash
kind delete cluster --name guestbook-cluster
```

---

# Manifest Files

# redis-master-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: redis-master
  namespace: guestbook-app

spec:
  replicas: 1

  selector:
    matchLabels:
      app: redis
      role: master

  template:
    metadata:
      labels:
        app: redis
        role: master

    spec:
      containers:

        - name: redis-master-container

          image: redis:7

          ports:
            - containerPort: 6379
```

---

# redis-replica-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: redis-replica
  namespace: guestbook-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: redis
      role: replica

  template:
    metadata:
      labels:
        app: redis
        role: replica

    spec:
      containers:

        - name: redis-replica-container

          image: redis:7

          ports:
            - containerPort: 6379
```

---

# frontend-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: frontend
  namespace: guestbook-app

spec:
  replicas: 3

  selector:
    matchLabels:
      app: guestbook
      tier: frontend

  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend

    spec:
      containers:

        - name: php-redis

          image: gcr.io/google-samples/gb-frontend:v5

          ports:
            - containerPort: 80
```

---

# frontend-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: frontend-service
  namespace: guestbook-app

spec:
  selector:
    app: guestbook
    tier: frontend

  ports:
    - port: 80
      targetPort: 80
      nodePort: 32005

  type: NodePort
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Redis Master deployed successfully
- Redis Replicas deployed successfully
- Guest Book frontend deployed successfully
- Kubernetes Services configured
- Redis connectivity verified
- Scaling operational
- Rolling updates operational
- Rollback operational
- Enterprise-grade application deployment workflow implemented

---

# Skills Covered

- Redis on Kubernetes
- Kubernetes Deployments
- Kubernetes Services
- NodePort Services
- ClusterIP Services
- Scaling Applications
- Rolling Updates
- Rollbacks
- Kubernetes Troubleshooting
- DevOps
- Cloud-Native Deployment

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
Frontend Deployment     Redis Deployments
       |                      |
       v                      v
Frontend Pods          Redis Pods
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

kind create cluster --name guestbook-cluster

kubectl create namespace guestbook-app

kubectl apply -f redis-master-deployment.yaml

kubectl apply -f redis-master-service.yaml

kubectl apply -f redis-replica-deployment.yaml

kubectl apply -f redis-replica-service.yaml

kubectl apply -f frontend-deployment.yaml

kubectl apply -f frontend-service.yaml

kubectl get pods -n guestbook-app

kubectl get svc -n guestbook-app

kubectl get endpoints -n guestbook-app

kubectl port-forward svc/frontend-service \
8080:80 -n guestbook-app

kubectl run redis-client \
-it --rm \
--image=redis:7 \
--restart=Never \
-n guestbook-app -- sh

redis-cli -h redis-master-service

kubectl scale deployment frontend \
--replicas=5 -n guestbook-app

kubectl scale deployment redis-replica \
--replicas=4 -n guestbook-app

kubectl rollout history deployment/frontend \
-n guestbook-app

kubectl rollout undo deployment/frontend \
-n guestbook-app

kubectl get all -n guestbook-app

kubectl delete namespace guestbook-app

kind delete cluster --name guestbook-cluster
```