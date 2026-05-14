# Kubernetes Troubleshooting

# Normal Problem Statement

The DevOps team at xFusionCorp Industries is facing multiple issues in Kubernetes clusters including Pod failures, Service connectivity issues, CrashLoopBackOff errors, ImagePullBackOff problems, and application downtime.

Your task is to create a Kubernetes cluster using Kind, simulate common Kubernetes failures, troubleshoot issues, fix the problems, and restore healthy application deployments on an Ubuntu-based system.

---

# Scenario-Based Problem Statement

xFusionCorp Industries manages enterprise-grade Kubernetes workloads including:

- AI/ML platforms
- APIs
- Microservices
- Monitoring systems
- Banking applications
- Enterprise dashboards

The engineering team currently faces several critical production issues:

- Pods stuck in CrashLoopBackOff
- ImagePullBackOff errors
- Failed Deployments
- Service connectivity failures
- Volume mount issues
- Secret and ConfigMap problems
- Resource exhaustion
- Node scheduling failures

These issues are impacting:

- Production applications
- CI/CD pipelines
- Customer-facing systems
- Internal services

As a DevOps Engineer, your responsibilities are to:

1. Create Kubernetes cluster using Kind.
2. Simulate real production issues.
3. Diagnose Kubernetes failures.
4. Troubleshoot Pods and Services.
5. Fix YAML misconfigurations.
6. Restore healthy workloads.
7. Implement production-grade debugging workflows.

---

# Objectives

1. Install Kubernetes tools.
2. Create Kind cluster.
3. Deploy broken applications.
4. Diagnose failures.
5. Fix Kubernetes issues.
6. Verify healthy workloads.
7. Learn production troubleshooting workflows.

---

# Common Kubernetes Issues

| Issue | Description |
|---|---|
| CrashLoopBackOff | Container repeatedly crashing |
| ImagePullBackOff | Invalid image |
| ErrImagePull | Registry/image issue |
| Pending Pods | Scheduling issue |
| Service Failure | Wrong selector/port |
| PVC Pending | Storage issue |
| ConfigMap Error | Missing config |
| Secret Failure | Invalid secret |

---

# Architecture Overview

```text
Broken Kubernetes Workload
            |
            v
+----------------------------+
| Kubernetes Cluster         |
+-------------+--------------+
              |
   +----------+----------+
   |                     |
   v                     v
Pod Failures      Service Failures
   |                     |
   +----------+----------+
              |
              v
      Troubleshooting
              |
              v
      Healthy Deployment
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
kind create cluster --name troubleshoot-cluster
```

Verify:

```bash
kubectl cluster-info
```

---

# Step 6: Verify Nodes

```bash
kubectl get nodes
```

Expected:

```text
STATUS: Ready
```

---

# Step 7: Create Namespace

```bash
kubectl create namespace troubleshooting
```

---

# Issue 1: ImagePullBackOff

# Step 8: Create Broken Deployment

```bash
nano broken-image.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: broken-image
  namespace: troubleshooting

spec:
  replicas: 2

  selector:
    matchLabels:
      app: broken-image

  template:
    metadata:
      labels:
        app: broken-image

    spec:
      containers:

        - name: nginx

          image: nginx:wrongtag
```

---

# Step 9: Apply Manifest

```bash
kubectl apply -f broken-image.yaml
```

---

# Step 10: Verify Pods

```bash
kubectl get pods -n troubleshooting
```

Expected:

```text
ImagePullBackOff
```

---

# Step 11: Describe Pod

```bash
kubectl describe pod <pod-name> \
-n troubleshooting
```

Verify:

```text
ErrImagePull
```

---

# Step 12: Fix Deployment

```bash
kubectl set image deployment/broken-image \
nginx=nginx:latest \
-n troubleshooting
```

---

# Step 13: Verify Rollout

```bash
kubectl rollout status deployment/broken-image \
-n troubleshooting
```

---

# Issue 2: CrashLoopBackOff

# Step 14: Create Crash Deployment

```bash
nano crashloop.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: crash-app
  namespace: troubleshooting

spec:
  replicas: 1

  selector:
    matchLabels:
      app: crash-app

  template:
    metadata:
      labels:
        app: crash-app

    spec:
      containers:

        - name: crash-container

          image: busybox

          command: ["false"]
```

---

# Step 15: Apply Manifest

```bash
kubectl apply -f crashloop.yaml
```

---

# Step 16: Verify Pod Failure

```bash
kubectl get pods -n troubleshooting
```

Expected:

```text
CrashLoopBackOff
```

---

# Step 17: Check Logs

