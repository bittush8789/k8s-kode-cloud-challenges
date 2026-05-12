# Day 3: Setup Kubernetes Namespaces and Pods using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to organize Kubernetes workloads using Namespaces and deploy Pods inside isolated environments.

Your task is to create a Kubernetes cluster (if not already available), configure Namespaces, deploy Pods into different Namespaces using Kubernetes manifest files, and verify workload isolation on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is managing multiple environments inside Kubernetes, including:

- Development
- Testing
- Staging
- Production

Currently, all applications are deployed inside the default namespace, creating several operational problems:

- Resource conflicts between teams
- Difficult environment isolation
- Poor workload organization
- Security policy management issues
- Complex monitoring and troubleshooting

To solve these issues, the organization wants to use Kubernetes Namespaces for workload isolation and resource organization.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if one does not exist.
2. Configure Kubernetes Namespaces.
3. Deploy Pods inside specific Namespaces.
4. Verify Namespace isolation and Pod communication.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Create Kubernetes Namespaces.
4. Deploy Pods inside Namespaces.
5. Verify Pod isolation.
6. Manage Namespace resources.
7. Delete Namespaces and workloads safely.

---

# Architecture Overview

```text
KIND Kubernetes Cluster
        |
        +-------------------+
        |                   |
        v                   v
 Development Namespace   Production Namespace
        |                   |
        v                   v
   NGINX Pod           Apache Pod
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
kind create cluster --name namespace-cluster
```

Expected output:

```text
Creating cluster "namespace-cluster" ...
Cluster creation complete.
```

---

# Step 6: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-namespace-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                                STATUS   ROLES           AGE
namespace-cluster-control-plane    Ready    control-plane   2m
```

---

# Step 7: Create Project Directory

```bash
mkdir -p ~/k8s-namespace-project

cd ~/k8s-namespace-project
```

---

# Step 8: Create Namespace Manifest File

```bash
nano namespaces.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: development

---
apiVersion: v1
kind: Namespace

metadata:
  name: production
```

---

# Complete Namespace Manifest File

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: development

---
apiVersion: v1
kind: Namespace

metadata:
  name: production
```

---

# Step 9: Create Development Pod Manifest

```bash
nano dev-nginx-pod.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod
  namespace: development

  labels:
    app: nginx

spec:
  containers:
    - name: nginx-container

      image: nginx:latest

      ports:
        - containerPort: 80
```

---

# Complete Development Pod Manifest

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod
  namespace: development

  labels:
    app: nginx

spec:
  containers:
    - name: nginx-container

      image: nginx:latest

      ports:
        - containerPort: 80
```

---

# Step 10: Create Production Pod Manifest

```bash
nano prod-apache-pod.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: apache-pod
  namespace: production

  labels:
    app: apache

spec:
  containers:
    - name: apache-container

      image: httpd:latest

      ports:
        - containerPort: 80
```

---

# Complete Production Pod Manifest

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: apache-pod
  namespace: production

  labels:
    app: apache

spec:
  containers:
    - name: apache-container

      image: httpd:latest

      ports:
        - containerPort: 80
```

---

# Step 11: Create Namespaces

```bash
kubectl apply -f namespaces.yaml
```

Expected output:

```text
namespace/development created
namespace/production created
```

---

# Step 12: Verify Namespaces

```bash
kubectl get namespaces
```

Expected output:

```text
NAME           STATUS   AGE
default        Active   10m
development    Active   5s
production     Active   5s
```

---

# Step 13: Deploy Pods into Namespaces

Deploy development Pod:

```bash
kubectl apply -f dev-nginx-pod.yaml
```

Deploy production Pod:

```bash
kubectl apply -f prod-apache-pod.yaml
```

Expected output:

```text
pod/nginx-pod created
pod/apache-pod created
```

---

# Step 14: Verify Pods in Development Namespace

```bash
kubectl get pods -n development
```

Expected output:

```text
NAME        READY   STATUS
nginx-pod   1/1     Running
```

---

# Step 15: Verify Pods in Production Namespace

```bash
kubectl get pods -n production
```

Expected output:

```text
NAME         READY   STATUS
apache-pod   1/1     Running
```

---

# Step 16: Describe Namespace Resources

```bash
kubectl describe namespace development
```

```bash
kubectl describe namespace production
```

---

# Step 17: Verify Pod Networking

```bash
kubectl get pods -A -o wide
```

Expected output:

```text
NAMESPACE     NAME         STATUS    IP
development   nginx-pod   Running   10.244.0.5
production    apache-pod  Running   10.244.0.6
```

---

# Step 18: Access Pod Logs

Development Pod:

```bash
kubectl logs nginx-pod -n development
```

Production Pod:

```bash
kubectl logs apache-pod -n production
```
---

# Step 19: Delete Pods

```bash
kubectl delete -f dev-nginx-pod.yaml
```

```bash
kubectl delete -f prod-apache-pod.yaml
```

--- 

# Step 20: Delete Namespaces

```bash
kubectl delete -f namespaces.yaml
```

---

# Step 21: Delete KIND Cluster

```bash
kind delete cluster --name namespace-cluster
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- Namespaces configured correctly
- Pods deployed into isolated environments
- Namespace-level workload separation achieved
- Kubernetes resource organization improved

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Environment isolation |
| Platform Engineering Team | Multi-tenant Kubernetes |
| SRE Team | Resource segmentation |
| Cloud Team | Namespace management |
| Enterprise Teams | Production workload isolation |

---

# Skills Covered

- Kubernetes Namespaces
- KIND
- kubectl
- Kubernetes Pods
- YAML Manifests
- Namespace Isolation
- Kubernetes Resource Management
- DevOps Automation

---

# Expected Project Structure

```text
k8s-namespace-project/
├── namespaces.yaml
├── dev-nginx-pod.yaml
└── prod-apache-pod.yaml
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

kind create cluster --name namespace-cluster

kubectl cluster-info --context kind-namespace-cluster

kubectl get nodes

mkdir -p ~/k8s-namespace-project

cd ~/k8s-namespace-project

nano namespaces.yaml

nano dev-nginx-pod.yaml

nano prod-apache-pod.yaml

kubectl apply -f namespaces.yaml

kubectl get namespaces

kubectl apply -f dev-nginx-pod.yaml

kubectl apply -f prod-apache-pod.yaml

kubectl get pods -n development

kubectl get pods -n production

kubectl describe namespace development

kubectl describe namespace production

kubectl get pods -A -o wide

kubectl logs nginx-pod -n development

kubectl logs apache-pod -n production

kubectl delete -f dev-nginx-pod.yaml

kubectl delete -f prod-apache-pod.yaml

kubectl delete -f namespaces.yaml

kind delete cluster --name namespace-cluster
```