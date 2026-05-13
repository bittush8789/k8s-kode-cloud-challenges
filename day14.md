# Day 14: Resolve VolumeMounts Issue in Kubernetes using KIND

# Normal Problem Statement

The DevOps team at xFusionCorp Industries is facing VolumeMount issues inside Kubernetes Pods where containers fail to start due to incorrect volume configurations.

Your task is to create a Kubernetes cluster (if not already available), deploy a faulty Pod with VolumeMount configuration issues, troubleshoot the problem, fix the manifest file, and verify successful Pod execution on an Ubuntu system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages containerized applications running inside Kubernetes clusters.

The organization uses Kubernetes Volumes for:

- Persistent application data
- Shared container storage
- Configuration management
- Log storage
- ML model storage
- Backup and recovery operations

Recently, several Pods failed due to incorrect VolumeMount configurations, causing:

- Pod startup failures
- CrashLoopBackOff errors
- Missing application files
- Invalid mount paths
- Configuration loading failures

The DevOps and SRE teams need a systematic troubleshooting workflow to identify and fix VolumeMount-related issues quickly.

As a DevOps Engineer, your responsibility is to:

1. Create a Kubernetes cluster if one does not exist.
2. Deploy a faulty Pod with incorrect VolumeMount configuration.
3. Investigate Pod failures.
4. Analyze Kubernetes events and logs.
5. Fix VolumeMount configuration issues.
6. Verify successful Pod recovery.

---

# Objectives

1. Install Docker, kubectl, and KIND.
2. Create a Kubernetes cluster using KIND.
3. Deploy a faulty Pod with VolumeMount issues.
4. Troubleshoot Pod failures.
5. Fix VolumeMount configuration errors.
6. Verify successful Pod execution.
7. Delete resources safely.

---

# Architecture Overview

```text
KIND Kubernetes Cluster
        |
        v
Faulty Pod
        |
        v
VolumeMount Error
        |
        +------------------+
        |                  |
        v                  v
Kubernetes Events      Pod Investigation
        |
        v
Fix Manifest
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
kind create cluster --name volumemount-cluster
```

Expected output:

```text
Creating cluster "volumemount-cluster" ...
Cluster creation complete.
```

---

# Step 6: Verify Kubernetes Cluster

```bash
kubectl cluster-info --context kind-volumemount-cluster
```

Check nodes:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                                 STATUS   ROLES
volumemount-cluster-control-plane   Ready    control-plane
```

---

# Step 7: Create Kubernetes Project Directory

```bash
mkdir -p ~/k8s-volumemount-project

cd ~/k8s-volumemount-project
```

---

# Step 8: Create Faulty Pod Manifest File

```bash
nano faulty-volume-pod.yaml
```

Add the following faulty configuration:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: faulty-volume-pod

spec:
  containers:
    - name: nginx-container

      image: nginx:latest

      volumeMounts:
        - name: app-storage
          mountPath: /usr/share/nginx/html

  volumes:
    - name: wrong-storage
      emptyDir: {}
```

---

# Problem in Manifest

The Pod uses:

```yaml
volumeMounts:
  - name: app-storage
```

But inside volumes:

```yaml
- name: wrong-storage
```

Volume names do not match.

---

# Step 9: Deploy Faulty Pod

```bash
kubectl apply -f faulty-volume-pod.yaml
```

Expected output:

```text
pod/faulty-volume-pod created
```

---

# Step 10: Verify Pod Failure

```bash
kubectl get pods
```

Expected output:

```text
NAME                READY   STATUS
faulty-volume-pod   0/1     CreateContainerConfigError
```

---

# Step 11: Describe Pod for Troubleshooting

```bash
kubectl describe pod faulty-volume-pod
```

Expected error:

```text
volumeMounts[0].name: Not found: "app-storage"
```

This command helps identify:

- Volume configuration errors
- Mount path issues
- Container startup problems
- Kubernetes events

---

# Step 12: View Kubernetes Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Expected output:

```text
CreateContainerConfigError
volumeMounts[0].name: Not found
```