```bash
kubectl logs <pod-name> \
-n troubleshooting
```

---

# Step 18: Describe Pod

```bash
kubectl describe pod <pod-name> \
-n troubleshooting
```

---

# Step 19: Fix CrashLoop

```bash
kubectl edit deployment crash-app \
-n troubleshooting
```

Replace:

```yaml
command: ["false"]
```

with:

```yaml
command: ["sleep", "3600"]
```

---

# Step 20: Verify Fixed Pods

```bash
kubectl get pods -n troubleshooting
```

Expected:

```text
Running
```

---

# Issue 3: Service Connectivity Failure

# Step 21: Create Deployment

```bash
nano nginx-app.yaml
```

Add:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-app
  namespace: troubleshooting

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx-app

  template:
    metadata:
      labels:
        app: nginx-app

    spec:
      containers:

        - name: nginx

          image: nginx

          ports:
            - containerPort: 80
```

---

# Step 22: Apply Deployment

```bash
kubectl apply -f nginx-app.yaml
```

---

# Step 23: Create Broken Service

```bash
nano broken-service.yaml
```

Add:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: broken-service
  namespace: troubleshooting

spec:
  selector:
    app: wrong-label

  ports:
    - port: 80
      targetPort: 80
```

---

# Step 24: Apply Service

```bash
kubectl apply -f broken-service.yaml
```

---

# Step 25: Verify Endpoints

```bash
kubectl get endpoints -n troubleshooting
```

Expected:

```text
No endpoints
```

---

# Step 26: Fix Service Selector

```bash
kubectl edit svc broken-service \
-n troubleshooting
```

Replace:

```yaml
app: wrong-label
```

with:

```yaml
app: nginx-app
```

---

# Step 27: Verify Endpoints

```bash
kubectl get endpoints -n troubleshooting
```

Expected:

```text
Pod IPs visible
```

---

# Issue 4: ConfigMap Failure

# Step 28: Create Broken ConfigMap Pod

```bash
nano broken-configmap.yaml
```

Add:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: configmap-pod
  namespace: troubleshooting

spec:
  containers:

    - name: nginx

      image: nginx

      env:
        - name: APP_MODE
          valueFrom:
            configMapKeyRef:
              name: missing-config
              key: APP_MODE
```

---

# Step 29: Apply Manifest

```bash
kubectl apply -f broken-configmap.yaml
```

---

# Step 30: Verify Pod Failure

```bash
kubectl get pods -n troubleshooting
```

Expected:

```text
CreateContainerConfigError
```

---

# Step 31: Describe Pod

```bash
kubectl describe pod configmap-pod \
-n troubleshooting
```

Expected:

```text
ConfigMap not found
```

---

# Step 32: Create Missing ConfigMap

```bash
kubectl create configmap missing-config \
--from-literal=APP_MODE=production \
-n troubleshooting
```

---

# Step 33: Delete Broken Pod

```bash
kubectl delete pod configmap-pod \
-n troubleshooting
```

---

# Step 34: Recreate Pod

```bash
kubectl apply -f broken-configmap.yaml
```

---

# Step 35: Verify Healthy Pod

```bash
kubectl get pods -n troubleshooting
```

Expected:

```text
Running
```

---

# Issue 5: PVC Pending Issue

# Step 36: Create PVC Without PV

```bash
nano pvc.yaml
```

Add:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: broken-pvc
  namespace: troubleshooting

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 1Gi
```

---

# Step 37: Apply PVC

```bash
kubectl apply -f pvc.yaml
```

---

# Step 38: Verify PVC

```bash
kubectl get pvc -n troubleshooting
```

Expected:

```text
Pending
```

---

# Step 39: Create Persistent Volume

```bash
nano pv.yaml
```

Add:

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: app-pv

spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/pv-storage
```

---

# Step 40: Apply PV

```bash
kubectl apply -f pv.yaml
```

---

# Step 41: Verify PVC Binding

```bash
kubectl get pvc -n troubleshooting
```

Expected:

```text
Bound
```

---

# Important Troubleshooting Commands

| Command | Purpose |
|---|---|
| kubectl get pods | Pod status |
| kubectl describe pod | Detailed pod info |
| kubectl logs | Container logs |
| kubectl get events | Cluster events |
| kubectl get endpoints | Service endpoints |
| kubectl rollout status | Deployment rollout |
| kubectl exec | Access containers |

---

# Step 42: Verify All Resources

```bash
kubectl get all -n troubleshooting

kubectl get pv

kubectl get pvc -n troubleshooting
```

---

# Step 43: Real Production Troubleshooting Workflow

```text
Application Failure
        |
        v
