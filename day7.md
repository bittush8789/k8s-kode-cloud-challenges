# Day 7: Deploy ReplicaSet in Kubernetes Cluster using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to deploy and manage multiple identical Pods using Kubernetes ReplicaSets to ensure application availability and fault tolerance.

Your task is to create a Kubernetes cluster (if not already available), deploy a ReplicaSet using Kubernetes manifest files, and verify automatic Pod replication on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is running containerized applications on Kubernetes infrastructure.

The organization requires:

- High availability applications
- Automatic Pod recovery
- Self-healing infrastructure
- Scalable workloads
- Consistent application replicas

Currently, the DevOps team faces several operational issues:

- Single Pod failures causing downtime
- Manual Pod recreation processes
- Inconsistent application scaling
- Poor workload resilience
- No automated fault recovery

To solve these problems, the organization wants to use Kubernetes ReplicaSets for maintaining desired Pod replicas automatically.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if one does not exist.
2. Deploy applications using ReplicaSets.
3. Maintain desired replica counts.
4. Verify automatic Pod recovery and self-healing.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Create ReplicaSet manifest files.
4. Deploy ReplicaSets into the cluster.
5. Verify Pod replication.
6. Test self-healing functionality.
7. Scale ReplicaSets dynamically.
8. Delete resources safely.

---

# Architecture Overview

```text
KIND Kubernetes Cluster
        |
        v
ReplicaSet
        |
        +-------------------+
        |        |          |
        v        v          v
      Pod-1    Pod-2      Pod-3
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
kind create cluster --name replicaset-cluster
```

Expected output:

```text
Creating cluster "replicaset-cluster" ...
Cluster creation complete.
```

---

# Step 6: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-replicaset-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                                  STATUS   ROLES
replicaset-cluster-control-plane     Ready    control-plane
```

---

# Step 7: Create Kubernetes Project Directory

```bash
mkdir -p ~/k8s-replicaset-project

cd ~/k8s-replicaset-project
```

---

# Step 8: Create ReplicaSet Manifest File

```bash
nano nginx-replicaset.yaml
```

Add the following configuration:

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: nginx-replicaset

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
```

---

# Complete ReplicaSet Manifest File

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: nginx-replicaset

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
```

---

# Step 9: Deploy ReplicaSet

```bash
kubectl apply -f nginx-replicaset.yaml
```

Expected output:

```text
replicaset.apps/nginx-replicaset created
```

---

# Step 10: Verify ReplicaSet

```bash
kubectl get replicasets
```

Expected output:

```text
NAME                DESIRED   CURRENT   READY
nginx-replicaset    3         3         3
```

---

# Step 11: Verify Running Pods

```bash
kubectl get pods
```

Expected output:

```text
NAME                      READY   STATUS
nginx-replicaset-abcde    1/1     Running
nginx-replicaset-fghij    1/1     Running
nginx-replicaset-klmno    1/1     Running
```

---

# Step 12: Describe ReplicaSet

```bash
kubectl describe replicaset nginx-replicaset
```

Displays:

- Replica information
- Pod template
- Events
- Current status

---

# Step 13: Test Self-Healing Capability

Delete one Pod manually:

```bash
kubectl delete pod <pod-name>
```

Example:

```bash
kubectl delete pod nginx-replicaset-abcde
```

Verify Pods again:

```bash
kubectl get pods
```

Expected behavior:

- Deleted Pod automatically recreated
- Replica count maintained at 3

---

# Step 14: Scale ReplicaSet

Increase replicas:

```bash
kubectl scale replicaset nginx-replicaset --replicas=5
```

Verify scaling:

```bash
kubectl get replicasets
```

Expected output:

```text
NAME                DESIRED   CURRENT   READY
nginx-replicaset    5         5         5
```

---

# Step 15: Verify Pods After Scaling

```bash
kubectl get pods
```

Expected output:

```text
5 Pods running successfully
```

---

# Step 16: Verify Pod Networking

```bash
kubectl get pods -o wide
```

Expected output:

```text
NAME                      READY   STATUS    IP
nginx-replicaset-xxxxx    1/1     Running   10.244.0.x
```

---

# Step 17: Access Pod Logs

```bash
kubectl logs <pod-name>
```

Example:

```bash
kubectl logs nginx-replicaset-fghij
```

---

# Step 18: Delete ReplicaSet

```bash
kubectl delete -f nginx-replicaset.yaml
```

Expected output:

```text
replicaset.apps "nginx-replicaset" deleted
```

---

# Step 19: Delete KIND Cluster

```bash
kind delete cluster --name replicaset-cluster
```

Expected output:

```text
Deleting cluster "replicaset-cluster" ...
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- ReplicaSet deployed successfully
- Multiple Pods managed automatically
- Self-healing functionality verified
- Scaling operations completed successfully
- Kubernetes workload resilience achieved

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Application availability |
| Platform Engineering Team | Pod replication |
| SRE Team | Fault tolerance |
| Cloud Team | Workload scaling |
| Enterprise Teams | High availability systems |

---

# Skills Covered

- Kubernetes ReplicaSets
- Pod Replication
- Self-Healing Infrastructure
- KIND
- kubectl
- Kubernetes YAML
- Scaling Workloads
- Container Orchestration
- DevOps Automation

---

# Expected Project Structure

```text
k8s-replicaset-project/
└── nginx-replicaset.yaml
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

kind create cluster --name replicaset-cluster

kubectl cluster-info --context kind-replicaset-cluster

kubectl get nodes

mkdir -p ~/k8s-replicaset-project

cd ~/k8s-replicaset-project

nano nginx-replicaset.yaml

kubectl apply -f nginx-replicaset.yaml

kubectl get replicasets

kubectl get pods

kubectl describe replicaset nginx-replicaset

kubectl delete pod <pod-name>

kubectl get pods

kubectl scale replicaset nginx-replicaset --replicas=5

kubectl get replicasets

kubectl get pods

kubectl get pods -o wide

kubectl logs <pod-name>

kubectl delete -f nginx-replicaset.yaml

kind delete cluster --name replicaset-cluster
```