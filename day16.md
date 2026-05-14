# Kubernetes Shared Volumes

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to configure shared storage between multiple Pods running inside a Kubernetes cluster.

Your task is to create a Kubernetes shared volume using an `emptyDir` volume and deploy multiple containers that can read and write data from the same shared storage on an Ubuntu-based Kubernetes cluster.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is deploying a microservices-based AI platform on Kubernetes.

The platform contains:

- AI preprocessing services
- Logging containers
- Monitoring sidecars
- Shared caching services
- File-processing pipelines

The engineering team wants multiple containers inside the same Pod to share files, logs, and temporary data.

Current issues:

- Containers cannot share data directly
- Logs are isolated
- Temporary processing files are inaccessible across containers
- Inter-container communication is inefficient

To solve this, the organization wants to implement Kubernetes Shared Volumes.

As a DevOps Engineer, your responsibilities are to:

1. Create a Kubernetes cluster using Kind.
2. Configure shared volumes using `emptyDir`.
3. Deploy multiple containers inside the same Pod.
4. Share files between containers.
5. Verify shared storage functionality.
6. Troubleshoot shared volume communication.

---

# Objectives

1. Install Kubernetes tools.
2. Create a Kind Kubernetes cluster.
3. Configure Kubernetes shared volumes.
4. Deploy multi-container Pods.
5. Share files between containers.
6. Verify volume persistence during Pod lifecycle.

---

# Architecture Overview

```text
+------------------------------------------------+
|                  Kubernetes Pod                |
|                                                |
|   +----------------+    +----------------+     |
|   | Container A    |    | Container B    |     |
|   | Writes Logs    |    | Reads Logs     |     |
|   +--------+-------+    +--------+-------+     |
|            |                     |             |
|            +----------+----------+             |
|                       |                        |
|                Shared Volume                  |
|                 (emptyDir)                    |
+------------------------------------------------+
```

---

# Solution

# Step 1: Update Ubuntu Packages

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

Verify:

```bash
docker --version
```

---

# Step 3: Install Kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
```

Make executable:

```bash
chmod +x ./kind
```

Move binary:

```bash
sudo mv ./kind /usr/local/bin/kind
```

Verify:

```bash
kind --version
```

---

# Step 4: Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s \
https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Make executable:

```bash
chmod +x kubectl
```

Move binary:

```bash
sudo mv kubectl /usr/local/bin/
```

Verify:

```bash
kubectl version --client
```

---

# Step 5: Create Kind Kubernetes Cluster

```bash
kind create cluster --name shared-volume-cluster
```

Verify cluster:

```bash
kubectl cluster-info
```

Expected output:

```text
Kubernetes control plane is running
```

---

# Step 6: Verify Cluster Nodes

```bash
kubectl get nodes
```

Expected output:

```text
NAME                                  STATUS   ROLES           AGE
shared-volume-cluster-control-plane   Ready    control-plane   1m
```

---

# Step 7: Create Shared Volume Manifest

```bash
nano shared-volume-pod.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: shared-volume-pod

spec:
  containers:

    - name: writer-container

      image: busybox

      command:
        - /bin/sh
        - -c
        - |
          while true
          do
            echo "Hello from writer container" >> /shared-data/log.txt
            sleep 5
          done

      volumeMounts:
        - name: shared-storage
          mountPath: /shared-data

    - name: reader-container

      image: busybox

      command:
        - /bin/sh
        - -c
        - |
          while true
          do
            cat /shared-data/log.txt
            sleep 10
          done

      volumeMounts:
        - name: shared-storage
          mountPath: /shared-data

  volumes:
    - name: shared-storage
      emptyDir: {}