Check Pod Status
        |
        v
Describe Resources
        |
        v
Check Logs & Events
        |
        v
Identify Root Cause
        |
        v
Fix Configuration
        |
        v
Redeploy Application
        |
        v
Verify Health
```

---

# Step 44: Common Kubernetes Error Meanings

| Error | Meaning |
|---|---|
| CrashLoopBackOff | App crashing repeatedly |
| ImagePullBackOff | Invalid image |
| Pending | Scheduling/storage issue |
| CreateContainerConfigError | ConfigMap/Secret issue |
| OOMKilled | Out of memory |
| ErrImagePull | Registry/image problem |

---

# Step 45: Delete Resources

```bash
kubectl delete deployment broken-image -n troubleshooting

kubectl delete deployment crash-app -n troubleshooting

kubectl delete deployment nginx-app -n troubleshooting

kubectl delete pod configmap-pod -n troubleshooting

kubectl delete svc broken-service -n troubleshooting

kubectl delete pvc broken-pvc -n troubleshooting

kubectl delete pv app-pv
```

---

# Step 46: Delete Namespace

```bash
kubectl delete namespace troubleshooting
```

---

# Step 47: Delete Kind Cluster

```bash
kind delete cluster --name troubleshoot-cluster
```

---

# Manifest Files

# broken-image.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: broken-image
  namespace: troubleshooting

spec:
  replicas: 2

  selector:
    matchLabels:
      app: broken-image

  template:
    metadata:
      labels:
        app: broken-image

    spec:
      containers:

        - name: nginx

          image: nginx:wrongtag
```

---

# crashloop.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: crash-app
  namespace: troubleshooting

spec:
  replicas: 1

  selector:
    matchLabels:
      app: crash-app

  template:
    metadata:
      labels:
        app: crash-app

    spec:
      containers:

        - name: crash-container

          image: busybox

          command: ["sleep", "3600"]
```

---

# broken-service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: broken-service
  namespace: troubleshooting

spec:
  selector:
    app: nginx-app

  ports:
    - port: 80
      targetPort: 80
```

---

# broken-configmap.yaml

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: configmap-pod
  namespace: troubleshooting

spec:
  containers:

    - name: nginx

      image: nginx

      env:
        - name: APP_MODE
          valueFrom:
            configMapKeyRef:
              name: missing-config
              key: APP_MODE
```

---

# pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: broken-pvc
  namespace: troubleshooting

spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 1Gi
```

---

# pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: app-pv

spec:
  capacity:
    storage: 1Gi

  accessModes:
    - ReadWriteOnce

  hostPath:
    path: /data/pv-storage
```

---

# Expected Outcome

- Kind Kubernetes cluster created successfully
- Multiple Kubernetes issues diagnosed
- CrashLoopBackOff fixed
- ImagePullBackOff fixed
- Service connectivity restored
- ConfigMap issue resolved
- Persistent storage issue resolved
- Production-grade troubleshooting workflow implemented

---

# Skills Covered

- Kubernetes Troubleshooting
- Pod Debugging
- Service Troubleshooting
- ConfigMap Troubleshooting
- Persistent Volume Troubleshooting
- CrashLoopBackOff Resolution
- DevOps Incident Management
- Cloud-Native Debugging

---

# Real Enterprise Use Cases

| Use Case | Description |
|---|---|
| Production Outages | Incident response |
| CI/CD Failures | Deployment debugging |
| Platform Engineering | Root cause analysis |
| Kubernetes Operations | Cluster troubleshooting |
| DevOps | Production support |

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

kind create cluster --name troubleshoot-cluster

kubectl create namespace troubleshooting

kubectl apply -f broken-image.yaml

kubectl get pods -n troubleshooting

kubectl describe pod <pod-name> \
-n troubleshooting

kubectl set image deployment/broken-image \
nginx=nginx:latest \
-n troubleshooting

kubectl apply -f crashloop.yaml

kubectl logs <pod-name> \
-n troubleshooting

kubectl edit deployment crash-app \
-n troubleshooting

kubectl apply -f nginx-app.yaml

kubectl apply -f broken-service.yaml

kubectl get endpoints -n troubleshooting

kubectl edit svc broken-service \
-n troubleshooting

kubectl apply -f broken-configmap.yaml

kubectl create configmap missing-config \
--from-literal=APP_MODE=production \
-n troubleshooting

kubectl apply -f pvc.yaml

kubectl apply -f pv.yaml

kubectl get all -n troubleshooting

kubectl get pv

kubectl get pvc -n troubleshooting

kubectl delete namespace troubleshooting

kind delete cluster --name troubleshoot-cluster
```