# Day 2: Deploy Applications with Kubernetes Deployments using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy scalable containerized applications using Kubernetes Deployments.

Your task is to create a Kubernetes cluster (if not already available), deploy applications using Kubernetes Deployment manifest files, and manage application replicas on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is migrating enterprise applications to Kubernetes for better scalability, reliability, and automated container orchestration.

The organization wants to deploy microservices using Kubernetes Deployments to achieve:

- Automated application rollout
- Self-healing containers
- Horizontal scaling
- High availability
- Simplified application management

Currently, the DevOps team faces several issues:

- Manual container deployment failures
- No automated replica management
- Downtime during deployments
- Inconsistent application scaling
- Difficult rollback management

To solve these problems, the organization adopted Kubernetes Deployments for managing containerized applications.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if it does not exist.
2. Deploy applications using Deployment manifests.
3. Manage application replicas.
4. Verify Deployment health and scalability.
5. Expose applications using Kubernetes Services.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Create Kubernetes Deployment manifest files.
4. Deploy applications with multiple replicas.
5. Verify Pods and Deployment health.
6. Expose applications using Services.
7. Scale Deployments dynamically.
8. Delete resources safely.

---

# Architecture Overview

```text
Developer
    |
    | kubectl apply
    |
    v
KIND Kubernetes Cluster
    |
    v
Deployment
    |
    v
ReplicaSet
    |
    v
Multiple Pods
    |
    v
NGINX Containers
```

---

# Solution

## Step 1: Update Ubuntu Packages

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

Verify Docker:

```bash
docker --version
```

Add current user to Docker group:

```bash
sudo usermod -aG docker $USER
```

Apply group changes:

```bash
newgrp docker
```

---

# Step 3: Install kubectl

```bash
sudo snap install kubectl --classic
```

Verify installation:

```bash
kubectl version --client
```

---

# Step 4: Install KIND

Download KIND:

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
```

Make executable:

```bash
chmod +x ./kind
```

Move binary:

```bash
sudo mv ./kind /usr/local/bin/kind
```

Verify installation:

```bash
kind --version
```

---

# Step 5: Create KIND Kubernetes Cluster

```bash
kind create cluster --name dev-cluster
```

Expected output:

```text
Creating cluster "dev-cluster" ...
Cluster creation complete.
```

---

# Step 6: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-dev-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                         STATUS   ROLES           AGE   VERSION
dev-cluster-control-plane   Ready    control-plane   2m    v1.30.0
```

---

# Step 7: Create Kubernetes Project Directory

```bash
mkdir -p ~/k8s-deployment-project

cd ~/k8s-deployment-project
```

---

# Step 8: Create Kubernetes Deployment Manifest File

```bash
nano nginx-deployment.yaml
```

Add the following configuration:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

  labels:
    app: nginx

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx-container

          image: nginx:latest

          ports:
            - containerPort: 80

          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"

            limits:
              memory: "128Mi"
              cpu: "500m"
```

---

# Step 9: Create Kubernetes Service Manifest File

```bash
nano nginx-service.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: nginx

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

  type: NodePort
```

---

# Complete Deployment Manifest File

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

  labels:
    app: nginx

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx-container

          image: nginx:latest

          ports:
            - containerPort: 80

          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"

            limits:
              memory: "128Mi"
              cpu: "500m"
```

---

# Complete Service Manifest File

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: nginx

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

  type: NodePort
```

---

# Step 10: Deploy Application

Deploy Deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

Deploy Service:

```bash
kubectl apply -f nginx-service.yaml
```

Expected output:

```text
deployment.apps/nginx-deployment created
service/nginx-service created
```

---

# Step 11: Verify Deployment

```bash
kubectl get deployments
```

Expected output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           1m
```

---

# Step 12: Verify Running Pods

```bash
kubectl get pods
```

Expected output:

```text
NAME                                READY   STATUS
nginx-deployment-xxxxxxxxxx-abcde   1/1     Running
nginx-deployment-xxxxxxxxxx-fghij   1/1     Running
nginx-deployment-xxxxxxxxxx-klmno   1/1     Running
```

---

# Step 13: Verify Service

```bash
kubectl get services
```

Expected output:

```text
NAME            TYPE       CLUSTER-IP      PORT(S)
nginx-service   NodePort   10.96.120.10   80:30001/TCP
```

---

# Step 14: Access Application

Get KIND cluster IP:

```bash
kubectl get nodes -o wide
```

Open browser:

```text
http://localhost:<NodePort>
```

OR use port-forwarding:

```bash
kubectl port-forward service/nginx-service 8080:80
```

Access:

```text
http://localhost:8080
```

---

# Step 15: Scale Deployment

Increase replicas:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Verify scaling:

```bash
kubectl get deployments
```

Expected output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   5/5     5            5
```

---

# Step 16: Describe Deployment

```bash
kubectl describe deployment nginx-deployment
```

Displays:

- Replica information
- Pod template
- Events
- Resource configuration

---

# Step 17: Check Pod Logs

```bash
kubectl logs <pod-name>
```

Example:

```bash
kubectl logs nginx-deployment-xxxxxxxxxx-abcde
```

---

# Step 18: Delete Resources

Delete Service:

```bash
kubectl delete -f nginx-service.yaml
```

Delete Deployment:

```bash
kubectl delete -f nginx-deployment.yaml
```

---

# Step 19: Delete KIND Cluster

```bash
kind delete cluster --name dev-cluster
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- Kubernetes Deployment configured properly
This playground allows you to play with Kubernetes
- Multiple application replicas deployed
- Service exposed successfully
- Deployment scaling working correctly
- Kubernetes orchestration operational

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Scalable application deployment |
| Platform Engineering Team | Kubernetes orchestration |
| SRE Team | High availability infrastructure |
| Cloud Team | Containerized workloads |
| Enterprise Teams | Microservices deployment |

---

# Skills Covered

- Kubernetes Deployments
- KIND
- kubectl
- ReplicaSets
- Kubernetes Services
- YAML Manifests
- Container Orchestration
- Scaling Applications
- DevOps Automation

---

# Expected Project Structure

```text
k8s-deployment-project/
├── nginx-deployment.yaml
└── nginx-service.yaml
```

---

# Complete Command Sequence

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install docker.io -y

sudo systemctl enable docker

sudo systemctl start docker

docker --version

sudo usermod -aG docker $USER

newgrp docker

sudo snap install kubectl --classic

kubectl version --client

curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind

kind --version

kind create cluster --name dev-cluster

kubectl cluster-info --context kind-dev-cluster

kubectl get nodes

mkdir -p ~/k8s-deployment-project

cd ~/k8s-deployment-project

nano nginx-deployment.yaml

nano nginx-service.yaml

kubectl apply -f nginx-deployment.yaml

kubectl apply -f nginx-service.yaml

kubectl get deployments

kubectl get pods

kubectl get services

kubectl port-forward service/nginx-service 8080:80

kubectl scale deployment nginx-deployment --replicas=5

kubectl describe deployment nginx-deployment

kubectl delete -f nginx-service.yaml

kubectl delete -f nginx-deployment.yaml

kind delete cluster --name dev-cluster
```