```

---

# Step 8: Apply Kubernetes Manifest

```bash
kubectl apply -f shared-volume-pod.yaml
```

Expected output:

```text
pod/shared-volume-pod created
```

---

# Step 9: Verify Pod Status

```bash
kubectl get pods
```

Expected output:

```text
NAME                READY   STATUS    RESTARTS
shared-volume-pod   2/2     Running   0
```

---

# Step 10: Inspect Pod Details

```bash
kubectl describe pod shared-volume-pod
```

Verify:

- Containers running
- Volume mounted
- No errors

---

# Step 11: Verify Shared Volume Data

Check logs from reader container:

```bash
kubectl logs shared-volume-pod -c reader-container
```

Expected output:

```text
Hello from writer container
Hello from writer container
```

---

# Step 12: Access Writer Container

```bash
kubectl exec -it shared-volume-pod -c writer-container -- sh
```

Verify shared files:

```bash
ls /shared-data
```

Expected output:

```text
log.txt
```

View content:

```bash
cat /shared-data/log.txt
```

---

# Step 13: Access Reader Container

```bash
kubectl exec -it shared-volume-pod -c reader-container -- sh
```

Verify:

```bash
cat /shared-data/log.txt
```

Expected output:

```text
Hello from writer container
```

---

# Step 14: Verify Shared Storage Functionality

Both containers should:

- Access same files
- Read shared logs
- Use same storage directory

---

# Step 15: Delete Pod

```bash
kubectl delete pod shared-volume-pod
```

Expected output:

```text
pod "shared-volume-pod" deleted
```

---

# Step 16: Verify emptyDir Behavior

`emptyDir` volume exists only while Pod exists.

Verify Pod deletion:

```bash
kubectl get pods
```

Expected:

```text
No resources found
```

---

# Step 17: Recreate Pod

```bash
kubectl apply -f shared-volume-pod.yaml
```

---

# Step 18: Verify New Shared Volume

```bash
kubectl logs shared-volume-pod -c reader-container
```

Observe:

- Fresh emptyDir volume created
- Old data removed

---

# Step 19: Verify Volume Mounts

```bash
kubectl describe pod shared-volume-pod
```

Check:

```text
Volumes:
Mounts:
```

---

# Step 20: Delete Cluster

```bash
kind delete cluster --name shared-volume-cluster
```

Expected output:

```text
Deleting cluster "shared-volume-cluster"
```

---

# Manifest File

## shared-volume-pod.yaml

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: shared-volume-pod

spec:
  containers:

    - name: writer-container

      image: busybox

      command:
        - /bin/sh
        - -c
        - |
          while true
          do
            echo "Hello from writer container" >> /shared-data/log.txt
            sleep 5
          done

      volumeMounts:
        - name: shared-storage
          mountPath: /shared-data

    - name: reader-container

      image: busybox

      command:
        - /bin/sh
        - -c
        - |
          while true
          do
            cat /shared-data/log.txt
            sleep 10
          done

      volumeMounts:
        - name: shared-storage
          mountPath: /shared-data

  volumes:
    - name: shared-storage
      emptyDir: {}
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Shared volume configured properly
- Multiple containers sharing storage
- Inter-container file sharing operational
- Shared logs accessible across containers
- emptyDir lifecycle behavior verified

---

# Real Industry Use Cases

| Use Case | Description |
|---|---|
| Sidecar Logging | Shared logs |
| AI Pipelines | Shared preprocessing data |
| Monitoring | Shared metrics files |
| Caching | Temporary cache sharing |
| File Processing | Shared processing directories |

---

# Skills Covered

- Kubernetes Volumes
- emptyDir Volumes
- Shared Storage
- Multi-Container Pods
- Kubernetes Storage
- Container Communication
- Kubernetes Debugging
- DevOps

---

# Real Enterprise Shared Volume Workflow

```text
Writer Container
        |
        v
Shared Volume
        |
        v
Reader Container
```

---

# Complete Command Sequence

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install docker.io -y

sudo systemctl enable docker

sudo systemctl start docker

docker --version

curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind

kind --version

curl -LO "https://dl.k8s.io/release/$(curl -L -s \
https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/

kubectl version --client

kind create cluster --name shared-volume-cluster

kubectl cluster-info

kubectl get nodes

nano shared-volume-pod.yaml

kubectl apply -f shared-volume-pod.yaml

kubectl get pods

kubectl describe pod shared-volume-pod

kubectl logs shared-volume-pod -c reader-container

kubectl exec -it shared-volume-pod -c writer-container -- sh

kubectl exec -it shared-volume-pod -c reader-container -- sh

kubectl delete pod shared-volume-pod

kubectl apply -f shared-volume-pod.yaml

kubectl describe pod shared-volume-pod

kind delete cluster --name shared-volume-cluster
```