---

# Step 13: Fix Manifest File

Edit manifest:

```bash
nano faulty-volume-pod.yaml
```

Replace:

```yaml
- name: wrong-storage
```

With:

```yaml
- name: app-storage
```

---

# Corrected Pod Manifest File

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: faulty-volume-pod

spec:
  containers:
    - name: nginx-container

      image: nginx:latest

      volumeMounts:
        - name: app-storage
          mountPath: /usr/share/nginx/html

  volumes:
    - name: app-storage
      emptyDir: {}
```

---

# Step 14: Delete Faulty Pod

```bash
kubectl delete -f faulty-volume-pod.yaml
```

---

# Step 15: Redeploy Fixed Pod

```bash
kubectl apply -f faulty-volume-pod.yaml
```

Expected output:

```text
pod/faulty-volume-pod created
```

---

# Step 16: Verify Healthy Pod

```bash
kubectl get pods
```

Expected output:

```text
NAME                READY   STATUS
faulty-volume-pod   1/1     Running
```

---

# Step 17: Verify Mounted Volume

Access Pod shell:

```bash
kubectl exec -it faulty-volume-pod -- /bin/bash
```

OR

```bash
kubectl exec -it faulty-volume-pod -- /bin/sh
```

Check mounted directory:

```bash
df -h
```

OR

```bash
mount | grep html
```

Exit container:

```bash
exit
```

---

# Step 18: Verify Pod Details

```bash
kubectl describe pod faulty-volume-pod
```

Expected output:

```text
Volumes:
  app-storage:
    Type: EmptyDir
```

---

# Step 19: Delete Pod

```bash
kubectl delete -f faulty-volume-pod.yaml
```

---

# Step 20: Delete KIND Cluster

```bash
kind delete cluster --name volumemount-cluster
```

Expected output:

```text
Deleting cluster "volumemount-cluster" ...
```

---

# Expected Outcome

- KIND Kubernetes cluster created successfully
- VolumeMount issue reproduced successfully
- Pod troubleshooting workflow implemented
- VolumeMount configuration fixed successfully
- Healthy Pod restored successfully
- Kubernetes volume management operational

---

# Common Kubernetes Volume Issues

| Issue | Description |
|---|---|
| Volume name mismatch | volumeMount name differs from volume name |
| Invalid mountPath | Incorrect container path |
| Missing PersistentVolume | PVC not bound |
| Read-only filesystem | Write permission failure |
| ConfigMap mount issue | Missing ConfigMap |

---

# Real Industry Use Cases

| Team | Use Case |
|---|---|
| DevOps Team | Kubernetes troubleshooting |
| SRE Team | Storage issue resolution |
| Platform Engineering Team | Volume management |
| Cloud Team | Persistent storage debugging |
| Enterprise Teams | Stateful application recovery |

---

# Skills Covered

- Kubernetes Volumes
- VolumeMounts
- Pod Troubleshooting
- CreateContainerConfigError
- KIND
- kubectl
- Kubernetes YAML
- Storage Debugging
- DevOps Incident Management

---

# Expected Project Structure

```text
k8s-volumemount-project/
└── faulty-volume-pod.yaml
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

kind create cluster --name volumemount-cluster

kubectl cluster-info --context kind-volumemount-cluster

kubectl get nodes

mkdir -p ~/k8s-volumemount-project

cd ~/k8s-volumemount-project

nano faulty-volume-pod.yaml

kubectl apply -f faulty-volume-pod.yaml

kubectl get pods

kubectl describe pod faulty-volume-pod

kubectl get events --sort-by=.metadata.creationTimestamp

nano faulty-volume-pod.yaml

kubectl delete -f faulty-volume-pod.yaml

kubectl apply -f faulty-volume-pod.yaml

kubectl get pods

kubectl exec -it faulty-volume-pod -- /bin/bash

kubectl describe pod faulty-volume-pod

kubectl delete -f faulty-volume-pod.yaml

kind delete cluster --name volumemount-cluster
```