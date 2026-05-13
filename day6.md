# Day 6: Revert Deployment to Previous Version in Kubernetes using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to safely roll back Kubernetes Deployments to previous stable versions when application updates fail.

Your task is to create a Kubernetes cluster (if not already available), deploy an application using Kubernetes Deployments, perform updates, and revert the Deployment to a previous version using Kubernetes rollback functionality.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is managing production-grade applications inside Kubernetes clusters.

The organization regularly deploys:

- Feature updates
- Security patches
- Bug fixes
- Performance optimizations

Recently, a new application version introduced:

- Application crashes
- API failures
- Increased latency
- Pod instability
- User-facing downtime

The DevOps team needs a reliable rollback strategy to restore the previous stable application version quickly.

To solve these challenges, the organization wants to use Kubernetes Deployment rollback functionality.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if one does not exist.
2. Deploy applications using Kubernetes Deployments.
3. Perform rolling updates.
4. View rollout history.
5. Roll back Deployments safely to previous stable versions.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Deploy applications using Deployment manifests.
4. Execute rolling updates.
5. Verify rollout history.
6. Roll back failed Deployments.
7. Verify application recovery.
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
        +----------------------+
        |                      |
        v                      v
Old Stable Pods         New Failed Pods
        |
        v
Rollback to Previous Version
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
kind create cluster --name rollback-cluster
```

Expected output:

```text
Creating cluster "rollback-cluster" ...
Cluster creation complete.
```

---

# Step 6: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-rollback-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                               STATUS   ROLES
rollback-cluster-control-plane    Ready    control-plane
```

---

# Step 7: Create Kubernetes Project Directory

```bash
mkdir -p ~/k8s-rollback-project

cd ~/k8s-rollback-project
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

---

# Step 12: Perform Rolling Update

Update Deployment image:

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.26
```

Expected output:

```text
deployment.apps/nginx-deployment image updated
```

---

# Step 13: Verify Updated Deployment

```bash
kubectl rollout status deployment/nginx-deployment
```

Check Deployment image:

```bash
kubectl describe deployment nginx-deployment
```

Expected image:

```text
Image: nginx:1.26
```

---

# Step 14: View Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

Expected output:

```text
REVISION  CHANGE-CAUSE
1         Initial deployment
2         Updated to nginx:1.26
```

---

# Step 15: Roll Back Deployment

```bash
kubectl rollout undo deployment/nginx-deployment
```

Expected output:

```text
deployment.apps/nginx-deployment rolled back
```

---

# Step 16: Verify Rollback

```bash
kubectl rollout status deployment/nginx-deployment
```

Check Deployment image:

```bash
kubectl describe deployment nginx-deployment
```

Expected image:

```text
Image: nginx:1.25
```

---

# Step 17: Verify Pods After Rollback

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

# Step 18: Roll Back to Specific Revision

View revision history:

```bash
kubectl rollout history deployment/nginx-deployment
```

Rollback to revision 1:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

---

# Step 19: Delete Deployment

```bash
kubectl delete -f nginx-deployment.yaml
```

---

# Step 20: Delete KIND Cluster

```bash
kind delete cluster --name rollback-cluster
```

Expected output:

```text
Deleting cluster "rollback-cluster" ...
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- Deployment updates executed successfully
- Rollout history tracked correctly
- Failed application versions rolled back safely
- Previous stable version restored successfully
- Zero-downtime rollback achieved

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Production rollback management |
| Platform Engineering Team | Safe application recovery |
| SRE Team | Incident mitigation |
| Cloud Team | Automated rollback strategies |
| Enterprise Teams | High availability deployments |

---

# Skills Covered

- Kubernetes Rollback
- Rolling Updates
- Deployment History
- KIND
- kubectl
- ReplicaSets
- Kubernetes YAML
- DevOps Automation
- Zero-Downtime Recovery

---

# Expected Project Structure

```text
k8s-rollback-project/
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

sudo apt install -y kubectl

kubectl version --client

curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind

kind --version

kind create cluster --name rollback-cluster

kubectl cluster-info --context kind-rollback-cluster

kubectl get nodes

mkdir -p ~/k8s-rollback-project

cd ~/k8s-rollback-project

nano nginx-deployment.yaml

kubectl apply -f nginx-deployment.yaml

kubectl get deployments

kubectl get pods

kubectl set image deployment/nginx-deployment nginx-container=nginx:1.26

kubectl rollout status deployment/nginx-deployment

kubectl describe deployment nginx-deployment

kubectl rollout history deployment/nginx-deployment

kubectl rollout undo deployment/nginx-deployment

kubectl rollout status deployment/nginx-deployment

kubectl describe deployment nginx-deployment

kubectl get pods

kubectl rollout undo deployment/nginx-deployment --to-revision=1

kubectl delete -f nginx-deployment.yaml

kind delete cluster --name rollback-cluster
```