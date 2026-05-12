# Day 4: Set Resource Limits in Kubernetes Pods using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to configure CPU and memory resource limits for Kubernetes Pods to optimize cluster resource utilization and prevent resource exhaustion.

Your task is to create a Kubernetes cluster (if not already available), deploy Pods with resource requests and limits using Kubernetes manifest files, and verify resource allocation on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is running multiple containerized applications inside Kubernetes clusters for:

- Web applications
- APIs
- Machine learning workloads
- Monitoring services
- Internal enterprise tools

Currently, applications are consuming uncontrolled CPU and memory resources, causing several operational issues:

- Node resource exhaustion
- Application crashes
- Unstable cluster performance
- Resource starvation for critical applications
- Increased infrastructure costs

To solve these problems, the organization wants to implement Kubernetes resource requests and limits for workload management.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if one does not exist.
2. Configure Pods with CPU and memory limits.
3. Deploy workloads using Kubernetes YAML manifests.
4. Verify resource allocation and enforcement.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Create Kubernetes Pod manifests with resource requests and limits.
4. Deploy Pods into the cluster.
5. Verify Pod resource allocation.
6. Monitor Pod resource usage.
7. Delete resources safely.

---

# Architecture Overview

```text
KIND Kubernetes Cluster
        |
        v
Resource-Limited Pod
        |
        +----------------+
        |                |
        v                v
CPU Requests/Limits   Memory Requests/Limits
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

Apply changes:

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
kind create cluster --name resource-cluster
```

Expected output:

```text
Creating cluster "resource-cluster" ...
Cluster creation complete.
```

---

# Step 6: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-resource-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                               STATUS   ROLES           AGE
resource-cluster-control-plane    Ready    control-plane   2m
```

---

# Step 7: Create Kubernetes Project Directory

```bash
mkdir -p ~/k8s-resource-project

cd ~/k8s-resource-project
```

---

# Step 8: Create Resource-Limited Pod Manifest File

```bash
nano nginx-resource-pod.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-resource-pod

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

# Complete Kubernetes Manifest File

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-resource-pod

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

# Step 9: Deploy Resource-Limited Pod

```bash
kubectl apply -f nginx-resource-pod.yaml
```

Expected output:

```text
pod/nginx-resource-pod created
```

---

# Step 10: Verify Running Pod

```bash
kubectl get pods
```

Expected output:

```text
NAME                 READY   STATUS
nginx-resource-pod   1/1     Running
```

---

# Step 11: Describe Pod Resource Allocation

```bash
kubectl describe pod nginx-resource-pod
```

Expected resource section:

```text
Limits:
  cpu:     500m
  memory:  128Mi

Requests:
  cpu:      250m
  memory:   64Mi
```

---

# Step 12: Verify Pod Resource Usage

Install metrics server (if not available):

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Check metrics server:

```bash
kubectl get deployment metrics-server -n kube-system
```

View Pod resource usage:

```bash
kubectl top pod nginx-resource-pod
```

Expected output:

```text
NAME                 CPU(cores)   MEMORY(bytes)
nginx-resource-pod   2m           15Mi
```

---

# Step 13: Access Pod Shell

```bash
kubectl exec -it nginx-resource-pod -- /bin/bash
```

OR

```bash
kubectl exec -it nginx-resource-pod -- /bin/sh
```

Verify NGINX:

```bash
nginx -v
```

Exit container:

```bash
exit
```

---

# Step 14: Verify Pod Networking

```bash
kubectl get pods -o wide
```

Expected output:

```text
NAME                 READY   STATUS    IP
nginx-resource-pod   1/1     Running   10.244.0.5
```

---

# Step 15: Delete Pod

```bash
kubectl delete -f nginx-resource-pod.yaml
```

---

# Step 16: Delete KIND Cluster

```bash
kind delete cluster --name resource-cluster
```

Expected output:

```text
Deleting cluster "resource-cluster" ...
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- Pod deployed with CPU and memory limits
- Resource requests and limits configured properly
- Kubernetes resource management operational
- Pod resource usage verified successfully

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Resource optimization |
| Platform Engineering Team | Cluster resource governance |
| SRE Team | Prevent resource exhaustion |
| Cloud Team | Infrastructure cost optimization |
| Enterprise Teams | Production workload management |

---

# Skills Covered

- Kubernetes Resource Limits
- CPU Requests and Limits
- Memory Requests and Limits
- KIND
- kubectl
- Kubernetes YAML
- Resource Monitoring
- Container Orchestration
- DevOps Automation

---

# Expected Project Structure

```text
k8s-resource-project/
└── nginx-resource-pod.yaml
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

kind create cluster --name resource-cluster

kubectl cluster-info --context kind-resource-cluster

kubectl get nodes

mkdir -p ~/k8s-resource-project

cd ~/k8s-resource-project

nano nginx-resource-pod.yaml

kubectl apply -f nginx-resource-pod.yaml

kubectl get pods

kubectl describe pod nginx-resource-pod

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl get deployment metrics-server -n kube-system

kubectl top pod nginx-resource-pod

kubectl exec -it nginx-resource-pod -- /bin/bash

kubectl get pods -o wide

kubectl delete -f nginx-resource-pod.yaml

kind delete cluster --name resource-cluster
```