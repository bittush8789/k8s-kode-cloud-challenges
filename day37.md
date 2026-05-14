# Deploy Redis Deployment on Kubernetes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy Redis on Kubernetes for caching, session storage, and high-speed data access for enterprise applications.

Your task is to create a Kubernetes cluster using Kind, deploy Redis using Kubernetes manifests, expose Redis internally using Services, and verify deployment on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise-grade applications including:

- AI/ML platforms
- APIs
- E-commerce systems
- Real-time analytics
- Banking applications
- Session-based web applications

The organization currently faces several performance issues:

- Slow database queries
- High API latency
- Repeated database calls
- Poor session management
- Scalability bottlenecks

To improve performance and scalability, the organization wants to deploy Redis on Kubernetes.

Redis will be used for:

- In-memory caching
- Session storage
- Queue management
- Real-time data processing
- Distributed application caching

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Deploy Redis on Kubernetes.
3. Configure Redis Services.
4. Verify Redis connectivity.
5. Scale Redis deployments.
6. Perform rolling updates and rollbacks.
7. Build enterprise-grade Redis architecture.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy Redis.
4. Configure Services.
5. Verify Redis functionality.
6. Scale Redis deployment.
7. Learn production Redis workflows.

---

# What is Redis?

Redis is an in-memory key-value data store used for:

- Caching
- Session management
- Real-time analytics
- Messaging queues
- Fast data retrieval

---

# Architecture Overview

```text
                    +----------------------+
                    |   Client Apps        |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Redis Service        |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Redis Pods           |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Kubernetes Cluster   |
                    +----------------------+
```

---

# Real Industry Use Cases

| Use Case | Purpose |
|---|---|
| Caching | Faster applications |
| Session Store | User sessions |
| AI/ML Apps | Fast feature storage |
| Real-Time Analytics | High-speed reads |
| APIs | Reduce DB load |

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
kind create cluster --name redis-cluster
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
kubectl create namespace redis-app
```

---

# Step 8: Create Redis Deployment Manifest

```bash
nano redis-deployment.yaml
```

Add the following YAML:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: redis-deployment
  namespace: redis-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: redis

  template:
    metadata:
      labels:
        app: redis

    spec:
      containers:

        - name: redis-container

          image: redis:7

          ports:
            - containerPort: 6379
```

---

# Step 9: Apply Deployment Manifest

```bash
kubectl apply -f redis-deployment.yaml
```

Expected output:

```text
deployment.apps/redis-deployment created
```

---

# Step 10: Verify Deployment

```bash
kubectl get deployments -n redis-app
```

Expected:

```text
READY 2/2
```

---

# Step 11: Verify Pods

```bash
kubectl get pods -n redis-app
```

Expected:

```text
Running
```

---

# Step 12: Create Redis Service Manifest

```bash
nano redis-service.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: redis-service
  namespace: redis-app

spec:
  selector:
    app: redis

  ports:
    - port: 6379
      targetPort: 6379

  type: ClusterIP
```

---

# Step 13: Apply Service Manifest

```bash
kubectl apply -f redis-service.yaml
```

Expected output:

```text
service/redis-service created
```

---

# Step 14: Verify Service

```bash
kubectl get svc -n redis-app
```

Expected:

```text
redis-service
```

---

# Step 15: Verify Endpoints

```bash
kubectl get endpoints -n redis-app
```

Expected:

```text
Pod IPs visible
```

---

# Step 16: Access Redis Pod

Get Pod name:

```bash
kubectl get pods -n redis-app
```

Access Pod shell:

```bash
kubectl exec -it <pod-name> \
-n redis-app -- sh
```

---

# Step 17: Verify Redis Server

Inside container:

```bash
redis-cli ping
```

Expected:

```text
PONG
```

---

# Step 18: Store Redis Data

Inside container:

```bash
redis-cli
```

Set value:

```bash
SET username devops
```

Expected:

```text
OK
```

Retrieve value:

```bash
GET username
```

Expected:

```text
"devops"
```

Exit Redis CLI:

```bash
exit
```

Exit container:

```bash
exit
```

---

# Step 19: Test Redis Service Connectivity

Run temporary Redis client Pod:

```bash
kubectl run redis-client \
-it --rm \
--image=redis:7 \
--restart=Never \
-n redis-app -- sh
```

Inside Pod:

```bash
redis-cli -h redis-service
```

Test connectivity:

```bash
PING
```

Expected:

```text
PONG
```

Set test key:

```bash
SET company xFusionCorp
```

