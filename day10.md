# Day 10: Set Up Time Check in Kubernetes using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to run a Kubernetes-based container that continuously checks and displays the current system time for monitoring and debugging purposes.

Your task is to create a Kubernetes cluster (if not already available), deploy a Time Check Pod using Kubernetes manifest files, and verify continuous time monitoring on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages distributed applications across Kubernetes clusters.

The operations team needs lightweight monitoring containers for:

- Time synchronization checks
- Cluster debugging
- Container runtime validation
- Infrastructure monitoring
- Logging verification
- Health monitoring

Currently, the team faces several operational issues:

- No centralized time validation
- Difficulty debugging container runtime issues
- Inconsistent logging timestamps
- Manual monitoring overhead
- Lack of lightweight monitoring Pods

To solve these problems, the organization wants to deploy Kubernetes Pods that continuously print timestamps for operational visibility.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if one does not exist.
2. Deploy a Time Check Pod using Kubernetes manifests.
3. Verify continuous log generation.
4. Monitor Pod logs and runtime behavior.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Create Kubernetes Pod manifest files.
4. Deploy a Time Check Pod.
5. Verify Pod execution.
6. Monitor continuous logs.
7. Delete resources safely.

---

# Architecture Overview

```text
KIND Kubernetes Cluster
        |
        v
Time Check Pod
        |
        v
BusyBox Container
        |
        v
Continuous Time Logging
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
kind create cluster --name timecheck-cluster
```

Expected output:

```text
Creating cluster "timecheck-cluster" ...
Cluster creation complete.
```

---

# Step 6: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-timecheck-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                               STATUS   ROLES
timecheck-cluster-control-plane   Ready    control-plane
```

---

# Step 7: Create Kubernetes Project Directory

```bash
mkdir -p ~/k8s-timecheck-project

cd ~/k8s-timecheck-project
```

---

# Step 8: Create Time Check Pod Manifest File

```bash
nano time-check-pod.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: time-check-pod

  labels:
    app: time-check

spec:
  containers:
    - name: time-container

      image: busybox

      command:
        - /bin/sh
        - -c
        - |
          while true
          do
            date
            sleep 5
          done
```

---

# Complete Kubernetes Manifest File

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: time-check-pod

  labels:
    app: time-check

spec:
  containers:
    - name: time-container

      image: busybox

      command:
        - /bin/sh
        - -c
        - |
          while true
          do
            date
            sleep 5
          done
```

---

# Manifest Explanation

| Field | Description |
|---|---|
| kind: Pod | Creates Kubernetes Pod |
| busybox | Lightweight Linux container |
| while true | Infinite loop execution |
| date | Prints current timestamp |
| sleep 5 | Waits 5 seconds between logs |

---

# Step 9: Deploy Time Check Pod

```bash
kubectl apply -f time-check-pod.yaml
```

Expected output:

```text
pod/time-check-pod created
```

---

# Step 10: Verify Running Pod

```bash
kubectl get pods
```

Expected output:

```text
NAME             READY   STATUS
time-check-pod   1/1     Running
```

---

# Step 11: Monitor Pod Logs

```bash
kubectl logs -f time-check-pod
```

Expected output:

```text
Thu Jan 2 10:00:01 UTC 2026
Thu Jan 2 10:00:06 UTC 2026
Thu Jan 2 10:00:11 UTC 2026
```

Stop log streaming:

```bash
CTRL + C
```

---

# Step 12: Describe Pod

```bash
kubectl describe pod time-check-pod
```

Displays:

- Pod events
- Container details
- Runtime status
- Pod IP
- Node assignment

---

# Step 13: Verify Pod Networking

```bash
kubectl get pods -o wide
```

Expected output:

```text
NAME             STATUS    IP
time-check-pod   Running   10.244.0.5
```

---

# Step 14: Access Pod Shell

```bash
kubectl exec -it time-check-pod -- /bin/sh
```

Run manual time check:

```bash
date
```

Exit container:

```bash
exit
```

---

# Step 15: Delete Pod

```bash
kubectl delete -f time-check-pod.yaml
```

Expected output:

```text
pod "time-check-pod" deleted
```

---

# Step 16: Delete KIND Cluster

```bash
kind delete cluster --name timecheck-cluster
```

Expected output:

```text
Deleting cluster "timecheck-cluster" ...
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- Time Check Pod deployed successfully
- Continuous timestamp logs generated
- Pod monitoring operational
- Kubernetes runtime verification achieved

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Runtime monitoring |
| SRE Team | Log timestamp validation |
| Platform Engineering Team | Container debugging |
| Cloud Team | Infrastructure diagnostics |
| Enterprise Teams | Lightweight monitoring workloads |

---

# Skills Covered

- Kubernetes Pods
- KIND
- kubectl
- Pod Logging
- Container Monitoring
- Kubernetes YAML
- BusyBox Containers
- DevOps Automation
- Runtime Diagnostics

---

# Expected Project Structure

```text
k8s-timecheck-project/
└── time-check-pod.yaml
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

kind create cluster --name timecheck-cluster

kubectl cluster-info --context kind-timecheck-cluster

kubectl get nodes

mkdir -p ~/k8s-timecheck-project

cd ~/k8s-timecheck-project

nano time-check-pod.yaml

kubectl apply -f time-check-pod.yaml

kubectl get pods

kubectl logs -f time-check-pod

kubectl describe pod time-check-pod

kubectl get pods -o wide

kubectl exec -it time-check-pod -- /bin/sh

kubectl delete -f time-check-pod.yaml

kind delete cluster --name timecheck-cluster
```