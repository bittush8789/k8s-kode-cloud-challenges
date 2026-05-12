# Day 1: Deploy Pods in Kubernetes Cluster using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy containerized applications inside a Kubernetes cluster using KIND (Kubernetes IN Docker) for local development and testing.

Your task is to create a Kubernetes cluster using KIND, deploy Pods using Kubernetes manifest files, and verify the application deployment on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is adopting Kubernetes for cloud-native application deployment and container orchestration.

The organization wants a lightweight Kubernetes environment for:

- Local Kubernetes testing
- CI/CD integration
- Development environments
- Container orchestration learning
- Rapid Kubernetes deployments

Currently, the DevOps team faces several issues:

- No standardized Kubernetes development environment
- Difficulty testing Kubernetes manifests locally
- Slow infrastructure provisioning
- Inconsistent deployment environments
- Manual container orchestration workflows

To solve these problems, the organization decided to use KIND (Kubernetes IN Docker) for lightweight Kubernetes cluster management.

As a DevOps Engineer, your responsibility is to:

1. Install Docker and Kubernetes tools.
2. Create a Kubernetes cluster using KIND.
3. Configure kubectl access.
4. Deploy application Pods using Kubernetes YAML manifests.
5. Verify Pod health, logs, and networking.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Verify Kubernetes cluster connectivity.
4. Create Kubernetes manifest files.
5. Deploy Pods into the cluster.
6. Verify application deployment.
7. Access Pod logs and shell.
8. Delete Pods and cluster safely.

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
Pod
    |
    v
NGINX Container
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

Download KIND binary:

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

Check cluster info:

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
mkdir -p ~/k8s-kind-project

cd ~/k8s-kind-project
```

---

# Step 8: Create Kubernetes Manifest File

```bash
nano nginx-pod.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod

  labels:
    app: nginx
    environment: development

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
  name: nginx-pod

  labels:
    app: nginx
    environment: development

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

# Step 9: Deploy Pod into Kubernetes Cluster

```bash
kubectl apply -f nginx-pod.yaml
```

Expected output:

```text
pod/nginx-pod created
```

---

# Step 10: Verify Running Pods

```bash
kubectl get pods
```

Expected output:

```text
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          15s
```

---

# Step 11: Describe Pod Details

```bash
kubectl describe pod nginx-pod
```

This displays:

- Pod IP
- Container details
- Events
- Node assignment
- Resource information

---

# Step 12: Check Pod Logs

```bash
kubectl logs nginx-pod
```

Expected output:

```text
nginx startup logs
```

---

# Step 13: Access Pod Shell

```bash
kubectl exec -it nginx-pod -- /bin/bash
```

OR

```bash
kubectl exec -it nginx-pod -- /bin/sh
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
NAME        READY   STATUS    IP           NODE
nginx-pod   1/1     Running   10.244.0.5   dev-cluster-control-plane
```

---

# Step 15: Delete Pod

```bash
kubectl delete pod nginx-pod
```

Expected output:

```text
pod "nginx-pod" deleted
```

---

# Step 16: Delete KIND Cluster

```bash
kind delete cluster --name dev-cluster
```

Expected output:

```text
Deleting cluster "dev-cluster" ...
Deleted nodes: ["dev-cluster-control-plane"]
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- kubectl configured correctly
- Pod deployed successfully using YAML manifest
- NGINX container running properly
- Pod networking and logs verified
- Kubernetes development environment operational

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Kubernetes local testing |
| Platform Engineering Team | Cluster validation |
| CI/CD Team | Pipeline testing |
| SRE Team | Infrastructure simulation |
| Enterprise Teams | Local Kubernetes development |

---

# Skills Covered

- Kubernetes
- KIND
- kubectl
- Pod Deployment
- Kubernetes YAML
- Container Orchestration
- Docker
- DevOps Automation
- Ubuntu Administration

---

# Expected Project Structure

```text
k8s-kind-project/
└── nginx-pod.yaml
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

mkdir -p ~/k8s-kind-project

cd ~/k8s-kind-project

nano nginx-pod.yaml

kubectl apply -f nginx-pod.yaml

kubectl get pods

kubectl describe pod nginx-pod

kubectl logs nginx-pod

kubectl exec -it nginx-pod -- /bin/bash

kubectl get pods -o wide

kubectl delete pod nginx-pod

kind delete cluster --name dev-cluster
```