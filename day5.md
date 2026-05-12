# Day 5: Execute Rolling Updates in Kubernetes using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to perform rolling updates on Kubernetes Deployments to upgrade applications without downtime.

Your task is to create a Kubernetes cluster (if not already available), deploy an application using Kubernetes Deployments, execute rolling updates, and verify zero-downtime deployment behavior on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is running production-grade applications inside Kubernetes clusters.

The organization frequently releases:

- New application features
- Security patches
- Bug fixes
- Performance improvements

Currently, application updates are causing several operational problems:

- Application downtime during deployments
- Failed upgrades impacting users
- No rollback strategy
- Inconsistent application versions
- Manual deployment management

To solve these challenges, the organization wants to implement Kubernetes Rolling Updates for zero-downtime application deployment.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if one does not exist.
2. Deploy applications using Kubernetes Deployments.
3. Perform rolling updates safely.
4. Monitor rollout status and application health.
5. Roll back deployments if failures occur.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Deploy an application using Deployment manifests.
4. Execute rolling updates using new container images.
5. Monitor rollout progress.
6. Verify zero-downtime deployment.
7. Roll back failed updates.
8. Delete resources safely.

---

# Architecture Overview

```text
KIND Kubernetes Cluster
        |
        v
Deployment
        |
        v
ReplicaSet
        |
        v
Pods (v1 → v2 Rolling Update)
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
kind create cluster --name rolling-update-cluster
```

Expected output:

```text
Creating cluster "rolling-update-cluster" ...
Cluster creation complete.
```

---

# Step 6: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-rolling-update-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                                     STATUS   ROLES
rolling-update-cluster-control-plane    Ready    control-plane
```

---

# Step 7: Create Kubernetes Project Directory

```bash
mkdir -p ~/k8s-rolling-update-project

cd ~/k8s-rolling-update-project
```

---

# Step 8: Create Deployment Manifest File

```bash
nano nginx-deployment.yaml
```

Add the following configuration:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

  strategy:
    type: RollingUpdate

    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1

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

          image: nginx:1.25

          ports:
            - containerPort: 80
```

---

# Complete Deployment Manifest File

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

  strategy:
    type: RollingUpdate

    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1

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

          image: nginx:1.25

          ports:
            - containerPort: 80
```

---

# Step 9: Deploy Application

```bash
kubectl apply -f nginx-deployment.yaml
```

Expected output:

```text
deployment.apps/nginx-deployment created
```

---

# Step 10: Verify Deployment

```bash
kubectl get deployments
```

Expected output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   3/3     3            3
```

---

# Step 11: Verify Running Pods

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

# Step 12: Execute Rolling Update

Update container image:

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.26
```

Expected output:

```text
deployment.apps/nginx-deployment image updated
```

---

# Step 13: Monitor Rolling Update

```bash
kubectl rollout status deployment/nginx-deployment
```

Expected output:

```text
deployment "nginx-deployment" successfully rolled out
```

Watch rolling update process:

```bash
kubectl get pods -w
```

You will observe:

- Old Pods terminating gradually
- New Pods starting automatically
- Zero downtime maintained

Stop watch mode:

```bash
CTRL + C
```

---

# Step 14: Verify Updated Pods

```bash
kubectl describe deployment nginx-deployment
```

Expected image:

```text
Image: nginx:1.26
```

---

# Step 15: Verify Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

Expected output:

```text
REVISION  CHANGE-CAUSE
1         Initial deployment
2         Rolling update to nginx:1.26
```

---

# Step 16: Roll Back Deployment

```bash
kubectl rollout undo deployment/nginx-deployment
```

Expected output:

```text
deployment.apps/nginx-deployment rolled back
```

Verify rollback:

```bash
kubectl describe deployment nginx-deployment
```

Expected image:

```text
Image: nginx:1.25
```

---

# Step 17: Verify Deployment After Rollback

```bash
kubectl get deployments
```

Expected output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   3/3     3            3
```

---

# Step 18: Delete Deployment

```bash
kubectl delete -f nginx-deployment.yaml
```

---

# Step 19: Delete KIND Cluster

```bash
kind delete cluster --name rolling-update-cluster
```

Expected output:

```text
Deleting cluster "rolling-update-cluster" ...
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- Kubernetes Deployment configured properly
- Rolling updates executed successfully
- Zero-downtime deployment achieved
- Rollback functionality verified
- Kubernetes deployment lifecycle operational

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Zero-downtime deployments |
| Platform Engineering Team | Automated application upgrades |
| SRE Team | Safe production rollouts |
| Cloud Team | Deployment automation |
| Enterprise Teams | High availability releases |

---

# Skills Covered

- Kubernetes Rolling Updates
- Deployments
- ReplicaSets
- KIND
- kubectl
- Rollback Management
- Zero-Downtime Deployment
- Kubernetes YAML
- DevOps Automation

---

# Expected Project Structure

```text
k8s-rolling-update-project/
└── nginx-deployment.yaml
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

kind create cluster --name rolling-update-cluster

kubectl cluster-info --context kind-rolling-update-cluster

kubectl get nodes

mkdir -p ~/k8s-rolling-update-project

cd ~/k8s-rolling-update-project

nano nginx-deployment.yaml

kubectl apply -f nginx-deployment.yaml

kubectl get deployments

kubectl get pods

kubectl set image deployment/nginx-deployment nginx-container=nginx:1.26

kubectl rollout status deployment/nginx-deployment

kubectl get pods -w

kubectl describe deployment nginx-deployment

kubectl rollout history deployment/nginx-deployment

kubectl rollout undo deployment/nginx-deployment

kubectl describe deployment nginx-deployment

kubectl delete -f nginx-deployment.yaml

kind delete cluster --name rolling-update-cluster
``` 