Retrieve key:

```bash
GET company
```

Expected:

```text
"xFusionCorp"
```

Exit:

```bash
exit
```

---

# Step 20: Scale Redis Deployment

```bash
kubectl scale deployment redis-deployment \
--replicas=4 -n redis-app
```

Verify:

```bash
kubectl get deployments -n redis-app
```

Expected:

```text
READY 4/4
```

---

# Step 21: Verify Scaled Pods

```bash
kubectl get pods -n redis-app
```

Expected:

```text
4 Running Pods
```

---

# Step 22: Describe Deployment

```bash
kubectl describe deployment redis-deployment \
-n redis-app
```

---

# Step 23: Describe Service

```bash
kubectl describe svc redis-service \
-n redis-app
```

---

# Step 24: Check Application Logs

Get Pod name:

```bash
kubectl get pods -n redis-app
```

Check logs:

```bash
kubectl logs <pod-name> -n redis-app
```

---

# Step 25: Perform Rolling Update

Update Redis version:

```bash
kubectl set image deployment/redis-deployment \
redis-container=redis:latest \
-n redis-app
```

Verify rollout:

```bash
kubectl rollout status deployment/redis-deployment \
-n redis-app
```

---

# Step 26: Verify Rollout History

```bash
kubectl rollout history deployment/redis-deployment \
-n redis-app
```

---

# Step 27: Rollback Deployment

```bash
kubectl rollout undo deployment/redis-deployment \
-n redis-app
```

---

# Step 28: Troubleshooting Redis Issues

# Check Pods

```bash
kubectl get pods -n redis-app
```

---

# Check Logs

```bash
kubectl logs <pod-name> -n redis-app
```

---

# Check Services

```bash
kubectl get svc -n redis-app
```

---

# Common Issues

| Issue | Description |
|---|---|
| CrashLoopBackOff | Redis crash |
| ImagePullBackOff | Invalid image |
| No Endpoints | Wrong selector |
| Connection Refused | Port issue |
| Service Unreachable | Network issue |

---

# Step 29: Verify Cluster Resources

```bash
kubectl get all -n redis-app
```

Expected:

```text
Pods
Services
Deployments
ReplicaSets
```

---

# Step 30: Delete Resources

```bash
kubectl delete deployment redis-deployment \
-n redis-app

kubectl delete svc redis-service \
-n redis-app
```

---

# Step 31: Delete Namespace

```bash
kubectl delete namespace redis-app
```

---

# Step 32: Delete Kind Cluster

```bash
kind delete cluster --name redis-cluster
```

---

# Manifest Files

# redis-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: redis-deployment
  namespace: redis-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: redis

  template:
    metadata:
      labels:
        app: redis

    spec:
      containers:

        - name: redis-container

          image: redis:7

          ports:
            - containerPort: 6379
```

---

# redis-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: redis-service
  namespace: redis-app

spec:
  selector:
    app: redis

  ports:
    - port: 6379
      targetPort: 6379

  type: ClusterIP
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Redis deployed successfully
- Kubernetes Service configured
- Redis connectivity verified
- Redis scaling operational
- Rolling updates operational
- Rollback operational
- Enterprise-grade Redis deployment workflow implemented

---

# Skills Covered

- Redis on Kubernetes
- Kubernetes Deployments
- Kubernetes Services
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
       v
Kubernetes Deployment
       |
       v
Redis Pods
       |
       v
ClusterIP Service
       |
       v
Applications Access Redis
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

kind create cluster --name redis-cluster

kubectl create namespace redis-app

kubectl apply -f redis-deployment.yaml

kubectl get deployments -n redis-app

kubectl get pods -n redis-app

kubectl apply -f redis-service.yaml

kubectl get svc -n redis-app

kubectl get endpoints -n redis-app

kubectl exec -it <pod-name> \
-n redis-app -- sh

redis-cli ping

kubectl run redis-client \
-it --rm \
--image=redis:7 \
--restart=Never \
-n redis-app -- sh

redis-cli -h redis-service

kubectl scale deployment redis-deployment \
--replicas=4 -n redis-app

kubectl describe deployment redis-deployment \
-n redis-app

kubectl describe svc redis-service \
-n redis-app

kubectl logs <pod-name> -n redis-app

kubectl rollout history deployment/redis-deployment \
-n redis-app

kubectl rollout undo deployment/redis-deployment \
-n redis-app

kubectl get all -n redis-app

kubectl delete namespace redis-app

kind delete cluster --name redis-cluster
```