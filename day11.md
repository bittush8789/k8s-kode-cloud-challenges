# Day 11: Resolve Pod Deployment Issue in Kubernetes using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries is facing Pod deployment failures inside a Kubernetes cluster.

Your task is to create a Kubernetes cluster (if not already available), identify Pod deployment issues, troubleshoot failed Pods, and resolve deployment problems on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages multiple production workloads inside Kubernetes clusters.

Recently, several application Pods failed to start due to:

- Invalid container images
- Configuration mistakes
- CrashLoopBackOff errors
- ImagePullBackOff failures
- Incorrect commands inside containers
- Resource allocation problems

The DevOps and SRE teams need a systematic troubleshooting process to identify and fix Kubernetes Pod deployment issues quickly.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if one does not exist.
2. Deploy a faulty Pod intentionally.
3. Investigate deployment issues.
4. Analyze logs and events.
5. Fix the Pod configuration.
6. Verify successful Pod recovery.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Deploy a faulty Pod.
4. Troubleshoot Pod failures.
5. Analyze logs and Kubernetes events.
6. Fix deployment configuration issues.
7. Verify healthy Pod status.

---

# Architecture Overview

```text
KIND Kubernetes Cluster
        |
        v
Faulty Pod
        |
        v
Pod Failure Investigation
        |
        +----------------+
        |                |
        v                v
Logs             Kubernetes Events
        |
        v
Fix Configuration
        |
        v
Healthy Running Pod
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
sudo apt install -y kubectl
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
kind create cluster --name troubleshooting-cluster
```

Expected output:

```text
Creating cluster "troubleshooting-cluster" ...
Cluster creation complete.
```

---

# Step 6: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-troubleshooting-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                                   STATUS   ROLES
troubleshooting-cluster-control-plane Ready    control-plane
```

---

# Step 7: Create Kubernetes Project Directory

```bash
mkdir -p ~/k8s-troubleshooting-project

cd ~/k8s-troubleshooting-project
```

---

# Step 8: Create Faulty Pod Manifest File

```bash
nano faulty-nginx-pod.yaml
```

Add the following faulty configuration:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: faulty-nginx-pod

spec:
  containers:
    - name: nginx-container

      image: nginx:wrongtag
```

---

# Complete Faulty Pod Manifest File

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: faulty-nginx-pod

spec:
  containers:
    - name: nginx-container

      image: nginx:wrongtag
```

---

# Step 9: Deploy Faulty Pod

```bash
kubectl apply -f faulty-nginx-pod.yaml
```

Expected output:

```text
pod/faulty-nginx-pod created
```

---

# Step 10: Verify Pod Failure

```bash
kubectl get pods
```

Expected output:

```text
NAME                READY   STATUS             RESTARTS
faulty-nginx-pod    0/1     ImagePullBackOff  0
```

---

# Step 11: Describe Pod for Troubleshooting

```bash
kubectl describe pod faulty-nginx-pod
```

Expected error:

```text
Failed to pull image "nginx:wrongtag"
```

This command helps identify:

- Image pull issues
- Scheduling problems
- Container startup failures
- Kubernetes events

---

# Step 12: View Pod Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Expected output:

```text
Failed to pull image "nginx:wrongtag"
ErrImagePull
ImagePullBackOff
```

---

# Step 13: Fix Pod Manifest File

Edit manifest:

```bash
nano faulty-nginx-pod.yaml
```

Replace:

```yaml
image: nginx:wrongtag
```

With:

```yaml
image: nginx:latest
```

---

# Corrected Pod Manifest File

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: faulty-nginx-pod

spec:
  containers:
    - name: nginx-container

      image: nginx:latest

      ports:
        - containerPort: 80
```

---

# Step 14: Delete Faulty Pod

```bash
kubectl delete -f faulty-nginx-pod.yaml
```

---

# Step 15: Redeploy Fixed Pod

```bash
kubectl apply -f faulty-nginx-pod.yaml
```

Expected output:

```text
pod/faulty-nginx-pod created
```

---

# Step 16: Verify Healthy Pod

```bash
kubectl get pods
```

Expected output:

```text
NAME                READY   STATUS
faulty-nginx-pod    1/1     Running
```

---

# Step 17: Check Pod Logs

```bash
kubectl logs faulty-nginx-pod
```

Expected output:

```text
nginx startup logs
```

---

# Step 18: Access Pod Shell

```bash
kubectl exec -it faulty-nginx-pod -- /bin/bash
```

OR

```bash
kubectl exec -it faulty-nginx-pod -- /bin/sh
```

Verify container:

```bash
nginx -v
```

Exit container:

```bash
exit
```

---

# Step 19: Verify Pod Networking

```bash
kubectl get pods -o wide
```

Expected output:

```text
NAME                STATUS    IP
faulty-nginx-pod    Running   10.244.0.5
```

---

# Step 20: Delete Pod

```bash
kubectl delete -f faulty-nginx-pod.yaml
```

---

# Step 21: Delete KIND Cluster

```bash
kind delete cluster --name troubleshooting-cluster
```

Expected output:

```text
Deleting cluster "troubleshooting-cluster" ...
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- Pod deployment issue reproduced successfully
- Kubernetes troubleshooting workflow implemented
- Pod failure root cause identified
- Deployment issue fixed successfully
- Healthy Pod restored successfully

---

# Common Kubernetes Pod Issues

| Issue | Description |
|---|---|
| ImagePullBackOff | Invalid or inaccessible container image |
| CrashLoopBackOff | Application crashing repeatedly |
| Pending | Scheduling/resource issues |
| ErrImagePull | Image download failure |
| OOMKilled | Memory limit exceeded |

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Kubernetes troubleshooting |
| SRE Team | Incident response |
| Platform Engineering Team | Cluster debugging |
| Cloud Team | Workload recovery |
| Enterprise Teams | Production issue resolution |

---

# Skills Covered

- Kubernetes Troubleshooting
- Pod Debugging
- ImagePullBackOff Resolution
- kubectl describe
- Kubernetes Events
- KIND
- Container Diagnostics
- Kubernetes YAML
- DevOps Incident Management

---

# Expected Project Structure

```text
k8s-troubleshooting-project/
└── faulty-nginx-pod.yaml
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

sudo apt install -y kubectl

kubectl version --client

curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind

kind --version

kind create cluster --name troubleshooting-cluster

kubectl cluster-info --context kind-troubleshooting-cluster

kubectl get nodes

mkdir -p ~/k8s-troubleshooting-project

cd ~/k8s-troubleshooting-project

nano faulty-nginx-pod.yaml

kubectl apply -f faulty-nginx-pod.yaml

kubectl get pods

kubectl describe pod faulty-nginx-pod

kubectl get events --sort-by=.metadata.creationTimestamp

nano faulty-nginx-pod.yaml

kubectl delete -f faulty-nginx-pod.yaml

kubectl apply -f faulty-nginx-pod.yaml

kubectl get pods

kubectl logs faulty-nginx-pod

kubectl exec -it faulty-nginx-pod -- /bin/bash

kubectl get pods -o wide

kubectl delete -f faulty-nginx-pod.yaml

kind delete cluster --name troubleshooting-cluster
```