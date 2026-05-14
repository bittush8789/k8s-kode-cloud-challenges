# Kubernetes Sidecar Containers

# Normal Problem Statement

The DevOps team at xFusionCorp Industries wants to implement Sidecar Containers in Kubernetes for centralized logging and monitoring.

Your task is to deploy a multi-container Kubernetes Pod where the primary application container writes logs and the sidecar container continuously reads and processes those logs using shared volumes on an Ubuntu-based Kubernetes cluster.

---

# Scenario-Based Problem Statement

xFusionCorp Industries is deploying cloud-native AI and microservices applications on Kubernetes.

The organization runs:

- AI applications
- Microservices
- API gateways
- Monitoring systems
- Log aggregation pipelines

The engineering team faces several challenges:

- Application logs are isolated inside containers
- Centralized logging is difficult
- Monitoring agents need shared access to logs
- Containers need helper utilities without modifying the main image

To solve these issues, the organization wants to implement the Kubernetes Sidecar Container pattern.

As a DevOps Engineer, your responsibilities are to:

1. Create a Kubernetes cluster using Kind.
2. Configure multi-container Pods.
3. Implement sidecar architecture.
4. Share logs using shared volumes.
5. Verify inter-container communication.
6. Troubleshoot sidecar logging workflows.

---

# Objectives

1. Install Kubernetes tools.
2. Create a Kind Kubernetes cluster.
3. Deploy sidecar containers.
4. Configure shared volumes.
5. Implement centralized log reading.
6. Verify sidecar functionality.

---

# What is a Sidecar Container?

A Sidecar Container is a helper container that runs alongside the primary application container inside the same Pod.

It is commonly used for:

- Logging
- Monitoring
- Proxying
- Security
- Configuration synchronization
- Data preprocessing

---

# Sidecar Architecture Overview

```text
+--------------------------------------------------+
|                Kubernetes Pod                    |
|                                                  |
|   +----------------+    +--------------------+   |
|   | Main App       |    | Sidecar Container  |   |
|   | Writes Logs    |    | Reads Logs         |   |
|   +--------+-------+    +---------+----------+   |
|            |                        |             |
|            +-----------+------------+             |
|                        |                          |
|                 Shared Volume                     |
|                  (emptyDir)                       |
+--------------------------------------------------+
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
kind create cluster --name sidecar-cluster
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

# Step 6: Verify Nodes

```bash
kubectl get nodes
```

Expected output:

```text
NAME                           STATUS   ROLES           AGE
sidecar-cluster-control-plane  Ready    control-plane   1m
```

---

# Step 7: Create Sidecar Pod Manifest

```bash
nano sidecar-pod.yaml
```

Add the following YAML:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: sidecar-pod

spec:
  containers:

    - name: main-app

      image: busybox

      command:
        - /bin/sh
        - -c
        - |
          while true
          do
            echo "$(date) : Application running" >> /var/log/app/app.log
            sleep 5
          done

      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app

    - name: log-sidecar

      image: busybox

      command:
        - /bin/sh
        - -c
        - |
          tail -f /var/log/app/app.log

      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app

  volumes:
    - name: shared-logs
      emptyDir: {}
```

---

# Step 8: Apply Kubernetes Manifest

```bash
kubectl apply -f sidecar-pod.yaml
```

Expected output:

```text
pod/sidecar-pod created
```

---

# Step 9: Verify Pod Status

```bash
kubectl get pods
```

Expected output:

```text
NAME           READY   STATUS    RESTARTS
sidecar-pod    2/2     Running   0
```

---

# Step 10: Inspect Pod Details

```bash
kubectl describe pod sidecar-pod
```

Verify:

- Both containers running
- Shared volume mounted
- No container errors

---

# Step 11: Verify Sidecar Logs

View logs from sidecar container:

```bash
kubectl logs sidecar-pod -c log-sidecar
```

Expected output:

```text
Fri Jan 10 12:00:00 UTC 2026 : Application running
Fri Jan 10 12:00:05 UTC 2026 : Application running
```

---

# Step 12: Access Main Application Container

```bash
kubectl exec -it sidecar-pod -c main-app -- sh
```

Verify logs:

```bash
cat /var/log/app/app.log
```

Expected output:

```text
Application running
```

---

# Step 13: Access Sidecar Container

```bash
kubectl exec -it sidecar-pod -c log-sidecar -- sh
```

Verify shared logs:

```bash
cat /var/log/app/app.log
```

---

# Step 14: Verify Shared Volume

Both containers should access:

```text
/var/log/app
```

Verify:

- Shared file access
- Shared logging workflow
- Real-time log streaming

---

# Step 15: Test Continuous Logging

Watch logs live:

```bash
kubectl logs -f sidecar-pod -c log-sidecar
```

Observe continuous log updates.

---

# Step 16: Delete Pod

```bash
kubectl delete pod sidecar-pod
```

Expected output:

```text
pod "sidecar-pod" deleted
```

---

# Step 17: Recreate Pod

```bash
kubectl apply -f sidecar-pod.yaml
```

---

# Step 18: Verify Sidecar Reinitialization

```bash
kubectl get pods
```

Verify both containers restart successfully.

---

# Step 19: Verify emptyDir Lifecycle

Since `emptyDir` is temporary storage:

- Data persists only while Pod exists
- Deleting Pod removes logs

---

# Step 20: Delete Kubernetes Cluster

```bash
kind delete cluster --name sidecar-cluster
```

Expected output:

```text
Deleting cluster "sidecar-cluster"
```

---

# Manifest File

## sidecar-pod.yaml

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: sidecar-pod

spec:
  containers:

    - name: main-app

      image: busybox

      command:
        - /bin/sh
        - -c
        - |
          while true
          do
            echo "$(date) : Application running" >> /var/log/app/app.log
            sleep 5
          done

      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app

    - name: log-sidecar

      image: busybox

      command:
        - /bin/sh
        - -c
        - |
          tail -f /var/log/app/app.log

      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app

  volumes:
    - name: shared-logs
      emptyDir: {}
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Sidecar architecture implemented
- Shared volume configured correctly
- Sidecar container reading logs successfully
- Inter-container communication operational
- Centralized logging workflow achieved

---

# Real Industry Use Cases

| Use Case | Description |
|---|---|
| Logging | Fluentd/Filebeat sidecars |
| Monitoring | Prometheus exporters |
| Security | Service mesh proxies |
| AI Pipelines | Shared preprocessing |
| Networking | Envoy sidecars |

---

# Skills Covered

- Kubernetes Sidecar Pattern
- Multi-Container Pods
- Shared Volumes
- emptyDir Volumes
- Container Logging
- Kubernetes Debugging
- DevOps
- Cloud-Native Architecture

---

# Real Enterprise Sidecar Workflow

```text
Main Application
        |
        v
Shared Volume
        |
        v
Sidecar Container
        |
        v
Log Aggregation / Monitoring
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

kind create cluster --name sidecar-cluster

kubectl cluster-info

kubectl get nodes

nano sidecar-pod.yaml

kubectl apply -f sidecar-pod.yaml

kubectl get pods

kubectl describe pod sidecar-pod

kubectl logs sidecar-pod -c log-sidecar

kubectl exec -it sidecar-pod -c main-app -- sh

kubectl exec -it sidecar-pod -c log-sidecar -- sh

kubectl logs -f sidecar-pod -c log-sidecar

kubectl delete pod sidecar-pod

kubectl apply -f sidecar-pod.yaml

kind delete cluster --name sidecar-cluster
```